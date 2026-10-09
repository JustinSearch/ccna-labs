# Lab: Switch Hardening and SSH Remote Access

**Tool:** Packet Tracer

## Goal
Configure a new, unconfigured switch (SW2) over its console port, secure console access, enable SSH for remote management with RSA keys, disable Telnet, and restrict remote access to a single management host (PC1).

## Topology
<img width="1184" height="922" alt="11-SSH-config" src="https://github.com/user-attachments/assets/91967b21-dc16-440b-90fe-ce30d5d70790" />

## What I configured
- Connected Laptop1 to SW2's console port and set the hostname to SW2
- Set an enable secret (hashed in the config) and created a local user account
- Configured the VLAN 1 SVI (192.168.2.253/24) and the default gateway (192.168.2.254) so SW2 is reachable from other subnets
- **Console line:** local user authentication and a 5-minute exec timeout
- **Remote access:** set a domain name, generated a 2048-bit RSA key pair, and enabled SSH
- **VTY lines:** local user authentication, a 5-minute exec timeout, and `transport input ssh` so Telnet is rejected
- **Access restriction:** applied a standard ACL to the VTY lines (`access-class`) that permits only PC1 (192.168.1.1)

## Key commands
```
hostname SW2
enable secret <REMOVED>
username jeremy secret <REMOVED>

interface vlan 1
 ip address 192.168.2.253 255.255.255.0
 no shutdown
ip default-gateway 192.168.2.254

line console 0
 login local
 exec-timeout 5 0

ip domain-name [domain]
crypto key generate rsa modulus 2048
ip ssh version 2

access-list 1 permit host 192.168.1.1

line vty 0 15
 login local
 exec-timeout 5 0
 transport input ssh
 access-class 1 in
```

## How I verified it
- `show ip ssh` to confirm SSH is enabled, the version, and the key size
- `show running-config | section line` to confirm console and VTY settings
- `show access-lists` to confirm the ACL entry and watch match counters
- `ssh -l jeremy 192.168.2.253` from PC1, which **succeeded**
- Attempted SSH from a different device (such as R1 or SW1), which was **refused**
- Attempted Telnet to SW2, which was **refused**

## What I learned
SSH needs a hostname, a domain name, and an RSA key pair before it will work, since the key name is built from both. Telnet sends everything in clear text, so `transport input ssh` on the VTY lines forces encrypted sessions. An ACL applied with `access-class` filters who can reach the VTY lines, which is different from `ip access-group` on an interface, which filters traffic passing through. A switch only needs a default gateway to answer devices on other subnets, because it doesn't route. Exec timeouts close idle sessions so a forgotten console doesn't stay logged in.

## Configs
See the [configs](configs) folder for the SW2 running config. Password hashes are replaced with `<REMOVED>`.
