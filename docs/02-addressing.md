# 02 — Interface Addressing

**Objective:** Assign the first usable IP in each subnet to the correct
router interface, bring every interface up, and verify with
`show ip interface brief`.

## Address Plan

| Router | Interface | IP Address | Mask | Connects To |
|--------|-----------|------------|------|-------------|
| ATX-R | Gig0/0 | 192.168.10.1 | /24 | ATX-CSW (HQ LAN) |
| ATX-R | Gig0/1 | 192.168.12.1 | /24 | RR-R (WAN — Sales link) |
| ATX-R | Gig0/2 | 192.168.13.1 | /24 | SM-R (WAN — Warehouse link) |
| RR-R | Gig0/0 | 192.168.12.2 | /24 | ATX-R (WAN) |
| RR-R | Gig0/1 | 192.168.20.1 | /24 | RR-CSW (Sales LAN) |
| SM-R | Gig0/0 | 192.168.13.2 | /24 | ATX-R (WAN) |
| SM-R | Gig0/1 | 192.168.30.1 | /24 | SM-CSW (Warehouse LAN) |

**Convention:** the `.1` address on each interface is the router's own
address in that subnet. The `.2` address on a spoke's WAN interface is
the spoke's side of the point-to-point link back to Austin.

## Sample Configuration (ATX-R)

enable
configure terminal
hostname ATX-R
interface GigabitEthernet0/0
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/1
ip address 192.168.12.1 255.255.255.0
no shutdown
exit
interface GigabitEthernet0/2
ip address 192.168.13.1 255.255.255.0
no shutdown
exit
end
copy running-config startup-config


**Why `no shutdown` matters:** router interfaces are administratively
down by default. Without `no shutdown`, an interface shows `up/down` or
`administratively down` in `show ip interface brief` and will not pass
traffic — even if the cable is plugged in and the IP is correct.

## Verification

ATX-R#show ip interface brief
Interface IP-Address OK? Method Status Protocol
GigabitEthernet0/0 192.168.10.1 YES manual up up
GigabitEthernet0/1 192.168.12.1 YES manual up up
GigabitEthernet0/2 192.168.13.1 YES manual up up


**What to look for in the output:**

| Column | Meaning | Correct Value |
|--------|---------|---------------|
| IP-Address | Assigned IP | Matches the address plan |
| OK? | Address valid | YES |
| Method | How it was set | manual |
| Status | Physical link | up |
| Protocol | Line protocol | up |

**The key distinction:** `Status up` means the physical cable is connected.
`Protocol up` means the interface is fully operational at Layer 2/3. Both
must say `up` for the interface to pass traffic.

## Screenshots

- [ATX Server configuration](../screenshots/02-addressing/atx-server-config.png)
- [RR-PC1 configuration](../screenshots/02-addressing/rr-pc1-config.png)
- [ATX-PC1 and ATX-PC2 configuration](../screenshots/02-addressing/atx-pc1-pc2-config.png)

## What I Learned

- The `up/up` vs `up/down` distinction is critical during troubleshooting.
  An interface can be physically connected but still not pass traffic if
  the line protocol is down (e.g., wrong encapsulation, clock rate missing
  on a serial link).
- The `show ip interface brief` command is the fastest single command for
  verifying addressing and interface state across a router.
- Using a documented addressing convention (`.1` for the router, `.2` for
  the spoke WAN side) makes the routing tables easier to reason about later.

## Next

→ [03-static-routing.md](03-static-routing.md) — Static routing between all three sites
