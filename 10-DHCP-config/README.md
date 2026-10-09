# Lab: DHCP Server, Client, and Relay Agent

**Tool:** Packet Tracer

## Goal
Configure R2 as a DHCP server with three pools, make R1's WAN interface a DHCP client, and configure R1 as a DHCP relay agent so hosts on a different subnet can get addresses from a server that isn't on their LAN.

## Topology
<img width="1148" height="897" alt="10-DHCP-config topology" src="https://github.com/user-attachments/assets/7e10cb58-7823-402f-ade3-7d2f87a3f00c" />

## DHCP pool design (on R2)
| Pool | Network | Excluded | Default gateway | DNS | Domain |
|------|---------|----------|-----------------|-----|--------|
| POOL1 | 192.168.1.0/24 | .1 to .10 | 192.168.1.1 (R1) | 8.8.8.8 | jeremysitlab.com |
| POOL2 | 192.168.2.0/24 | .1 to .10 | 192.168.2.1 (R2) | 8.8.8.8 | jeremysitlab.com |
| POOL3 | 203.0.113.0/30 | .1 | | | |

## What I configured
- Excluded the reserved addresses (192.168.1.1 to .10, 192.168.2.1 to .10, and 203.0.113.1) so the server doesn't hand them out
- Created the three DHCP pools on R2 with the network, default gateway, DNS server, and domain name from the table above
- Set R1's G0/0 interface to obtain its address by DHCP, and confirmed it received **203.0.113.2/30** from POOL3
- Configured R1 as a DHCP relay agent on its LAN interface (G0/1), pointing to R2's address on the link (203.0.113.1)
- Set PC1 and PC2 to request addresses by DHCP and confirmed they received addresses from the correct pools

## Key commands
```
! R2 (DHCP server)
ip dhcp excluded-address 192.168.1.1 192.168.1.10
ip dhcp excluded-address 192.168.2.1 192.168.2.10
ip dhcp excluded-address 203.0.113.1

ip dhcp pool POOL1
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8
 domain-name jeremysitlab.com

ip dhcp pool POOL2
 network 192.168.2.0 255.255.255.0
 default-router 192.168.2.1
 dns-server 8.8.8.8
 domain-name jeremysitlab.com

ip dhcp pool POOL3
 network 203.0.113.0 255.255.255.252

! R1 (DHCP client on WAN, relay on LAN)
interface g0/0
 ip address dhcp
 no shutdown
interface g0/1
 ip helper-address 203.0.113.1
 no shutdown
```
## How I verified it
- `show ip interface brief` on R1 to confirm G0/0 received an address by DHCP
- `show ip dhcp binding` on R2 to see which addresses were leased and to which devices
- `show ip dhcp pool` on R2 to confirm pool usage
- `show run | section dhcp` on R2 to confirm the excluded addresses and pool settings
- `ipconfig /renew` and `ipconfig` on PC1 and PC2 to confirm each received the right address, gateway, and DNS
- Ping tests: PC1 to PC2, and each PC to its default gateway

## Results
| Device | Address received | Gateway | DNS |
|--------|------------------|---------|-----|
| R1 G0/0 | [203.0.113.2/30] | | |
| PC1 | [192.168.1.12] | [192.168.1.1] | [8.8.8.8] |
| PC2 | [192.168.2.11] | [192.168.2.1] | [8.8.8.8] |

## How the relay works
PC1's DHCP discover is a broadcast, and routers don't forward broadcasts, so it can never reach R2 on its own. With `ip helper-address` set on R1's LAN interface, R1 receives the broadcast and forwards it as a unicast to R2 (203.0.113.1). R2 sees which interface address the request came from (the relay's address on 192.168.1.0/24), picks the matching pool, POOL1, and replies through R1 back to PC1. Without the relay, only hosts on R2's own LAN could get addresses from it.

## What I learned
DHCP uses broadcasts, so a server must either be on the same subnet or reached through a relay agent. The relay command goes on the interface facing the clients, and it points to the server's address. A DHCP server picks a pool based on the subnet the request came from, and excluded addresses keep static devices like routers from being leased out. A router interface can also be a DHCP client, which is how many small-office WAN links get their address from an ISP.

## Configs
See the [configs](configs) folder for the running configs.
