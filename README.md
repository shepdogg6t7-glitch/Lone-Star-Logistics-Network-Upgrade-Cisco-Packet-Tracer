# Lone Star Logistics — Multi-Site Network Upgrade

A three-site hub-and-spoke network designed and configured in Cisco
Packet Tracer. Built as a capstone project, then expanded with a
**deliberate troubleshooting exercise** that simulates a real support
ticket — including root-cause diagnosis and verification of the fix.

## The Scenario

Lone Star Logistics operates three sites across Central Texas:

| Site | Role | LAN |
|------|------|-----|
| **Austin** | Headquarters (hub) | 192.168.10.0/24 |
| **Round Rock** | Sales office (spoke) | 192.168.20.0/24 |
| **San Marcos** | Warehouse (spoke) | 192.168.30.0/24 |

Both spoke sites connect only to Austin (single-homed). All traffic
between Round Rock and San Marcos must transit Austin.

![Network topology](screenshots/01-topology/topology.png)

## What Was Built

| Area | Detail |
|------|--------|
| **Addressing** | 5 subnets across 7 router interfaces |
| **Static routing** | Full reachability between all three LANs |
| **DHCP** | Centralized server at HQ + `ip helper-address` relay on both spokes |
| **Bonus: OSPF** | Area 0 enabled alongside static routes as a self-healing backup |
| **Troubleshooting** | Diagnosed and corrected a broken spoke-to-spoke path |

## The Troubleshooting Story

The project includes a **deliberately broken** network state. A junior
technician's prior configuration left three symptoms:

1. HQ could reach the Warehouse but not Sales
2. Sales could not reach the Warehouse
3. Warehouse could reach HQ but could not send print jobs to Sales

**All three symptoms turned out to be the same root cause:** the spoke
routers were missing routes to the *other spoke's WAN transit subnet*.
Router-originated traffic uses its source interface's subnet as its
source IP — with no route back to that subnet, replies were silently
dropped.

**Fix:** two static routes — one on each spoke.

**The interesting part isn't the fix — it's the diagnostic method:**

- Rule out physical/interface problems with `show ip interface brief`
- Compare routing tables against requirements
- Use `traceroute` to see where the packet dies
- **Use extended ping sourced from a specific interface** to isolate
  whether the fault is outbound or return path

That last step is the trick most people miss. Full writeup in
[04-troubleshooting.md](docs/04-troubleshooting.md).

## Project Structure

### Documentation

| # | Doc | Covers |
|---|-----|--------|
| 01 | [Topology](docs/01-topology.md) | Sites, subnets, design rationale |
| 02 | [Addressing](docs/02-addressing.md) | Interface IP plan and verification |
| 03 | [Static Routing](docs/03-static-routing.md) | Route requirements and configuration |
| 04 | [Troubleshooting](docs/04-troubleshooting.md) | **The broken-path diagnosis (start here if short on time)** |
| 05 | [DHCP](docs/05-dhcp.md) | Scopes, relay, and the `serverPool` pitfall |
| 06 | [OSPF Bonus](docs/06-ospf-bonus.md) | Dynamic routing + administrative distance |
| 07 | [Analysis Questions](docs/07-analysis-questions.md) | ARP, Layer 2 vs 3, OSPF protocol deep-dive |
| 08 | [Interview Prep](docs/08-interview-prep.md) | Conceptual, practical, and behavioral Q&A |

### Files

- **`packet-tracer/Kelvin_S_Project.pkz`** — full topology (open in Cisco Packet Tracer)
- **`sop/Kelvin_Shepherd_SOP_Manual.docx`** — standard operating procedures
- **`Kelvin_S_Project.docx`** — original project writeup
- **`screenshots/`** — 16 curated screenshots organized by topic

## Verification Evidence

Every claim in the docs is backed by a screenshot. Key evidence:

**Cross-branch reachability (the fix, confirmed):**

![Sales PC to Warehouse PC](screenshots/04-troubleshooting/sales-pc-to-warehouse-pc.png)

**Routing tables on all three routers:**

- [ATX-R](screenshots/03-routing/atx-r-show-ip-route.png)
- [RR-R](screenshots/03-routing/rr-r-show-ip-route.png)
- [SM-R](screenshots/03-routing/sm-r-show-ip-route.png)

**DHCP scopes and relay:**

- [DHCP configuration](screenshots/05-dhcp/dhcp-scopes.png)
- [DHCP server IP](screenshots/05-dhcp/dhcp-server-ip.png)

## What I Learned

- **Static routes must exist in both directions.** A one-way route
  produces traffic that arrives but never returns.
- **Router-originated traffic uses the source interface's subnet.** A
  route to the destination LAN is not enough if the destination router
  can't route back to your WAN transit subnet.
- **Extended ping sourced from a specific interface** is the single
  most useful diagnostic for asymmetric routing problems.
- **`traceroute` shows you where the packet dies** — hop 1 responding
  but hop 2 timing out is a strong clue about which direction is broken.
- **DHCP broadcasts need relay** to cross router boundaries. There's no
  way around `ip helper-address` if the client and server are on
  different subnets.
- **Administrative distance decides between routing sources.** OSPF and
  static routes can coexist; the lower AD wins the forwarding decision.
- **Multiple symptoms can be one root cause.** Treating each symptom as
  a separate problem wastes time — look for what they have in common.

## Certification

Completed as part of the **Cisco Networking Academy "Getting Started
with Cisco Packet Tracer"** course.

![Certificate](screenshots/certificate.png)

## Contact

- GitHub: [@shepdogg6t7-glitch](https://github.com/shepdogg6t7-glitch)
- Portfolio: [wazuh-home-lab](https://github.com/shepdogg6t7-glitch/wazuh-home-lab) — companion security operations lab

---

*Built in Cisco Packet Tracer. All screenshots and configurations
included in this repository.*
