# 06 — Bonus: OSPF Dynamic Routing

**Objective:** Configure OSPF Area 0 on all three routers in addition to
the required static routes, then verify OSPF convergence and explain why
static routes remain the routes actually used for forwarding.

## Why Run OSPF Alongside Static Routes

Static routes in this network are sufficient. But OSPF adds value as a
demonstration of dynamic routing and would take over automatically if
the static routes were ever removed or misconfigured:

- **Self-healing** — if a link fails, OSPF recalculates within seconds
- **No manual updates** — new subnets are advertised automatically
- **Standard practice** — most production networks run a dynamic
  protocol, not static routes

Configuring both shows that they can coexist — and demonstrates how
administrative distance resolves the conflict.

## Configuration

# 06 — Bonus: OSPF Dynamic Routing

**Objective:** Configure OSPF Area 0 on all three routers in addition to
the required static routes, then verify OSPF convergence and explain why
static routes remain the routes actually used for forwarding.

## Why Run OSPF Alongside Static Routes

Static routes in this network are sufficient. But OSPF adds value as a
demonstration of dynamic routing and would take over automatically if
the static routes were ever removed or misconfigured:

- **Self-healing** — if a link fails, OSPF recalculates within seconds
- **No manual updates** — new subnets are advertised automatically
- **Standard practice** — most production networks run a dynamic
  protocol, not static routes

Configuring both shows that they can coexist — and demonstrates how
administrative distance resolves the conflict.

## Configuration

! ATX-R
router ospf 1
network 192.168.10.0 0.0.0.255 area 0
network 192.168.12.0 0.0.0.255 area 0
network 192.168.13.0 0.0.0.255 area 0

! RR-R
router ospf 1
network 192.168.20.0 0.0.0.255 area 0
network 192.168.12.0 0.0.0.255 area 0

! SM-R
router ospf 1
network 192.168.30.0 0.0.0.255 area 0
network 192.168.13.0 0.0.0.255 area 0

**Note on the wildcard masks:** `0.0.0.255` means "match the first three
octets, ignore the last." This is an inverted subnet mask — OSPF uses
wildcard masks instead of subnet masks, and the conversion is:
`255.255.255.0` → `0.0.0.255`.

## Verification — Neighbors

ATX-R#show ip ospf neighbor
Neighbor ID Pri State Dead Time Address Interface
192.168.30.1 1 FULL/BDR 00:00:39 192.168.13.2 GigabitEthernet0/2
192.168.20.1 1 FULL/DR 00:00:34 192.168.12.2 GigabitEthernet0/1


Austin shows **two neighbors** (RR-R and SM-R) — expected, since it's the
hub. Each spoke shows **one neighbor** (Austin) — expected, since spokes
are single-homed.

**Reading the output:**

- `Neighbor ID` = the OSPF router-ID of the neighbor (usually its highest
  interface IP or a loopback)
- `State` = adjacency state; `FULL` means the LSDBs are synchronized
- `Dead Time` = seconds until the neighbor is declared down if no Hello
  is received (default 40s, refreshed by each Hello every 10s)
- `Address` = the neighbor's interface IP on the shared subnet

## The AD Puzzle — Why `show ip route ospf` Returns Nothing

ATX-R#show ip route ospf
(no output)

**This is expected and correct.** Here's why:

Cisco routers select routes based on **administrative distance (AD)**:

| Source | Default AD |
|--------|-----------:|
| Directly connected | 0 |
| Static route | 1 |
| OSPF | 110 |

When multiple routing sources know about the same destination, **the
lower AD wins.** So a static route to `192.168.20.0/24` (AD 1) always
beats an OSPF-learned route to the same destination (AD 110).

**Result:** OSPF computes those routes and installs them in its own
database, but the forwarding table only gets the static route.

## Proving OSPF Actually Converged

The absence of OSPF routes in the routing table doesn't mean OSPF failed.
It means static routes outrank them. To prove OSPF is working, look at
the LSDB instead:

ATX-R#show ip ospf database
OSPF Router with ID (192.168.13.1) (Process ID 1)

Link ID ADV Router Age Seq# Checksum
192.168.20.1 192.168.20.1 640 0x80000003 0x0098402
192.168.13.1 192.168.13.1 440 0x80000005 0x008d633
192.168.30.1 192.168.30.1 440 0x80000003 0x0056632

**Three router LSAs** — one from each router. OSPF has learned the full
topology. If you were to remove the static routes, OSPF-learned routes
would immediately appear in the routing table (because there'd be no
lower-AD route to compete with them).

## Confirming That Claim

If you want to verify this behavior directly:

ATX-R(config)#no ip route 192.168.20.0 255.255.255.0 192.168.12.2

After removing the static route, `show ip route` will now show:

ATX-R(config)#ip route 192.168.20.0 255.255.255.0 192.168.12.2


## Screenshots

- [SM-R routing table showing OSPF behavior](../screenshots/06-ospf/sm-r-routes.png)

## What I Learned

- **Administrative distance is what actually decides between routing
  sources.** Two sources can both know about a destination; the lower AD
  determines which one gets installed.
- **`show ip route ospf` returning nothing is not a failure.** It means
  static routes are winning — which, in this case, is the intended design.
- **The OSPF LSDB is separate from the routing table.** To verify OSPF
  converged, check `show ip ospf neighbor` and `show ip ospf database`,
  not just the routing table.
- **Running two routing sources simultaneously is a real production
  pattern** — often during a migration from static to dynamic, or when
  static routes are kept as a deterministic backup.

## Next

→ [07-analysis-questions.md](07-analysis-questions.md) — Deep dive into ARP, Layer 2 vs 3, and OSPF behavior


















