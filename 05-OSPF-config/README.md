# Lab: Single-Area OSPF with Default Route Advertisement

**Tool:** Packet Tracer

## Goal
Configure single-area OSPF (area 0) across four routers, add loopback interfaces, set passive interfaces, and make R1 an ASBR that advertises a default route toward the ISP into the OSPF domain.

## Topology
<img width="1275" height="866" alt="05-OSPF-config topology" src="https://github.com/user-attachments/assets/a4b6794f-d5ea-484c-a93b-d5780ed29f4e" />

## What I configured
- Set hostnames and IP addresses on all routers and enabled the interfaces
- Created a loopback interface on each router (1.1.1.1/32 through 4.4.4.4/32)
- Enabled OSPF (area 0) on every interface, including loopbacks, but not on R1's link to the ISP
- Set loopbacks and the R4 LAN interface (G0/0) as passive interfaces so they're advertised without forming neighbors
- Configured R1 as an ASBR with a static default route toward ISPR1 and `default-information originate` to advertise it into OSPF

## Key commands
```
router ospf 1
 network [address] [wildcard] area 0
 passive-interface [interface]
 default-information originate

ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

## How I verified it
- `show ip ospf neighbor` on each router to confirm adjacencies are FULL
- `show ip ospf interface brief` to confirm interfaces are in area 0 and passive where intended
- `show ip route ospf` on R2, R3, and R4 to confirm learned routes and the default route
- `show ip protocols` to confirm passive interfaces and OSPF networks
- Ping tests from PC1 to the ISP

## Results
**Default routes on R2, R3, and R4:**

| Router | Default route | Next hop |
|--------|---------------|----------|
| R2 | O*E2 0.0.0.0/0 | [10.0.12.1 (R1)] |
| R3 | O*E2 0.0.0.0/0 | [10.0.13.1 (R1)] |
| R4 | O*E2 0.0.0.0/0 | [10.0.24.1 (R2] [10.0.34.1 (R3)] |

R4s cost to R1 is equal on both routes because the default reference bandwidth in OSPF was not changed. Therefore a FastEthernet and Gigabit Ethernet link have the same cost.

## What I learned
OSPF routers form adjacencies only on interfaces where OSPF is enabled and not passive, so passive interfaces are used on loopbacks and user-facing links to advertise networks without sending hellos to hosts. An ASBR injects routes learned outside OSPF, and `default-information originate` shares a default route with the whole area, so only R1 needs to know about the ISP.

## Configs
See the [configs](configs) folder for the running configs
