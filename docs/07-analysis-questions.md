# 07 — Protocol Analysis Questions

**Objective:** Answer eight analysis questions on ARP, Layer 2 vs Layer 3
addressing, and OSPF behavior, using packet captures from the running
topology to justify each answer.

## Part 1 — Address Resolution Protocol (ARP)

### Question 1: Why does a router need ARP before sending a ping?

**Answer:** Ethernet frames can only be delivered using MAC addresses, not
IP addresses — the Layer 2 header has no field for a destination IP.
Before a router (or PC) can build a frame to carry an ICMP Echo Request,
it needs to know the MAC address of the next hop. If that MAC isn't
already in the ARP cache, the device sends an ARP Request to discover it
before the ICMP packet can be encapsulated and transmitted.

**Why this matters:** the sender knows the destination's *IP* (from the
user's command or a routing decision), but Layer 2 forwarding needs a
*MAC*. ARP is the bridge between Layer 3 knowledge and Layer 2 delivery.

### Question 2: What destination MAC does an ARP Request use, and why?

**Answer:** `FFFF.FFFF.FFFF` — the Ethernet broadcast address.

The sender doesn't yet know which specific device owns the target IP, so
it can't address the frame to one host. Broadcasting lets every device on
the local segment receive it; only the device that actually owns that IP
replies (via unicast) with its real MAC address.

**Evidence from the PDU walkthrough:**

- **DEST ADDR:** `FFFF.FFFF.FFFF`
- **OPCODE:** `0x0001` (ARP Request)
- **TARGET MAC:** `0000.0000.0000` (unknown / being resolved)

## Part 2 — Layer 2 vs Layer 3 Addressing (the "hop-by-hop" rule)

### Question 3: What happens to Source and Destination IP addresses at each router hop?

**Answer:** Nothing — they stay **exactly the same** at every hop.

Between the inbound and outbound PDUs at each router:

- Source IP: `192.168.20.10` (unchanged)
- Destination IP: `192.168.30.10` (unchanged)

Layer 3 addressing identifies the original sender and the ultimate
destination **end-to-end** and is never rewritten by an intermediate
router. Only the TTL field changes (decremented by 1 per hop — see
Question 5).

### Question 4: What happens to Source and Destination MAC addresses between Inbound and Outbound frames?

**Answer:** They **change completely** at every router.

- **Inbound (RR-R receives from RR-PC1):**
  - SRC MAC: `0001.6390.8098` (RR-PC1's NIC)
  - DST MAC: `0001.9780.5202` (RR-R's LAN interface)
- **Outbound (RR-R sends toward Austin):**
  - SRC MAC: `0001.9780.5201` (RR-R's WAN interface)
  - DST MAC: `0010.111E.7D02` (ATX-R's WAN interface)

RR-R makes its routing decision, then discards the original Ethernet
header entirely and builds a new one for the next hop.

**This is the "hop-by-hop" rule:** Layer 2 addressing is rewritten at
every router, while the Layer 3 IP addressing underneath rides along
unchanged.

### Question 5: What happens to the TTL value as a packet passes through routers?

**Answer:** TTL decrements by **exactly 1 at every router hop**.

Traced through the network:

| Point | TTL |
|-------|----:|
| Leaves RR-PC1 | 128 |
| Arrives at RR-R | 128 |
| Leaves RR-R | 127 |
| Arrives at ATX-R | 127 |
| Leaves ATX-R | 126 |
| Arrives at SM-R | 126 |
| Leaves SM-R | 125 |
| Arrives at destination SM-PC | 125 |

**Why TTL exists:** so a packet caught in a routing loop is eventually
discarded (when TTL reaches 0) rather than circulating the network
forever. This is the mechanism that makes `traceroute` work — each
successive probe uses an incrementing TTL, causing each router in the
path to send back an ICMP Time Exceeded message.

## Part 3 — OSPF Protocol Behavior

### Question 6: What is the destination IP of an OSPF Hello packet, and what does it signify?

**Answer:** `224.0.0.5` — the reserved **"All OSPF Routers"** multicast
address.

Every OSPF-enabled router listens for this address on its OSPF-active
interfaces. Multicast lets a router announce itself and discover
neighbors without needing to already know their individual IP addresses —
any OSPF speaker on the segment receives and processes the Hello.

**Evidence from the PDU capture:**

- **DST IP:** `224.0.0.5`
- **TTL:** 1 (OSPF Hello packets don't leave the local segment)
- **Multicast MAC:** `0100.5E00.0005` (multicast mapping for that IP)

### Question 7: Does OSPF use TCP or UDP for transport?

**Answer:** **Neither.**

The IP header's protocol field reads `0x59` (decimal 89), which is OSPF's
own dedicated IP protocol number. OSPF rides **directly on top of IP**
rather than using a Layer 4 transport protocol such as TCP (6) or UDP (17).

**Why this matters:** OSPF is a routing protocol, not an application. It
needs efficient, periodic, connectionless communication — TCP would add
unnecessary overhead, and UDP would add a layer of abstraction that OSPF
doesn't need.

### Question 8: Why don't we see OSPF Hello packets on the LAN-facing interfaces?

**Answer:** OSPF only sends Hellos on interfaces that face another router
capable of forming an adjacency.

The LAN-facing interfaces connect only to end-user PCs and servers, none
of which run OSPF or can become a neighbor. Hellos there would serve no
purpose. Marking those interfaces **passive** suppresses this multicast
traffic on the LAN as a deliberate best practice.

**Why suppress it:**

- **Reduces unnecessary broadcast/multicast load** on end-user segments
- **Prevents a rogue device from forming a false OSPF adjacency** — an
  attacker who plugs into the LAN and runs OSPF could otherwise become
  a neighbor and inject false routes into the network

## What I Learned

- **Layer 2 and Layer 3 addresses have completely different scopes.**
  Layer 2 addresses are local to each link and change every hop. Layer 3
  addresses are end-to-end and never change.
- **ARP is the glue between the two.** It resolves a known Layer 3
  address into the Layer 2 address needed for actual delivery.
- **TTL is a safety mechanism** that also happens to make `traceroute`
  possible.
- **OSPF is its own IP protocol (89)**, not a TCP or UDP application.
  Understanding the transport layer matters when reading packet captures.
- **Multicast addresses are purpose-built.** `224.0.0.5` is defined
  specifically as "all OSPF routers" — routers listen for it without
  any prior configuration or discovery.

## Next

→ [08-interview-prep.md](08-interview-prep.md) — Sample interview questions drawn from this project





































