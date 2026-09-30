# 03 — Static Routing

**Objective:** Configure static routes so all three sites can reach each
other's LANs. Spoke sites are single-homed, so every route to a remote
LAN goes through Austin.

## Why Static Routing Here

Both spoke sites have exactly one path off-site (the WAN link to Austin).
There's no redundant path, no alternate next-hop, and no expectation of
the topology changing. Static routing is:

- **Predictable** — the routes are deterministic
- **Simple** — 2 routes on the hub, 3 on each spoke
- **Resource-light** — no routing protocol overhead

Dynamic routing is added as a bonus in [06-ospf-bonus.md](06-ospf-bonus.md),
but static routes are what actually forward traffic in this design.

## Route Requirements

For full site-to-site reachability, each router needs to know how to reach:

| Router | Needs routes to | Via |
|--------|-----------------|-----|
| ATX-R (hub) | Sales LAN (192.168.20.0/24) | RR-R's WAN interface |
| ATX-R (hub) | Warehouse LAN (192.168.30.0/24) | SM-R's WAN interface |
| RR-R (Sales) | HQ LAN (192.168.10.0/24) | ATX-R's WAN interface |
| RR-R (Sales) | Warehouse LAN (192.168.30.0/24) | ATX-R (transit) |
| RR-R (Sales) | Warehouse WAN transit (192.168.13.0/24) | ATX-R (transit) |
| SM-R (Warehouse) | HQ LAN (192.168.10.0/24) | ATX-R's WAN interface |
| SM-R (Warehouse) | Sales LAN (192.168.20.0/24) | ATX-R (transit) |
| SM-R (Warehouse) | Sales WAN transit (192.168.12.0/24) | ATX-R (transit) |

**The last two rows in each spoke are the subtle ones.** They're the
routes that turned the trouble-ticket scenario from broken to working —
see [04-troubleshooting.md](04-troubleshooting.md) for the full story.

## Configuration

### ATX-R (Austin Headquarters)
# 03 — Static Routing

**Objective:** Configure static routes so all three sites can reach each
other's LANs. Spoke sites are single-homed, so every route to a remote
LAN goes through Austin.

## Why Static Routing Here

Both spoke sites have exactly one path off-site (the WAN link to Austin).
There's no redundant path, no alternate next-hop, and no expectation of
the topology changing. Static routing is:

- **Predictable** — the routes are deterministic
- **Simple** — 2 routes on the hub, 3 on each spoke
- **Resource-light** — no routing protocol overhead

Dynamic routing is added as a bonus in [06-ospf-bonus.md](06-ospf-bonus.md),
but static routes are what actually forward traffic in this design.

## Route Requirements

For full site-to-site reachability, each router needs to know how to reach:

| Router | Needs routes to | Via |
|--------|-----------------|-----|
| ATX-R (hub) | Sales LAN (192.168.20.0/24) | RR-R's WAN interface |
| ATX-R (hub) | Warehouse LAN (192.168.30.0/24) | SM-R's WAN interface |
| RR-R (Sales) | HQ LAN (192.168.10.0/24) | ATX-R's WAN interface |
| RR-R (Sales) | Warehouse LAN (192.168.30.0/24) | ATX-R (transit) |
| RR-R (Sales) | Warehouse WAN transit (192.168.13.0/24) | ATX-R (transit) |
| SM-R (Warehouse) | HQ LAN (192.168.10.0/24) | ATX-R's WAN interface |
| SM-R (Warehouse) | Sales LAN (192.168.20.0/24) | ATX-R (transit) |
| SM-R (Warehouse) | Sales WAN transit (192.168.12.0/24) | ATX-R (transit) |

**The last two rows in each spoke are the subtle ones.** They're the
routes that turned the trouble-ticket scenario from broken to working —
see [04-troubleshooting.md](04-troubleshooting.md) for the full story.

## Configuration

### ATX-R (Austin Headquarters)

ip route 192.168.20.0 255.255.255.0 192.168.12.2
ip route 192.168.30.0 255.255.255.0 192.168.13.2

### RR-R (Round Rock Sales Office)

ip route 192.168.10.0 255.255.255.0 192.168.12.1
ip route 192.168.30.0 255.255.255.0 192.168.12.1
ip route 192.168.13.0 255.255.255.0 192.168.12.1


### SM-R (San Marcos Warehouse)

ip route 192.168.10.0 255.255.255.0 192.168.13.1
ip route 192.168.20.0 255.255.255.0 192.168.13.1
ip route 192.168.12.0 255.255.255.0 192.168.13.1


## Summary Table

| Router | Total Static Routes | Reaches HQ LAN | Reaches Other Branch LAN | Reaches Other Branch WAN |
|--------|--------------------:|:--------------:|:------------------------:|:------------------------:|
| ATX-R | 2 | local | Yes | n/a (directly connected) |
| RR-R | 3 | Yes | Yes (via HQ) | Yes (via HQ) |
| SM-R | 3 | Yes | Yes (via HQ) | Yes (via HQ) |

## Verification

ATX-R#show ip route
Gateway of last resort is not set

192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C 192.168.10.0/24 is directly connected, GigabitEthernet0/0
L 192.168.10.1/32 is directly connected, GigabitEthernet0/0
192.168.12.0/24 is variably subnetted, 2 subnets, 2 masks
C 192.168.12.0/24 is directly connected, GigabitEthernet0/1
L 192.168.12.1/32 is directly connected, GigabitEthernet0/1
192.168.13.0/24 is variably subnetted, 2 subnets, 2 masks
C 192.168.13.0/24 is directly connected, GigabitEthernet0/2
L 192.168.13.1/32 is directly connected, GigabitEthernet0/2
S 192.168.20.0/24 [1/0] via 192.168.12.2
S 192.168.30.0/24 [1/0] via 192.168.13.2


**Reading the output:**

- `C` = directly connected network
- `L` = local interface address (/32)
- `S` = static route
- `[1/0]` = administrative distance 1 (static), metric 0
- `via X.X.X.X` = next-hop IP

## Screenshots

- [ATX-R routing table](../screenshots/03-routing/atx-r-show-ip-route.png)
- [RR-R routing table](../screenshots/03-routing/rr-r-show-ip-route.png)
- [SM-R routing table](../screenshots/03-routing/sm-r-show-ip-route.png)

## What I Learned

- Static routes must be configured in **both directions**. If Austin has
  a route to Sales but Sales doesn't have a route to Austin, packets make
  it to Sales but the reply never comes back.
- **A route to a subnet is not enough.** Router-originated traffic uses
  the *source interface's subnet* as its source IP. If the destination
  router has no route back to that source subnet, replies are silently
  dropped.
- `show ip route` is the first command to run when troubleshooting
  reachability between subnets.

## Next

→ [04-troubleshooting.md](04-troubleshooting.md) — Diagnosing and correcting the broken spoke-to-spoke path
