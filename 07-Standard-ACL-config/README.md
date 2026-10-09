# Lab: Standard ACLs (Numbered and Named)

**Tool:** Packet Tracer

## Goal
Configure OSPF between two routers for full connectivity, then use standard numbered ACLs on R1 and standard named ACLs on R2 to enforce four network access policies.

## Topology
<img width="1381" height="753" alt="07-Standard-ACL-config topology" src="https://github.com/user-attachments/assets/633f18e8-604c-4451-9c46-0983f549e2e9" />

## Policies
1. Only PC1 and PC3 can access 192.168.1.0/24
2. Hosts in 172.16.2.0/24 can't access 192.168.2.0/24
3. 172.16.1.0/24 can't access 172.16.2.0/24
4. 172.16.2.0/24 can't access 172.16.1.0/24

## What I configured
- Configured IP addresses, the serial link (clock rate on the DCE side), and OSPF on R1 and R2 for full connectivity before adding any ACLs
- **R2 (named standard ACLs):** one ACL applied outbound on G0/0 permitting only PC1 and PC3 to reach 192.168.1.0/24 (policy 1), and one applied outbound on G0/1 blocking 172.16.2.0/24 from 192.168.2.0/24 (policy 2)
- **R1 (numbered standard ACLs):** one ACL applied outbound on G0/1 blocking 172.16.1.0/24 from reaching 172.16.2.0/24 (policy 3), and one applied outbound on G0/0 blocking 172.16.2.0/24 from reaching 172.16.1.0/24 (policy 4)
- Placed each ACL on the interface closest to the destination, because a standard ACL only matches the source address. Placing it near the source would also block that source from everything else.

## Key commands
```
! R1 (numbered)
access-list 1 deny 172.16.1.0 0.0.0.255
access-list 1 permit any
interface g0/1
 ip access-group 1 out

access-list 2 deny 172.16.2.0 0.0.0.255
access-list 2 permit any
interface g0/0
 ip access-group 2 out

! R2 (named)
ip access-list standard TO_192.168.1.0/24
 permit host 172.16.1.1
 permit host 172.16.2.1
interface g0/0
 ip access-group TO_192.168.1.0/24 out

ip access-list standard TO_192.168.2.0/24
 deny 172.16.2.0 0.0.0.255
 permit any
interface g0/1
 ip access-group TO_192.168.2.0/24 out
```
## How I verified it
- `show ip route` to confirm OSPF routes before applying ACLs
- `show access-lists` to confirm each entry and watch the match counters increase during testing
- `show ip interface g0/0` (and g0/1) to confirm which ACL is applied and in which direction
- Ping tests from each host

## What I learned
Standard ACLs match only the source address, so they belong close to the destination. Every ACL ends with an implicit deny, so the named ACL for policy 1 needs no explicit deny line, while the "deny" ACLs need `permit any` at the end to avoid blocking everything else. ACLs are processed top to bottom and stop at the first match, so entry order matters. Wildcard masks are the inverse of subnet masks (a /24 is 0.0.0.255). 

## Configs
See the [configs](configs) folder for the running configs.
