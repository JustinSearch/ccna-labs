# Lab: Extended ACLs

**Tool:** Packet Tracer

## Goal
Use extended ACLs to enforce three access policies that standard ACLs can't express, since they require matching on protocol and destination port as well as source and destination address.

## Topology
<img width="1727" height="902" alt="08-Extended-ACL-config topology" src="https://github.com/user-attachments/assets/73bece33-7545-4009-95c9-f6e7e016092d" />

## Policies
1. Hosts in 172.16.2.0/24 can't communicate with PC1 (172.16.1.1)
2. Hosts in 172.16.1.0/24 can't access the DNS service on SRV1 (192.168.1.100)
3. Hosts in 172.16.2.0/24 can't access the HTTP or HTTPS services on SRV2 (192.168.2.100)

## What I configured
- Applied extended ACLs on R1, **inbound on the interface closest to the source**. Extended ACLs match on destination and port, so they can sit near the source without blocking unrelated traffic, which also stops denied traffic from crossing the network.
- **R1 G0/1 (inbound):** one ACL covering policies 1 and 3. It denies 172.16.2.0/24 to PC1, denies TCP 80 and 443 from 172.16.2.0/24 to SRV2, and permits everything else.
- **R1 G0/0 (inbound):** one ACL for policy 2. It denies DNS (UDP and TCP 53) from 172.16.1.0/24 to SRV1, and permits everything else.
- Combined policies 1 and 3 in a single ACL because only one ACL can be applied per interface, per direction.

## Key commands
```
! R1 G0/1 (inbound): policies 1 and 3
ip access-list extended FROM_172.16.2.0/24
 deny ip 172.16.2.0 0.0.0.255 host 172.16.1.1
 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 80
 deny tcp 172.16.2.0 0.0.0.255 host 192.168.2.100 eq 443
 permit ip any any
interface g0/1
 ip access-group FROM_172.16.2.0/24 in

! R1 G0/0 (inbound): policy 2
ip access-list extended FROM_172.16.1.0/24
 deny udp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
 deny tcp 172.16.1.0 0.0.0.255 host 192.168.1.100 eq 53
 permit ip any any
interface g0/0
 ip access-group FROM_172.16.1.0/24 in
```

## How I verified it
- `show access-lists` to confirm each entry and watch the match counters rise during testing
- `show ip interface g0/0` and `show ip interface g0/1` to confirm which ACL is applied and in which direction
- Tested each policy, and confirmed that unrelated traffic still works

## What I learned
Extended ACLs match protocol, source, destination, and port, so they're placed as close to the source as possible, unlike standard ACLs, which go near the destination. Rules are checked top to bottom and stop at the first match, so specific denies go before the final `permit ip any any`, and the implicit deny would otherwise drop everything else. Only one ACL is allowed per interface, per protocol, per direction, so multiple policies for the same traffic path have to be merged into one ACL. Blocking a service means matching the right protocol and port, such as DNS on both UDP and TCP 53.

## Configs
See the [configs](configs) folder for the running configs.
