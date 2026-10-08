# Lab: Spanning Tree Protocol (PVST+) Configuration

**Tool:** Packet Tracer

## Goal
Examine the default STP topology on four switches, then control root bridge placement per VLAN, influence root port selection with port cost and priority, and protect edge ports with PortFast and BPDU Guard.

## Topology
<img width="954" height="881" alt="04-STP-config topology" src="https://github.com/user-attachments/assets/f8536de1-c40d-4792-8f3f-d7c4899ada6c" />

## What I configured
- Checked the default STP topology on all four switches and identified the root bridge for each VLAN
- Made SW1 the primary root for VLAN 1 and the secondary root for VLAN 2
- Made SW2 the primary root for VLAN 2 and the secondary root for VLAN 1
- Changed the VLAN 1 port cost on SW4 F0/2 to 100
- Changed the VLAN 1 port priority on SW1 F0/1 to 240
- Enabled PortFast and BPDU Guard on the host-facing F0/3 ports on SW3 and SW4

## Key commands
```
show spanning-tree
show spanning-tree vlan 1
spanning-tree vlan 1 root primary
spanning-tree vlan 2 root secondary
spanning-tree vlan 1 cost 100
spanning-tree vlan 1 port-priority 240
spanning-tree portfast
spanning-tree bpduguard enable
```

## Results

### 1. Default topology
- **Root bridge (VLAN 1):** SW2
- **Root bridge (VLAN 2):** SW2

### 2. After setting root bridges
SW1 is now the root for VLAN 1 and SW2 is the root for VLAN 2, each with the other as the backup.

## How I verified it
- `show spanning-tree vlan 1` and `show spanning-tree vlan 2` on each switch to confirm the root bridge, port roles, and states
- `show spanning-tree summary` to confirm PortFast and BPDU Guard are enabled
- `show running-config interface f0/3` on SW3 and SW4

## What I learned
STP in Cisco switches runs per VLAN (PVST+), so different VLANs can use different root bridges and balance traffic across the links. The root bridge is chosen by lowest bridge ID, which I can control with `root primary` and `root secondary` instead of leaving it to MAC addresses. Root port selection goes in order: lowest root path cost, then lowest sender bridge ID, then lowest sender port priority. That's why changing cost moved the root port but changing priority didn't. PortFast and BPDU Guard are used together on edge ports.

## Configs
See the [configs](configs) folder for the running configs.
