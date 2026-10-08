# Lab: Router-on-a-Stick (ROAS) and Trunk Configuration

**Tool:** Packet Tracer

## Goal
Segment a network into 3 VLANs, configure trunks between switches and the router, and enable inter-VLAN communication using a router-on-a-stick (ROAS) topology with a single physical router link.

## Topology
<img width="1170" height="891" alt="02-ROAS-config-topology" src="https://github.com/user-attachments/assets/09e2f8b5-b3af-44b9-ad84-678a289dfc31" />

## Addressing
| VLAN | Subnet | Gateway (R1 sub-interface) |
|------|--------|----------------------------|
| 10 | 255.255.255.192 | 10.0.0.62 (g0/0.10) |
| 20 | 255.255.255.192 | 10.0.0.126 (g0/0.20) |
| 30 | 255.255.255.192 | 10.0.0.190 (g0/0.30) |

## What I configured
- Assigned access ports to the correct VLAN for each client (e.g. Fa0/1 to Fa0/2 in VLAN 10)
- Configured 802.1Q trunks between SW1-SW2 and SW2-R1, allowing VLANs 10, 20, and 30
- Set the trunk native VLAN to VLAN 1001 (an unused VLAN)
- Created a sub-interface on R1 for each VLAN with dot1q encapsulation and an IP address (e.g. `g0/0.10`, 10.0.0.62 255.255.255.192)

## How I verified it
- `show vlan brief` to confirm VLANs and port assignments on each switch
- `show interfaces trunk` to confirm trunking, allowed VLANs, and native VLAN
- `show ip interface brief` on R1 to confirm sub-interfaces were up
- `show ip route` on R1 to confirm connected routes for each VLAN
- Ping tests between hosts in the same VLAN and across VLANs

## Problems I hit and how I fixed them
- **Problem:** VLAN 20 & VLAN 30 didn't exist on SW1 or SW2 respectively, so traffic from those VLANs was not being tagged on the trunk
- **Fix:** Created VLAN 20 on SW1 and VLAN 30 on SW2 so every VLAN existed on both switches

## What I learned
This lab builds on the previous one. Instead of three physical router links, ROAS carries all three VLANs over a single trunk using sub-interfaces. That saves router ports, but the one link becomes a bottleneck and a single point of failure. I also learned that a trunk only carries VLANs that exist on each switch, so VLANs must be created on every switch in the path.

## Configs
See the [configs](configs) folder for the switch and router running configs.
