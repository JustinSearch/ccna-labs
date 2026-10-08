# Lab: OSPF Troubleshooting

**Tool:** Packet Tracer

## Goal
Add and configure a new serial link between R1 and R2, then find and fix several OSPF faults in a pre-configured single-area network: a missing route, a failed neighbor adjacency, and no external connectivity.

## Topology
<img width="1206" height="780" alt="06-OSPF-troubleshooting topology" src="https://github.com/user-attachments/assets/f8369bff-360a-446e-a90f-5d35c7416bbe" />

## Part 1: New serial link between R1 and R2
**What I configured**
- Configured the serial link addresses on R1 and R2 and brought the interfaces up
- Set the clock rate to 128000 on the DCE side (R1)
- Enabled OSPF on the new link and on R1's and R2's other interfaces in area 0

**How I verified it**
- `show controllers s0/0/0` to identify which end is the DCE
- `show ip interface brief` to confirm the interface was up/up
- `show ip ospf neighbor` to confirm the R1-R2 adjacency reached FULL

## Part 2: Only R3 has a route to 10.0.2.0/24
- **Symptom:** Other routers had no route to 10.0.2.0/24
- **How I investigated:** I ran `show ip route` on other routers, then `show ip ospf interface g0/1` on R3 & R4
- **Root cause:** The link between R3 and R4 was configured as a point-to-point on R3 and a broadcast on R4
- **Fix:** Removed the point-to-point network configuration on R3s g0/1 interface with `no ip ospf network point-to-point` 
- **Verified with:** The route appearing in `show ip route` on other routers

## Part 3: R2 and R4 won't become OSPF neighbors with R5
- **Symptom:** R2 & R4 wouldn't become neighbors with R5
- **How I investigated:** I ran `show ip ospf interface g0/0` on R2, R4, and R5, comparing area, timers, network type, and subnet mask
- **Root cause:** Hello and dead timers were different on R5
- **Fix:** I set the timers to default on R5s g0/0 interface with `ip ospf hello-interval 10` and `ip ospf dead-interval 40`
- **Verified with:** `show ip ospf neighbor` showing FULL adjacencies. On a multi-access segment like this one, one router becomes the DR and another the BDR, and the rest stay at 2-WAY

## Part 4: PC1 and PC2 can't ping the external server 8.8.8.8
- **Symptom:** Pings from PC1 & PC2 to 8.8.8.8 failed
- **How I investigated:** Traced hop by hop with `traceroute`, checked for a default route in each router's table, checked R5
- **Root cause:** R5 was not configured with a default route or as an ASBR
- **Fix:** Configured a default route on R5 and made it an ASBR with `default-information originate`
- **Verified with:** Successful ping from PC1 and PC2 to 8.8.8.8, default route visible on R1 through R4

## Troubleshooting approach
1. Check the layer 1/2 status of the interfaces (`show ip interface brief`)
2. Check OSPF adjacencies (`show ip ospf neighbor`)
3. Check which interfaces and networks OSPF is enabled on (`show ip ospf interface brief`, `show ip protocols`)
4. Check routing tables hop by hop (`show ip route`)
5. Test end to end with ping and traceroute

## What I learned
OSPF can be interesting to troubleshoot, neighbors can still be in a full adjacency even when they aren't functioning properly. The hardest fix to find was the route between R3 and R4 having a network type mismatch because of this fact.

## Configs
See the [configs](configs) folder for the running configs
