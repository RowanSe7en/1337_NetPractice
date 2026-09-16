# 🌐 NetPractice — The Networking Handbook


<div align="center">

```
███╗   ██╗███████╗████████╗        ██████╗ ██████╗  █████╗  ██████╗████████╗██╗ ██████╗███████╗
████╗  ██║██╔════╝╚══██╔══╝        ██╔══██╗██╔══██╗██╔══██╗██╔════╝╚══██╔══╝██║██╔════╝██╔════╝
██╔██╗ ██║█████╗     ██║           ██████╔╝██████╔╝███████║██║        ██║   ██║██║     █████╗  
██║╚██╗██║██╔══╝     ██║           ██╔═══╝ ██╔══██╗██╔══██║██║        ██║   ██║██║     ██╔══╝  
██║ ╚████║███████╗   ██║  ███████╗ ██║     ██║  ██║██║  ██║╚██████╗   ██║   ██║╚██████╗███████╗
╚═╝  ╚═══╝╚══════╝   ╚═╝  ╚══════╝ ╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝ ╚═════╝   ╚═╝   ╚═╝ ╚═════╝╚══════╝
```

[![42 School](https://img.shields.io/badge/42-brouane-000000?style=for-the-badge&logo=42&logoColor=white)](https://42.fr/)
[![Topic](https://img.shields.io/badge/Topic-TCP%2FIP%20%7C%20Subnetting%20%7C%20Routing-blueviolet?style=for-the-badge)](.)
[![Levels](https://img.shields.io/badge/Levels-10%2F10-success?style=for-the-badge)](.)
[![Interface](https://img.shields.io/badge/Interface-Local%20Web%20Simulator-orange?style=for-the-badge)](.)
[![Status](https://img.shields.io/badge/Status-FINISHED-success?style=for-the-badge)](.)

*Every packet finds its way home, one route at a time.*

</div>

> *A deep dive into IPv4 addressing, subnetting, and routing — built while solving 42's NetPractice project.*

---

## 📖 Table of Contents

1. [What This Repository Is](#what-this-repository-is)
2. [What NetPractice Actually Teaches](#what-netpractice-actually-teaches)
3. [Mental Model: How a Packet Gets Anywhere](#mental-model-how-a-packet-gets-anywhere)
4. [The OSI Model](#the-osi-model)
5. [The TCP/IP Model](#the-tcpip-model)
6. [IPv4 From First Principles](#ipv4-from-first-principles)
7. [Subnet Masks & CIDR](#subnet-masks--cidr)
8. [Subnetting, Step by Step](#subnetting-step-by-step)
9. [The Complete Subnetting Reference Table](#the-complete-subnetting-reference-table)
10. [Routing: How Packets Choose a Path](#routing-how-packets-choose-a-path)
11. [Default Routes & `0.0.0.0/0`](#default-routes--0000)
12. [ARP, MAC Addresses & Local Delivery](#arp-mac-addresses--local-delivery)
13. [ICMP & Ping](#icmp--ping)
14. [Loopback & `127.0.0.1`](#loopback--127001)
15. [Private vs Public IPs, NAT & CGNAT](#private-vs-public-ips-nat--cgnat)
16. [Non-Overlapping Networks](#non-overlapping-networks)
17. [Inside the NetPractice Simulator](#inside-the-netpractice-simulator)
18. [A Worked Level, Field by Field](#a-worked-level-field-by-field)
19. [My Solving Methodology](#my-solving-methodology)
20. [Troubleshooting Almanac](#troubleshooting-almanac)
21. [Running the Project](#running-the-project)
22. [Repository Contents — The 10 Levels](#repository-contents--the-10-levels)
23. [Cheat Sheet](#cheat-sheet)
24. [Resources](#resources)

---

## What This Repository Is

This repo holds my completed work for **NetPractice**, a 42-curriculum project whose entire goal is to build *intuition* for IPv4 addressing, subnetting, and routing by repairing broken network diagrams until traffic actually flows.

The project ships a small local training interface (`run.sh`) that renders ten progressively harder network topologies. Each level shows hosts, routers, switches, and their interfaces — some fields locked, some editable — and gives you one or more communication goals ("host A must reach host C"). You edit IP addresses, subnet masks, gateways, and routes until the simulator's internal packet-forwarding logic confirms the goal is met.

This README isn't the minimal project-mandated version — it's the reference document I wish I'd had on day one: the theory, the reasoning, the diagrams, and the troubleshooting playbook that actually got me through all ten levels.

### Quick start


Extract the project archive into any folder of your choice
```bash
unzip netpractice.zip -d netpractice
cd netpractice
```
Launch the local web server + open the training interface
```bash
./run.sh
```
If run.sh doesn't work on your setup, start the server manually:
```bash
python3 -m http.server 49242
```
then open this URL in your browser:
```bash
http://localhost:49242
```

---

## What NetPractice Actually Teaches

NetPractice isn't really about memorizing subnet masks. It's about building a working mental model of **three simultaneous questions** a computer asks every time it wants to send a packet:

1. **"Is the destination on my own local network?"** → decided by the subnet mask.
2. **"If not, who do I hand this packet to?"** → decided by the routing table / default gateway.
3. **"How do I physically address a device on my local wire?"** → decided by ARP and MAC addresses.

Every level in this project is really just a variation on: *some device's answer to one of these three questions is wrong — find it and fix it.*

---

## Mental Model: How a Packet Gets Anywhere

Before the theory gets dense, here's the whole story in miniature. Everything below expands on this.

```text
Host wants to send a packet to a destination IP
          │
          ▼
Is the destination inside MY subnet?
   (compare my IP+mask against the destination)
          │
   ┌──────┴──────┐
  YES             NO
   │               │
   ▼               ▼
Send directly    Send to my default gateway
via ARP+Ethernet  (the router handles it from here)
                   │
                   ▼
        Router checks its routing table
                   │
                   ▼
        Finds the MOST SPECIFIC matching route
                   │
                   ▼
        Forwards to that route's next hop
                   │
                   ▼
        Repeat until the packet reaches
        a router directly connected to the
        destination's subnet
                   │
                   ▼
        Delivered locally via ARP+Ethernet
```

That loop — *"local? send directly. Not local? send to gateway, let it decide"* — is the single idea that makes 90% of NetPractice levels solvable.

---

## The OSI Model

The **OSI (Open Systems Interconnection) model** is a 7-layer conceptual map of everything that has to happen for one application on one computer to talk to another application on another computer. NetPractice lives almost entirely in Layers 1–3, but understanding the whole stack makes it obvious *why* those layers behave the way they do.

```text
┌─────────────────────────────────────────────────────────┐
│ L7  Application    │ HTTP, DNS, SSH, FTP...              │
├─────────────────────────────────────────────────────────┤
│ L6  Presentation    │ Encryption, encoding, compression   │
├─────────────────────────────────────────────────────────┤
│ L5  Session         │ Sessions, connections, handshakes   │
├─────────────────────────────────────────────────────────┤
│ L4  Transport       │ TCP, UDP — ports, reliability       │
├─────────────────────────────────────────────────────────┤
│ L3  Network         │ IP — logical addressing, routing    │◄─ NetPractice
├─────────────────────────────────────────────────────────┤
│ L2  Data Link       │ MAC addresses, Ethernet, switches   │◄─ NetPractice
├─────────────────────────────────────────────────────────┤
│ L1  Physical        │ Cables, voltages, radio, bits       │
└─────────────────────────────────────────────────────────┘
```

| Layer | Name | Job | NetPractice-relevant examples |
|---|---|---|---|
| 7 | Application | The actual thing you wanted to do | HTTP request, DNS lookup |
| 6 | Presentation | Format/translate/encrypt data | TLS, character encoding |
| 5 | Session | Open, manage, close a conversation | Login sessions |
| 4 | Transport | End-to-end delivery, ports | TCP, UDP |
| 3 | Network | Logical addressing & routing across networks | **IP, routing tables, routers** |
| 2 | Data Link | Addressing & delivery on one physical link | **MAC addresses, Ethernet, switches** |
| 1 | Physical | Raw bit transmission | Cables, radio waves, connectors |

### Why this matters for the project

- Every **IP address, subnet mask, and route** you touch in NetPractice is **Layer 3**.
- Every **switch and MAC/ARP concept** underneath it is **Layer 2**.
- A **router** is fundamentally a Layer 3 device: it looks at the destination IP and decides where the packet goes next.
- A **switch** is fundamentally a Layer 2 device: it doesn't understand IP addresses at all — it just forwards Ethernet frames based on MAC addresses on a single local network.

**Practical example:** when Host A pings Host C on the same switch, Layers 1–2 do all the work (Ethernet frame, MAC addressing) — Layer 3 routing never gets involved because there's no router in the path. The moment a router has to be crossed, Layer 3 takes over the decision-making, while Layer 2 is still used to actually move the frame across *each individual link*.

---

## The TCP/IP Model

The **TCP/IP model** is the practical, 4-layer model the real Internet actually runs on. It maps cleanly onto OSI but merges some layers together.

```text
┌───────────────────────────┐        ┌─────────────────────────┐
│      OSI (7 layers)       │        │   TCP/IP (4 layers)     │
├───────────────────────────┤        ├─────────────────────────┤
│ Application               │        │                         │
│ Presentation               │  ───►  │      Application        │
│ Session                    │        │                         │
├───────────────────────────┤        ├─────────────────────────┤
│ Transport                  │  ───►  │      Transport          │
├───────────────────────────┤        ├─────────────────────────┤
│ Network                    │  ───►  │      Internet           │
├───────────────────────────┤        ├─────────────────────────┤
│ Data Link                  │  ───►  │                         │
│ Physical                   │        │   Link / Network Access │
└───────────────────────────┘        └─────────────────────────┘
```

| TCP/IP Layer | Equivalent OSI Layers | Protocols involved |
|---|---|---|
| Application | 7, 6, 5 | HTTP, DNS, SSH |
| Transport | 4 | **TCP**, **UDP** |
| Internet | 3 | **IP**, **ICMP**, routing |
| Link / Network Access | 2, 1 | **Ethernet**, **ARP**, physical cabling |

### The protocols that matter here

- **IP (Internet Protocol)** — gives every device a logical address and lets routers decide how to move packets between networks. This is what your IP addresses and subnet masks *are*.
- **ICMP (Internet Control Message Protocol)** — the "diagnostics and error reporting" protocol riding on top of IP. `ping` is built on ICMP.
- **ARP (Address Resolution Protocol)** — translates a Layer 3 IP address into a Layer 2 MAC address so a frame can actually be put on the wire.
- **TCP/UDP** — sit above IP and add ports + (for TCP) reliability. NetPractice doesn't touch these directly, but they're *why* IP connectivity matters in the first place — no working IP routing, no working web browser.
- **Ethernet** — the Layer 2 framing and MAC addressing scheme used on local links, handled invisibly by switches.

---

## IPv4 From First Principles

An IPv4 address identifies a device (technically, an *interface*) on a network. It is **32 bits** long, almost always written as **four decimal octets** separated by dots — "dotted-decimal notation":

```text
192   .   168   .    1   .   132
 8         8          8        8    = 32 bits total
```

Each octet is 8 bits, so it can represent `0`–`255` (`2^8 = 256` values).

In binary, that same address looks like this:

```text
192      168      1        132
11000000.10101000.00000001.10000100
```

The address alone tells you *which device* — but not where the boundary is between "which network this device is on" and "which specific host it is within that network." That's the subnet mask's job.

---

## Subnet Masks & CIDR

### The core idea

A subnet mask is a second 32-bit number that, lined up against the IP address, marks which bits are the **network portion** and which are the **host portion**.

```text
IP:    192.168.1.132
Mask:  255.255.255.192
       └───────────┘└┘
         network     host
```

`255.255.255.192` in binary:

```text
11111111.11111111.11111111.11000000
```

The mask's `1` bits are network bits; the `0` bits are host bits. Here that's **26 ones**, so this mask is written in **CIDR notation** as `/26`.

```text
IP:    11000000.10101000.00000001.10|000100
Mask:  11111111.11111111.11111111.11|000000
                                      ↑
                              network | host
                              (26 bits)(6 bits)
```

### CIDR notation

**CIDR (Classless Inter-Domain Routing)** notation writes the mask as a slash + a number: `IP/prefix-length`. It's shorthand for "the first `N` bits are the network."

```text
192.168.1.10/24
└──── network ────┘└host┘
      24 bits       8 bits

192.168.1.10/26
└────── network ──────┘└host┘
         26 bits        6 bits
```

### Why the prefix length matters

An IP address alone **does not** tell you the size of its network. `192.168.1.10` could equally be part of a `/24`, a `/25`, or a `/26` — each implying a completely different set of neighbors, a different broadcast address, and a different gateway range. The prefix is not optional information; without it, "which devices are local to me" is undefined.

### Mask ↔ Prefix conversion table

| Prefix | Subnet Mask | Network bits | Host bits |
|---|---|---|---|
| /8  | `255.0.0.0` | 8 | 24 |
| /16 | `255.255.0.0` | 16 | 16 |
| /24 | `255.255.255.0` | 24 | 8 |
| /25 | `255.255.255.128` | 25 | 7 |
| /26 | `255.255.255.192` | 26 | 6 |
| /27 | `255.255.255.224` | 27 | 5 |
| /28 | `255.255.255.240` | 28 | 4 |
| /29 | `255.255.255.248` | 29 | 3 |
| /30 | `255.255.255.252` | 30 | 2 |

---

## Subnetting, Step by Step

### Step 1 — Count the host bits

```text
host bits = 32 - prefix length
```

For `/26`: `32 - 26 = 6` host bits.

### Step 2 — Compute the block size (addresses per network)

```text
addresses per network = 2 ^ (host bits)
```

For `/26`: `2^6 = 64` addresses.

### Step 3 — Subtract network + broadcast to get usable hosts

```text
usable hosts = (2 ^ host bits) - 2
```

For `/26`: `64 - 2 = 62` usable host addresses.

> The `-2` is because every subnet reserves its **first address** as the network address (identifies the subnet itself) and its **last address** as the broadcast address (reaches every host on that subnet). Neither can be assigned to a device. (The lone exception is `/31` and `/32`, used for special point-to-point/host-route cases outside this project's scope.)

### Step 4 — Find where a given subnet starts and ends (the fast trick)

The increment between consecutive subnets equals:

```text
256 - (the mask's "interesting" octet value)
```

For `/26`, the interesting octet is `192`:

```text
256 - 192 = 64
```

So `/26` networks always start on multiples of 64:

```text
0, 64, 128, 192
```

### Full worked example: `192.168.1.132/26`

```text
/26
  ↓
26 network bits, 6 host bits
  ↓
2^6 = 64 addresses per block
  ↓
blocks of 64, starting at:
0 – 63
64 – 127
128 – 191   ← .132 falls here
192 – 255
```

`.132` falls inside the `128–191` block, so:

```text
Network:    192.168.1.128
Usable:     192.168.1.129 – 192.168.1.190
Broadcast:  192.168.1.191
```

### More worked examples

**`10.0.0.50/28`**

```text
host bits = 4  →  2^4 = 16 addresses per block
blocks: 0,16,32,48,64...
.50 falls in 48–63
```

```text
Network:    10.0.0.48
Usable:     10.0.0.49 – 10.0.0.62
Broadcast:  10.0.0.63
```

**`172.16.5.200/27`**

```text
host bits = 5  →  2^5 = 32 addresses per block
blocks: 0,32,64,96,128,160,192,224
.200 falls in 192–223
```

```text
Network:    172.16.5.192
Usable:     172.16.5.193 – 172.16.5.222
Broadcast:  172.16.5.223
```

**`138.216.209.66/26`** *(pulled from my own notes/level data)*

```text
host bits = 6  →  64 addresses per block
blocks: 0,64,128,192
.66 falls in 64–127
```

```text
Network:    138.216.209.64
Usable:     138.216.209.65 – 138.216.209.126
Broadcast:  138.216.209.127
```

### How many subnets did I create? (borrowing bits)

If you take a bigger network and cut it into smaller ones, the number of resulting subnets is:

```text
2 ^ (borrowed bits)
```

Example: splitting a `/24` into `/26`s.

```text
borrowed bits = 26 - 24 = 2
2^2 = 4 subnets
```

```text
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

---

## The Complete Subnetting Reference Table

| Prefix | Mask | Host bits | Addresses/block | Usable hosts | Block increment |
|---|---|---|---|---|---|
| /24 | 255.255.255.0   | 8 | 256 | 254 | 256 |
| /25 | 255.255.255.128 | 7 | 128 | 126 | 128 |
| /26 | 255.255.255.192 | 6 | 64  | 62  | 64 |
| /27 | 255.255.255.224 | 5 | 32  | 30  | 32 |
| /28 | 255.255.255.240 | 4 | 16  | 14  | 16 |
| /29 | 255.255.255.248 | 3 | 8   | 6   | 8 |
| /30 | 255.255.255.252 | 2 | 4   | 2   | 4 |

**Remember this:** the "block increment" column is always `256 − mask` for the octet that's being subdivided — this single trick replaces almost all manual binary math on this project.

---

## Routing: How Packets Choose a Path

A **routing table** is a list of **routes**. Each route answers one question: *"if the destination falls inside this network, where do I forward the packet next?"*

```text
Destination        Next Hop
10.0.0.0/24         10.0.0.1
192.168.1.0/24      192.168.1.1
0.0.0.0/0           192.168.1.1     ← default route
```

General route structure:

```text
Destination/Prefix  =>  Next Hop
```

- **Destination** — the network being described (not a single host, a whole range).
- **Next Hop** — the IP address of the next device the packet should be handed to. It's rarely the final destination itself — usually it's "the next router along the path."

### Routing is a decision process, not a lookup table with one answer

```text
Packet arrives with a destination IP
          │
          ▼
Check every route in the table
          │
          ▼
Find every route whose network CONTAINS the destination
          │
          ▼
Among all matches, pick the MOST SPECIFIC
   (the one with the LONGEST prefix / smallest network)
          │
          ▼
Forward to that route's next hop
```

### Why the most-specific match wins ("longest-prefix match")

Imagine a routing table with these four entries, and a packet destined for `10.10.20.55`:

```text
10.0.0.0/8       ← matches (contains 10.10.20.55)
10.10.0.0/16     ← matches (more specific)
10.10.20.0/24    ← matches (even more specific)
0.0.0.0/0        ← matches (matches literally everything)
```

All four technically "contain" the destination. A router doesn't pick the first match or a random one — it picks the entry with the **longest prefix length**, because that's the *most precise description available* of where the destination actually lives. Here, `10.10.20.0/24` wins over the broader `/16` and `/8` entries, and all of them win over the catch-all `/0`.

**Intuition:** think of prefixes as addresses of decreasing precision — "somewhere in this country" (`/8`) vs "somewhere in this city" (`/16`) vs "this exact street" (`/24`). You always trust the most precise address you have.

---

## Default Routes & `0.0.0.0/0`

### What "default" means in networking

**Default** means: *"use this only when nothing more specific tells me what to do."* It's the fallback of last resort.

### Why `0.0.0.0/0` matches everything

```text
0.0.0.0/0
   ↓
0 network bits, 32 host bits
   ↓
no bits are fixed → every possible address satisfies the mask
   ↓
matches every IPv4 destination that exists
```

Since it never loses a longest-prefix comparison against anything more specific, `0.0.0.0/0` is guaranteed to only be used when nothing else applies — which is exactly the behavior you want from a "send everything I don't recognize to my gateway/ISP" rule.

### Example: a host reaching the public internet

A host wants to reach `8.8.8.8`. Its routing table:

```text
10.0.0.0/24          10.0.0.1
192.168.1.0/24       192.168.1.1
0.0.0.0/0            192.168.1.1     ← default
```

```text
Is 8.8.8.8 inside 10.0.0.0/24?      ❌
Is 8.8.8.8 inside 192.168.1.0/24?    ❌
Is there a default route?            ✅ → use it
```

Result: the packet is sent to `192.168.1.1` — typically the local router/gateway, which repeats this same decision process one hop further out, and so on until the packet reaches a router that actually knows a specific route toward `8.8.8.8` (in the real world, this cascades all the way to your ISP and beyond).

### `default` as a shorthand

In this project's interface, some devices show their default route simply as `default => <gateway IP>` rather than spelling out `0.0.0.0/0`. They mean exactly the same thing.

---

## ARP, MAC Addresses & Local Delivery

IP addresses are **logical** (Layer 3) — they can be reassigned, they describe network membership, and routers use them to make forwarding decisions. But a physical Ethernet frame can't actually be delivered on a wire using an IP address; the hardware needs a **MAC address** (Layer 2, burned into the network interface).

**ARP (Address Resolution Protocol)** is the glue between the two: given "I need to reach IP `X` on my local network," ARP answers "the MAC address of the device holding IP `X` is `Y`."

```text
Host A wants to send a frame to 192.168.1.5 (same subnet)
          │
          ▼
Does A already know 192.168.1.5's MAC address?
          │
   ┌──────┴──────┐
  YES             NO
   │               │
   │               ▼
   │        Broadcast ARP request: "Who has 192.168.1.5?"
   │               │
   │               ▼
   │        Owner replies: "192.168.1.5 is at MAC xx:xx:xx..."
   │               │
   └───────┬───────┘
           ▼
   Build Ethernet frame addressed to that MAC
           ▼
   Switch forwards the frame to the correct port
```

ARP only ever operates **within a single local network** — it's how "same-subnet" delivery actually happens after routing decides "this destination is local, don't involve the gateway." A switch's whole job is to look at destination MAC addresses and forward Ethernet frames toward the right port, with zero awareness of IP addresses or subnets.

---

## ICMP & Ping

**ICMP (Internet Control Message Protocol)** is a Layer-3 companion to IP used for diagnostics and error reporting rather than carrying application data. `ping` is the most familiar tool built on it: it sends an ICMP Echo Request and waits for an ICMP Echo Reply.

```text
Host A ──ICMP Echo Request──►  Host B
Host A ◄──ICMP Echo Reply───   Host B
```

In NetPractice, a level's "goal" is essentially: *make sure a simulated ping (or generic forward path) can leave the source, get correctly routed at every hop, and be recognized as arriving at the intended destination.* The simulator's log output narrates this exact journey — "packet accepted," "pass through routing table," "destination IP reached" — which mirrors what a real ICMP Echo Request would experience hop by hop.

---

## Loopback & `127.0.0.1`

**Loopback** means a device sending traffic to *itself*, without that traffic ever touching a physical network interface. IPv4 reserves the whole `127.0.0.0/8` range for this, and `127.0.0.1` — nicknamed **localhost** — is the address almost everyone actually uses.

```text
Your computer
┌───────────────────────────┐
│                            │
│  Browser ──► 127.0.0.1:8080│
│                  │          │
│                  ▼          │
│            Web server       │
│                            │
└───────────────────────────┘
```

Visiting `http://127.0.0.1:8080` means "connect to port 8080 on *this same machine*" — nothing leaves toward a router or a LAN.

**Loopback vs a normal LAN address:**

```text
192.168.1.10   → "me, as seen from my local network"
127.0.0.1      → "me, talking to myself, regardless of any network"
```

Loopback is why NetPractice's own `run.sh` script can spin up a tiny local web server (commonly on `127.0.0.1:<port>`) — the training interface never has to leave your machine.

---

## Private vs Public IPs, NAT & CGNAT

Not every IPv4 address is meant to be reachable from the public internet. **Private ranges** are reserved for use inside local networks and are never supposed to be globally routed:

| Range | CIDR | Typical use |
|---|---|---|
| `10.0.0.0 – 10.255.255.255` | `10.0.0.0/8` | Large private networks, enterprises |
| `172.16.0.0 – 172.31.255.255` | `172.16.0.0/12` | Medium private networks |
| `192.168.0.0 – 192.168.255.255` | `192.168.0.0/16` | Home/small office networks |

Everything outside these blocks (that isn't otherwise reserved) is considered **public** — globally unique and routable across the internet.

**NAT (Network Address Translation)** is what lets a whole private network share a single public IP: a router rewrites the source address of outgoing packets from a private address to its own public one, and reverses the process for replies. Without NAT, every device on a private LAN would be invisible to the outside internet by design (private addresses aren't globally routable).

**CGNAT (Carrier-Grade NAT)** is the same idea applied one layer further out: an ISP applies NAT across *many customers* sharing a pool of public addresses, because IPv4's total address space (~4.3 billion addresses) is not nearly enough for every device on Earth to have its own public IP.

NetPractice's simulated addresses (e.g. `138.216.209.x`, `163.172.250.x`) are fictitious and don't correspond to real allocations, but the private/public distinction is exactly why a level's topology often separates an "internal" side from an "internet-facing" router interface.

---

## Non-Overlapping Networks

A router's interfaces must sit on **non-overlapping** networks — no single IP address may plausibly belong to two different subnets assigned to the same router.

### ✅ No overlap

```text
Network A = 192.168.1.0/25   → range 0–127
Network B = 192.168.1.128/26 → range 128–191

Network A
0 ─────────────── 127

Network B
                  128 ────── 191
```

The ranges don't touch — safe.

### ❌ Overlap

```text
Network A = 192.168.1.0/24   → range 0–255
Network B = 192.168.1.128/26 → range 128–191

Network A
0 ─────────────────────────────────── 255
              ┌──────────┐
Network B     │128 → 191 │
              └──────────┘
```

Network B's entire range sits *inside* Network A's. This is a genuine problem.

### Why it breaks a router

```text
             Router
            /      \
          R1        R2
```

```text
R1 = 192.168.1.1/24     → claims "I can reach 0–255"
R2 = 192.168.1.129/26   → claims "I can reach 128–191"
```

When a packet destined for `192.168.1.150` arrives, **both interfaces' subnets technically contain it**. The router has no unambiguous rule for which interface should own that address — this is exactly the kind of misconfiguration several NetPractice levels are designed to test you on. The fix is always the same: shrink or relocate one of the subnets so the ranges no longer intersect.

---

## Inside the NetPractice Simulator

### What the diagram represents

Each level renders a small fictitious network: **hosts** (end devices, drawn as computers/servers), **routers** (drawn with antenna/box icons, each having multiple interfaces), and sometimes a **switch** (a passive device connecting several hosts on one shared subnet, with no IP configuration of its own — consistent with switches being Layer 2 devices).

### Field types

| Field | Meaning |
|---|---|
| **IP** | The interface's own IPv4 address |
| **Mask** | The subnet mask for that interface, defining its local network's boundaries |
| **Routes** | `Destination => Next Hop` pairs telling the device how to reach non-local networks |
| **`(fixed)`** | A locked field you cannot change — a constraint you must design *around* |
| **Editable (unshaded)** | A field you're expected to fill in or correct |

### Objectives, checking, and logs

Every level states one or more goals, typically "host X needs to communicate with host Y." The **[Check again]** button re-runs the simulator's internal forwarding logic against your current configuration and reports `OK`/`KO` per goal. The **[Get my config]** button exports your current answer as a file you keep for submission. The **log panel** narrates the packet's simulated journey hop by hop:

```text
Forward way: A -> B (125.63.120.193)
on A: packet accepted
on A: destination does not match any interface.
pass through routing table
on A: destination does not match any route      ← the actual bug
```

Reading these logs literally, line by line, tells you *exactly* which device and which decision (interface match vs. routing-table match) failed — this is the single most useful debugging tool the project gives you.

### How to think when solving a level (not just "what to click")

1. A level is broken because **one of the three core questions** (from the [Mental Model](#mental-model-how-a-packet-gets-anywhere) section) has a wrong answer *somewhere along the path* — a wrong mask, an unreachable gateway, or a missing/incorrect route.
2. Don't guess-and-check randomly. Trace the packet's intended journey by hand first (source → gateway → router(s) → destination), and predict at which hop it should logically fail given the *current* configuration.
3. Fixed fields are your anchors — they tell you what the "correct" network layout has to accommodate. Editable fields are your degrees of freedom.
4. Every host needs a gateway that is (a) inside its own subnet and (b) actually configured with a matching IP on the router side.
5. Every router interface needs a mask that produces non-overlapping subnets against every *other* interface on that router.
6. Every router needs a route (often the default `0.0.0.0/0`) covering any destination it doesn't have a directly-connected interface for.

---

## A Worked Level, Field by Field

Here's a representative topology (based on the kind of layout this project uses), fully reasoned through — the same style of table appears in my exported level configs.

```text
Node          Interface   IP Address          Mask                Routes
Host H1       H11         138.216.209.2 (f)   255.255.255.128     0.0.0.0/0 (f) => 138.216.209.1 (f)
Host H2       H21         138.216.209.3       255.255.255.128     default (f)   => 138.216.209.1 (f)
Host H3       H31         138.216.209.66      255.255.255.192     0.0.0.0/0 (f) => 138.216.209.65
Host H4       H41         138.216.209.131 (f) 255.255.255.192 (f) default (f)   => 138.216.209.129 (f)
Router R1     R11         138.216.209.1 (f)   255.255.255.128 (f) 138.216.209.64/26  => 138.216.209.253 (f)
              R12         163.172.250.12 (f)  255.255.255.240 (f) 138.216.209.128/26 => 138.216.209.253 (f)
              R13         138.216.209.254 (f) 255.255.255.252     0.0.0.0/0 (f)      => 163.172.250.1 (f)
Router R2     R21         138.216.209.253 (f) 255.255.255.252 (f) 0.0.0.0/0 (f)      => 138.216.209.254 (f)
              R22         138.216.209.65      138.216.209.192     —
              R23         138.216.209.129     255.255.255.192     —
Internet I    —           —                   —                  138.216.209.0/24 => 163.172.250.12 (f)
Switch S1     —           —                   —                  —
```

*(`(f)` marks fixed fields.)*

**Reasoning through it:**

- **H1 and H2** (`.2` and `.3`) both use mask `255.255.255.128` = `/25` → block size 128 → they belong to network `138.216.209.0/25` (range `.0`–`.127`), whose gateway is fixed at `138.216.209.1`. That IP falls in the same range — ✅ consistent.
- **R1's `R11`** interface is the fixed gateway `138.216.209.1/25` — matches H1/H2's subnet exactly, so H1/H2 can reach R1 directly.
- **H3** (`.66`) claims mask `138.216.209.192` — that's actually malformed as written (it should be a mask like `255.255.255.192`, not another IP-shaped value); this is exactly the kind of typo the simulator's logs will flag. Once corrected to `255.255.255.192` (`/26`, block size 64), `.66` falls in the `64–127` block → network `138.216.209.64/26`, gateway `138.216.209.65` — which matches R2's `R22` interface once *its* mask is likewise corrected to `255.255.255.192`.
- **H4** (`.131`, fixed) with fixed mask `255.255.255.192` (`/26`) falls in the `128–191` block → network `138.216.209.128/26`, gateway `138.216.209.129` — matching R2's `R23` interface.
- **R1's routes** point `138.216.209.64/26` and `138.216.209.128/26` both toward `138.216.209.253` — that's `R2's R21` interface, the point-to-point link between the two routers (`/30`, only 4 addresses, 2 usable — a classic router-to-router link size).
- **R1's `R13`** interface bridges to the fixed `163.172.250.0/28`-ish network toward "Internet I," with a default route out to `163.172.250.1`.
- **The "Internet" node** has a static route back: `138.216.209.0/24 => 163.172.250.12`, telling it how to reach the *entire* internal `.0/24` super-network through R1's public-facing interface.

This single diagram exercises almost every concept in this README at once: mask correction, gateway matching, non-overlapping subnetting, router-to-router links, and both a default route and a specific route coexisting correctly.

---

## My Solving Methodology

```text
 1. Read the topology completely before touching anything
 2. Identify every device: hosts, routers, switches
 3. Identify every interface on every device
 4. Separate fixed fields (constraints) from editable fields (freedom)
 5. For each host, determine its intended subnet from its IP + mask
 6. Calculate that subnet's valid host range, network & broadcast address
 7. Verify each host's gateway is (a) inside its own subnet, (b) matches
    a real router interface IP
 8. For each router-to-router link, verify both sides share one
    correctly-sized subnet (commonly a /30)
 9. Verify every router has a route (often 0.0.0.0/0) covering anything
    not directly connected
10. Check that no two subnets on the same router overlap
11. Trace the required path goal-by-goal, hop by hop, on paper
12. Fix one issue at a time — don't change five fields before re-checking
13. Read the simulator's log after every attempt; it tells you exactly
    which hop and which check failed
14. Re-verify ALL previously-passing goals after any edit (multi-goal
    levels can regress)
15. Export the config once every goal shows OK
```

**Why this order works:** fixed fields are constraints, not obstacles — reading them first tells you the *shape* of the correct answer before you start guessing. Masks and gateways are almost always the root cause in early levels; missing or wrong routes dominate later levels once router-to-router topologies appear.

---

## Troubleshooting Almanac

| Symptom | Likely cause | How to reason about it |
|---|---|---|
| Host cannot reach its own gateway | Gateway IP falls outside the host's subnet range | Recompute the subnet from the host's IP+mask; the gateway must be a valid address *inside* that same range |
| "Gateway is outside the subnet" | Mask too small/large for the addresses involved | Recheck block size (`2^host bits`) against both host IP and gateway IP |
| Wrong subnet mask | Off-by-one prefix length | Recompute host range with the *given* mask and see if it actually contains both endpoints you expect |
| Wrong network address configured | Someone configured a broadcast or non-network address where a network address was expected | Network address = first address of the block; recompute from mask |
| Broadcast address used as a host IP | An editable IP field was set to the last address of its block | Last address in a block is *always* reserved — never assign it to a device |
| Two router interfaces on incompatible networks | Interfaces meant to be on the same link have addresses in different subnets | Both ends of a link must share the exact same network + mask |
| Overlapping subnets | Two subnets on the same router share addresses | See [Non-Overlapping Networks](#non-overlapping-networks) — shrink or relocate one subnet |
| Missing route | A router has no entry (and no default) covering a needed destination | Add either a specific route or a `0.0.0.0/0` default toward the correct next hop |
| Wrong next hop | Route exists but points to an IP that isn't actually a working next router | Next hop must be a real, directly-reachable interface — usually the router on the *other end* of a directly-connected link |
| Incorrect default route | `0.0.0.0/0` points somewhere that doesn't lead toward the rest of the network | Trace what's actually reachable from that next hop; fix or replace it |
| "Correct IP reached but wrong host" | Two devices' IPs got swapped, or a subnet boundary miscalculation routed traffic to a neighboring host instead of the intended one | Recompute exact host ranges; verify the intended target's IP genuinely belongs to the subnet you think it does |
| Packet reaches a router but goes no further | The router lacks a route for the next segment, or the next hop is misconfigured | Check that router's own routing table for the destination — treat it as its own mini-puzzle |
| Host communicates locally but not remotely | Local (same-subnet) delivery uses ARP/switching and needs no gateway — remote delivery does. A missing/incorrect gateway or route only breaks the remote case | Isolate: is the failing destination on the same subnet or not? If not, the gateway/route path is the first suspect |
| Internet-bound communication fails | Missing default route at some hop, or NAT-equivalent boundary misconfigured | Trace the full hop-by-hop path outward; find the first hop where the simulator log shows a failed match |
| Routing loop / circular logic | Two routers each point toward each other as the next hop for the same destination, with neither having an actual path forward | Follow the destination through the table by hand; if you return to a router you've already visited, that's the loop |

**General debugging discipline:** always change one field, re-check, and re-read the log before changing another. Multi-cause bugs are rare in this project — most "broken" levels have exactly one root misconfiguration whose symptom just *looks* complicated.

---

## Running the Project

1. Download and extract the NetPractice project files into any folder.
2. From that folder, run:
   ```bash
   ./run.sh
   ```
   This launches a small local web server and opens the training interface in your default browser.
3. If `run.sh` doesn't work (common on some browser/OS combinations due to local file-loading restrictions), start the server manually:
   ```bash
   python3 -m http.server 49242
   ```
   then open `http://localhost:49242` in your browser (any free port works if `49242` is taken).
4. On the welcome screen, enter your intranet login in the **Training** tab (this is what the grading Moulinette uses to validate your personal configuration), or use the **Evaluation** tab to generate a random configuration suitable for defense practice.
5. Work through each of the 10 levels:
   - Read the topology and the stated goal(s).
   - Edit only the unshaded (non-fixed) fields.
   - Click **Check again** to validate your current configuration against the goal(s).
   - Read the log panel for hop-by-hop diagnostic detail whenever a goal fails.
   - Once every goal for the level shows `OK`, click **Get my config** to export that level's configuration file, then click **Next level**.
6. **Important:** always enter your login before exporting — configs exported without a login are not valid for submission.

### Submission requirements

- Ten exported configuration files — one per level — placed at the **root** of this repository.
- Filenames matching the level they came from, so anyone reviewing this repo can immediately see which file corresponds to which level.

---

## Repository Contents — The 10 Levels

| Level | Config file |
|---|---|
| 1  | `level1.json` |
| 2  | `level2.json` |
| 3  | `level3.json` |
| 4  | `level4.json` |
| 5  | `level5.json` |
| 6  | `level6.json` |
| 7  | `level7.json` |
| 8  | `level8.json` |
| 9  | `level9.json` |
| 10 | `level10.json` |

Each file is the exported configuration proving that level's goal(s) were satisfied in the training interface, generated via **Get my config** as described above.

---

## Cheat Sheet

### Prefix ↔ Mask ↔ Block size ↔ Usable hosts

```text
/24 = 255.255.255.0    → 256 addresses → 254 usable hosts
/25 = 255.255.255.128  → 128 addresses → 126 usable hosts
/26 = 255.255.255.192  →  64 addresses →  62 usable hosts
/27 = 255.255.255.224  →  32 addresses →  30 usable hosts
/28 = 255.255.255.240  →  16 addresses →  14 usable hosts
/29 = 255.255.255.248  →   8 addresses →   6 usable hosts
/30 = 255.255.255.252  →   4 addresses →   2 usable hosts   (classic router-to-router link)
```

### The two formulas that solve almost everything

```text
Addresses per block = 2 ^ (32 - prefix length)
Block increment      = 256 - (mask's relevant octet)
```

### Quick sanity checks before you trust an answer

- [ ] Does the gateway IP actually fall inside the host's computed range?
- [ ] Do both ends of a router-to-router link share the exact same network?
- [ ] Do any two subnets on the *same* router overlap?
- [ ] Does every router have either a specific route or a default route (`0.0.0.0/0`) for anything not directly connected?
- [ ] Is the next hop of every route a real, reachable interface?
- [ ] Is the destination in this goal actually inside the range you think it is — not the network address, not the broadcast address?

### "Remember this"

> **Network address** = first address of a block. **Broadcast address** = last address of a block. Neither is ever assignable to a device.
>
> **Local delivery** (same subnet) never needs a gateway or a route — it's pure ARP + Ethernet. **Remote delivery** (different subnet) always needs a correctly configured gateway and, at the router, a correct route.
>
> `0.0.0.0/0` isn't a special case of routing — it's just the least specific possible route, which is exactly why it only ever gets used as a last resort.

---

## Resources

**Networking concepts studied for this project:**

- TCP/IP addressing and IPv4 structure
- Subnet masks and CIDR notation
- Subnetting math (host bits, block sizes, usable ranges)
- Default gateways and routing tables
- Longest-prefix / most-specific route matching and default routes (`0.0.0.0/0`)
- Routers and switches, and the Layer 2 vs Layer 3 distinction
- ARP and MAC addressing
- ICMP and `ping`
- Loopback addressing (`127.0.0.1`)
- Private vs public IP ranges, NAT/CGNAT
- The OSI model and the TCP/IP model

**Classic references:**

- RFC 791 — Internet Protocol (the original IPv4 specification)
- RFC 950 / RFC 1878 — Subnetting IP networks
- RFC 1918 — Address Allocation for Private Internets
- Cisco Networking Academy — IPv4 addressing and subnetting materials
- *TCP/IP Illustrated, Volume 1* by W. Richard Stevens

**How AI was used in this project:**

An AI assistant (Claude) was used to draft this README: organizing and expanding the networking theory I studied, generating the ASCII diagrams, formatting the reference tables, and writing the troubleshooting almanac and worked example based on the subject PDF and my own Obsidian notes. All of the underlying networking problem-solving for the 10 NetPractice levels themselves — reading each topology, computing subnets, and fixing each configuration until it passed — was done independently in the training interface; the AI was not used to solve the levels, only to help write and structure this documentation.

---

*Discover the basics of networking, one broken router at a time.*
