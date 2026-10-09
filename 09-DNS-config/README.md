# Lab: DNS Configuration and Name Resolution

**Tool:** Packet Tracer

## Goal
Configure a default route to the Internet, point hosts and the router at a DNS server, create local host entries on the router, and use Packet Tracer's Simulation Mode to analyze how a DNS lookup works step by step.

## Topology
<img width="1093" height="674" alt="09-DNS-config" src="https://github.com/user-attachments/assets/3ca07a02-f383-4fbb-a4b6-2173b9efad79" />

## What I configured
- Configured a default route on R1 pointing to the Internet router
- Set PC1, PC2, and PC3 to use 1.1.1.1 as their DNS server (and set IP, mask, and default gateway 192.168.0.254)
- Set R1 to use 1.1.1.1 as its DNS server with `ip name-server`
- Created static host entries on R1 for R1, PC1, PC2, and PC3 with `ip host`
- Pinged PC1 by name from R1 to confirm the local host table works
- Used Simulation Mode to capture and analyze a ping to youtube.com from PC1

## Key commands
```
ip route 0.0.0.0 0.0.0.0 203.0.113.2
ip name-server 1.1.1.1
ip host R1 192.168.0.254
ip host PC1 192.168.0.1
ip host PC2 192.168.0.2
ip host PC3 192.168.0.3
```
## How I verified it
- `show ip route` on R1 to confirm the default route (`S*`)
- `show hosts` on R1 to confirm the static entries and the name server
- `ping PC1` from R1, resolving the name from the local host table
- `ping youtube.com` from PC1, resolving through the DNS server at 1.1.1.1
- Simulation Mode capture of the DNS lookup (below)

## Simulation Mode analysis

| Step | Protocol | From to To | What happened |
|------|----------|-----------|---------------|
| 1 | [ARP] | PC1 to LAN | PC1 asked for the MAC of its default gateway, 192.168.0.254, before sending off-subnet traffic. |
| 2 | DNS query | PC1 to 1.1.1.1 | PC1 sent a UDP request to port 53 asking for the address of youtube.com |
| 3 | DNS reply | 1.1.1.1 to PC1 | The server returned the IP address for youtube.com |
| 4 | ICMP echo request | PC1 to youtube.com | After resolving the name, PC1 sent the actual ping to the returned IP |
| 5 | ICMP echo reply | youtube.com to PC1 | The server replied, and the ping succeeded |

## What I learned
DNS translates names to IP addresses before any other traffic can be sent, so a ping to a name starts with a DNS query, not an ICMP packet. Queries go out over UDP port 53 to the configured DNS server, and the host then sends traffic to the IP address it got back. A failed name lookup with a working IP ping points to a DNS problem, not a routing problem, which is a quick way to split a fault in half when troubleshooting. Routers can also resolve names using `ip host` entries or a DNS server, and the local table is checked first.

## Configs
See the [configs](configs) folder for the R1 running config.
