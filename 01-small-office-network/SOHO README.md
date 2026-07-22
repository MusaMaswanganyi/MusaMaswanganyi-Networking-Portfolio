# Project 1: Small Office Network Build

## Goal
Design and configure a small office network for a fictional company with three departments (Sales, IT, Guest Wi-Fi), using VLANs for segmentation, a router for inter-VLAN routing, and DHCP for automatic addressing. This project demonstrates core switching and routing fundamentals.

## Scenario
A small business has ~30 employees split across Sales and IT, plus a guest network for visitors that must **not** be able to reach internal resources. Requirements:
- Each department isolated on its own VLAN
- Devices get IP addresses automatically via DHCP
- Departments can reach the internet, but Guest cannot reach Sales/IT
- Simple port security on access ports to prevent unauthorized devices

## Topology
- 1 Router (acts as ISP gateway + router-on-a-stick for inter-VLAN routing)
- 1 Layer 2 Switch (trunk to router, access ports to VLANs)
- 3 VLANs: Sales (10), IT (20), Guest (30)
- End devices (PCs) in each VLAN

*(See diagram - router-on-a-stick topology with switch trunking to three VLANs)*

## IP Addressing Plan

| VLAN | Name  | Subnet            | Gateway        | DHCP Range              |
|------|-------|--------------------|-----------------|--------------------------|
| 10   | Sales | 192.168.10.0/24     | 192.168.10.1    | .10 - .100               |
| 20   | IT    | 192.168.20.0/24     | 192.168.20.1    | .10 - .100               |
| 30   | Guest | 192.168.30.0/24     | 192.168.30.1    | .10 - .100               |

## What I Did

1. Created VLANs 10, 20, and 30 on the switch and assigned access ports to each department.
2. Configured the switch-to-router link as an **802.1Q trunk** carrying all three VLANs.
3. Configured **router-on-a-stick**: created sub-interfaces on the router (one per VLAN) so a single physical link could route between all three networks.
4. Set up **DHCP pools** on the router, one per VLAN, so devices get addresses automatically.
5. Enabled **port security** on access ports (max 1 MAC address per port, violation = shutdown) to stop rogue devices.
6. Added an **access list** to block Guest VLAN traffic from reaching the Sales/IT subnets while still allowing internet access (see Project 3 for the full firewall/ACL breakdown — a lightweight version is included here).

## Key Configs

See [`configs/switch.txt`](configs/switch.txt) and [`configs/router.txt`](configs/router.txt) for the full running configuration.

Switch VLAN + trunk setup (excerpt):
```
vlan 10
 name SALES
vlan 20
 name IT
vlan 30
 name GUEST

interface range fastEthernet 0/1-8
 switchport mode access
 switchport access vlan 10
 switchport port-security
 switchport port-security maximum 1
 switchport port-security violation shutdown

interface fastEthernet 0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
```

Router-on-a-stick sub-interfaces (excerpt):
```
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

interface gigabitEthernet 0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
```

## Testing / Verification
- `show vlan brief` — confirmed correct port-to-VLAN assignment
- `show interfaces trunk` — confirmed trunk carrying all 3 VLANs
- `ipconfig` on each end device — confirmed correct DHCP-assigned IP per VLAN
- `ping` between VLANs — confirmed Sales ↔ IT reachable, Guest blocked from both

## Challenges & What I Learned
- Initially forgot to enable the trunk on the switch side and could only get one VLAN routing at a time — this taught me how critical `switchport trunk allowed vlan` is for router-on-a-stick to work.
- Learned the difference between Layer 2 segmentation (VLANs) and Layer 3 reachability (routing) — VLANs alone don't let devices talk to each other; you need the router sub-interfaces for that.
- Port security taught me how quickly access ports can be secured with minimal config, which is a good "quick win" to mention in interviews.

## Tools Used
Cisco Packet Tracer
