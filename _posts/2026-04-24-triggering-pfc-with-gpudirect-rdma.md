---
layout: post
comments: True
status: publish
published: true
title: Triggering PFC with GPUDirect RDMA
date: '2026-04-23 00:00:00 +0000'
categories:
- Linux
- Networking
- RDMA
- GPU
permalink: triggering-pfc-with-gpudirect-rdma
---

In the [previous post](/enabling-gpudirect-rdma-on-geforce-gpus) I patched the NVIDIA driver to enable GPUDirect RDMA on my GeForce GPUs. The motivation was Priority Flow Control which requires the NIC's receive buffer to fill up before it sends a PAUSE frame and with standard CPU-memory RDMA the NIC drains its buffer to DRAM faster than the wire can fill it. GPUDirect changes this by targeting the GPU memory instead of system RAM, introducing enough PCIe latency to create backpressure.

This post puts that theory to the test on the [point-to-point lab](/multi-node-rdma-lab-single-machine) by configuring PFC, running bidirectional GPUDirect RDMA traffic and observing PFC pause frames in action.

___

<p align="center">
  <img src="/public/images/pfc-gpudirect-rdma.svg" alt="PFC with GPUDirect RDMA: NIC buffer backpressure triggers lossless flow control"/>
</p>

## Why PFC Needs GPUDirect

PFC is simple in principle: when a receiver's NIC buffer fills past the xoff threshold, it sends a PAUSE frame telling the sender to stop transmitting on that priority. When the buffer drains below the xon threshold traffic resumes. This prevents packet drops due to buffer overflow, the L2 backstop for lossless RoCEv2.

The challenge in my lab is triggering it. With CPU-memory RDMA on a 25G link the NIC writes incoming data to system RAM at PCIe Gen3 x8 speed (~7.88 GB/s), well above the 25G wire rate (~3.1 GB/s). The buffer never fills up and PFC never fires.

GPUDirect RDMA redirects the NIC's writes from system RAM to the GPU memory. The PCIe path now crosses from the NIC's root port through the CPU's Infinity Fabric to the GPU's root port which is a longer path with higher latency. In my setup the GPU and NIC also sit on different NUMA nodes (GPU on NUMA 0/1, NIC on NUMA 2/3) which is adding even more hops to every DMA transaction.

```
GPUDirect RDMA:
  Wire (3.1 GB/s) → NIC RX buffer → DMA to GPU BAR1 (cross-NUMA PCIe)
                     ↑ can fill up    ↑ higher latency,stalls possible
```

During bidirectional traffic both the NIC and GPU are simultaneously handling DMA reads (for outgoing data) and DMA writes (for incoming data). The PCIe bus must arbitrate between these competing transactions. When the NIC exhausts its outstanding PCIe transaction credits waiting for GPU BAR1 write completions, incoming packets pile up in the receive buffer until it hits the xoff threshold and fires a PAUSE frame.

## Prerequisites

This experiment builds on the [GPUDirect RDMA setup](/enabling-gpudirect-rdma-on-geforce-gpus). Both VMs need:

- Rocky Linux 9.5 with MLNX_OFED 24.10
- Patched NVIDIA 570.133.20 open kernel modules (BAR1 P2P + persistent peermem)
- Patched perftest with GPUDirect GeForce support
- PF passthrough of ConnectX-4 Lx NICs

Verify the setup on both VMs:

```bash
# Patched NVIDIA modules with peermem
lsmod | grep -E "nvidia_peermem|nvidia_uvm"
# nvidia_peermem         20480  0
# nvidia_uvm           4001792  0

# Peer memory client registered
cat /sys/kernel/mm/memory_peers/nv_mem/version
# 570.133.20

# RDMA device active
ibv_devinfo mlx5_0 | grep -E "state|active_mtu"
#			state:			PORT_ACTIVE (4)
#			active_mtu:		4096 (5)
```

## Configuring PFC

Enable PFC on priority 0 on both VMs:

```bash
# Enable PFC on priority 0
mlnx_qos -i eth1 --pfc 1,0,0,0,0,0,0,0

# Map priority 0 to buffer 1
mlnx_qos -i eth1 --prio2buffer 1,0,0,0,0,0,0,0

# Set cable length for threshold calculation
mlnx_qos -i eth1 --cable_len 1

# Set the lossless buffer size to lower the xoff threshold
mlnx_qos -i eth1 --buffer_size 0,114944,0,0,0,0,0,0
```

When PFC is enabled the NIC splits its ~256 KB internal buffer between lossless (PFC-enabled) and lossy traffic classes. Priority 0 maps to buffer 1, the lossless buffer:

```bash
mlnx_qos -i eth1
# PFC configuration:
#     priority    0   1   2   3   4   5   6   7
#     enabled     1   0   0   0   0   0   0   0
#     buffer      1   0   0   0   0   0   0   0

cat /sys/class/net/eth1/qos/buffer_size
# Port buffer size = 262016
# Spare buffer size = 147072
# Buffer  Size      xoff_threshold  xon_threshold
# 0       0         0               524160
# 1       114944    91392           81408
```

