# Lab: Switch Virtual Interface (SVI) Inter-VLAN Routing

**Tool:** Packet Tracer

## Goal
Replace router-on-a-stick with inter-VLAN routing on a Layer 3 switch using SVIs, and connect the switch to the router over a routed point-to-point link.

## Topology
<img width="1298" height="931" alt="03-SVI-config topology" src="https://github.com/user-attachments/assets/2cb14ae1-87a6-4f61-81a5-d554c6af9691" />

## Addressing
| Interface | IP address | Purpose |
|-----------|------------|---------|
| SW2 SVI VLAN 10 | 10.0.0.62/26 | Gateway for VLAN 10 |
| SW2 SVI VLAN 20 | 10.0.0.126/26 | Gateway for VLAN 20 |
| SW2 SVI VLAN 30 | 10.0.0.190/26 | Gateway for VLAN 30 |
| SW2 g1/0/2 | 10.0.0.193/30 | Link to R1 |
| R1 g0/0 | 10.0.0.194/30 | Link to SW2 |

## What I configured
- Removed the ROAS sub-interfaces (g0/0.10, .20, .30) from R1 and addressed R1's g0/0 as a routed link (10.0.0.194/30)
- Enabled Layer 3 routing on SW2 with `ip routing`
- Configured SW2's g1/0/2 as a routed port [with `no switchport`] with IP 10.0.0.193/30
- Created an SVI for each VLAN (10, 20, 30) on SW2 to act as the default gateway for its hosts
- Added a default route on SW2 pointing to R1 (10.0.0.194)

## How I verified it
- `show vlan brief` to confirm VLANs and port assignments
- `show ip interface brief` on R1 and SW2 to confirm interfaces and SVIs were up/up
- `show ip route` on SW2 to confirm connected routes for each VLAN and the default route
- Ping tests between hosts in the same VLAN and across VLANs

## What I learned
This lab builds on the previous one. Instead of the router handling inter-VLAN routing over a single trunk, the Layer 3 switch routes between VLANs with SVIs. That removes the single-link bottleneck, routes in hardware, and frees the router to handle other tasks like routing to the internet. The trade-off is that it requires a Layer 3 capable switch. I also learned that a Layer 3 switch won't route until `ip routing` is enabled.

## Configs
See the [configs](configs) folder for the switch and router running configs.
