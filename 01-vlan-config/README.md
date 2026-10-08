# Lab: VLAN Configuration and Inter-VLAN Routing

**Tool:** Packet Tracer

## Goal
Segment a network into 3 VLANs and enable communication between them through a router, using one physical router interface per VLAN.

## Topology
<img width="1137" height="904" alt="01-vlan-config-topology" src="https://github.com/user-attachments/assets/f455b0bb-b208-4802-9751-e78c93e5e654" />


## What I configured
- Created and named 3 VLANs on the switch: VLAN [10] [Engineering], VLAN [20] [HR], VLAN [30] [Sales]
- Assigned access ports to the correct VLAN for each client (e.g. Fa3/1 to Fa4/1 in VLAN [10])
- Configured IP addresses, subnet masks, and default gateways on each client
- Configured an IP address on each router interface to serve as the gateway for its VLAN

## How I verified it
- `show vlan brief` to confirm VLAN names and port assignments
- `show ip interface brief` on the router to confirm interfaces were up
- `show ip route` to confirm the connected networks
- Ping tests between hosts in the same VLAN and across VLANs

## What I learned
This lab demonstrates how to configure end devices and switches with VLANs, and why trunks matter. Here I used 3 separate physical connections to the router, one per VLAN. With a trunk, I could use a single physical connection and configure a sub-interface for each VLAN.

## Configs
See the [configs](configs) folder for the switch and router running configs.