The xoff threshold is the buffer fill level that triggers a PAUSE frame. The xon threshold is where the NIC tells the sender to resume.

## Baseline: CPU-Memory Bidirectional

First, verify that standard CPU-memory RDMA never triggers PFC. Run bidirectional RDMA WRITE with both VMs simultaneously writing to each other:

```bash
# vm00 (server):
ib_write_bw -d mlx5_0 -x 3 -s 65536 -D 30 -b

# vm01 (client):
ib_write_bw -d mlx5_0 -x 3 -s 65536 -D 30 -b 10.0.0.1
```

```
#  #bytes     #iterations    BW peak[MiB/sec]    BW average[MiB/sec]   MsgRate[Mpps]
#  65536      279928           0.00               5830.79               0.093293
```

Check PFC counters on both VMs:

```bash
ethtool -S eth1 | grep prio0_pause
# rx_prio0_pause: 0
# tx_prio0_pause: 0
```

The `-b` flag runs traffic in both directions simultaneously and reports the aggregate bandwidth. Since 25G Ethernet is full-duplex (25 Gbps in each direction) the aggregate can reach ~5960 MiB/s.

## GPUDirect Bidirectional

Now run the same test with GPUDirect RDMA using the patched perftest. Starting with a 30-second duration to give PFC events enough time to accumulate:

```bash
# vm00 (server):
GPUDIRECT_GPU=0 ./ib_write_bw -d mlx5_0 -x 3 --use_cuda=0 -s 65536 -D 30 -b

# vm01 (client):
GPUDIRECT_GPU=0 ./ib_write_bw -d mlx5_0 -x 3 --use_cuda=0 -s 65536 -D 30 -b 10.0.0.1
```

```
#  #bytes     #iterations    BW peak[MiB/sec]    BW average[MiB/sec]   MsgRate[Mpps]
#  65536      674811           0.00               5295.75               0.084732
```

Check the PFC counters:

```bash
# vm00:
ethtool -S eth1 | grep prio0_pause
# rx_prio0_pause: 12667
# tx_prio0_pause: 80
# rx_prio0_pause_transition: 5418

# vm01:
ethtool -S eth1 | grep prio0_pause
# rx_prio0_pause: 80
# tx_prio0_pause: 12667
```

12,667 PFC pause frames from vm01 to vm00 in 30 seconds. vm01's receive buffer filled past the xoff threshold thousands of times, each time sending a PAUSE frame that made vm00 briefly stop transmitting.

The `rx_prio0_pause_transition` counter on vm00 (5,418) shows the number of times vm00's transmitter transitioned between paused and active states.

| Test | Duration | Aggregate BW | PFC Pauses | Drops |
|------|----------|--------------|------------|-------|
| CPU-memory bidirectional | 30s | 5831 MiB/s | 0 | 0 |
| GPUDirect bidirectional | 30s | 5296 MiB/s | 12,667 | 0 |

## Verifying Lossless Delivery

The whole point of PFC is preventing drops. Verify that no packets were lost and no RDMA retransmissions occurred:

```bash
# On both VMs after the GPUDirect test:
ethtool -S eth1 | grep -E "rx_out_of_buffer|rx_discards"
# rx_out_of_buffer: 0
# rx_discards_phy: 0

rdma statistic show link mlx5_0/1 | grep -oP '(packet_seq_err|out_of_sequence|local_ack_timeout_err)\s+\d+'
# out_of_sequence 0
# packet_seq_err 0
# local_ack_timeout_err 0
```

## The Full Lossless RoCEv2 Picture

With PFC now demonstrated the lab has exercised both halves of the lossless RoCEv2 stack:

```
                 DCQCN (end-to-end)
  ┌──────────────────────────────────────────┐
  │                                          │
  │    marks CE          sends CNP           │
  │  CP ──────────► NP ──────────► RP        │
  │  (bridge)    (receiver)     (sender)     │
  │                                          │
  └──────────────────────────────────────────┘

                 PFC (hop-by-hop)
  ┌──────────────────────────────────────────┐
  │                                          │
  │  buffer fills → PAUSE → sender stops     │
  │  buffer drains → RESUME → sender resumes │
  │  (receiver NIC)          (sender NIC)    │
  │                                          │
  └──────────────────────────────────────────┘
```

- **DCQCN** ([demonstrated through the software congestion point](/dcqcn-software-congestion-point)) is the end-to-end congestion control mechanism. A congested switch marks packets with ECN, the receiver generates CNPs and the sender reduces its rate. It prevents congestion from building up in the first place.

- **PFC** (demonstrated in this post) is the L2 hop-by-hop backstop. When DCQCN can't reduce rates fast enough and a receiver's buffer starts to overflow PFC pauses the sender on that specific priority. It's the last line of defense against packet drops.

Real-world GPU clusters use both mechanisms simultaneously. DCQCN manages steady-state congestion, while PFC catches transient bursts that exceed switch buffer limits. We covered them separately because our point-to-point topology lacks a switch. However, despite the simplified topology, I was able to experiment and familiarize myself with the congestion control mechanisms.
