*This project has been created as part of the 42 curriculum by brouane.*

# NetPractice

## Description

**NetPractice** is a networking project from the 42 curriculum focused on understanding and configuring small simulated networks.

The goal is to make networks work correctly by configuring **IP addresses, subnet masks, gateways, routers, switches, and routes**.

The project contains **10 levels**, each presenting a different network configuration problem.

Through these exercises, I learned how devices communicate, how IP networks are divided into subnets, and how routers determine where packets should go.

---

## Networking Concepts

### IP Addressing

An IPv4 address contains **32 bits**, divided into four octets:

```text
192.168.1.10
```

A subnet mask determines which part represents the network and which part represents the host.

For example:

```text
192.168.1.10/24
255.255.255.0
```

### Subnetting

The prefix determines the number of network bits.

```text
/24 → 24 network bits + 8 host bits
/26 → 26 network bits + 6 host bits
/30 → 30 network bits + 2 host bits
```

The number of addresses is:

```text
2^(host bits)
```

Common masks:

| CIDR  | Mask              | Addresses |
| ----- | ----------------- | --------: |
| `/24` | `255.255.255.0`   |       256 |
| `/25` | `255.255.255.128` |       128 |
| `/26` | `255.255.255.192` |        64 |
| `/27` | `255.255.255.224` |        32 |
| `/28` | `255.255.255.240` |        16 |
| `/30` | `255.255.255.252` |         4 |

### Routing

A route tells a device where to send traffic:

```text
Destination => Next Hop
```

A **default route** is:

```text
0.0.0.0/0
```

It matches any IPv4 destination and is used when no more specific route exists.

### Gateway

A **default gateway** is normally the router interface used by a host to reach networks outside its own subnet.

### Routers and Switches

* **Switches** connect devices within a local network and primarily operate at Layer 2.
* **Routers** connect different networks and make forwarding decisions using IP addresses and routing tables.

### ARP and ICMP

**ARP** resolves an IPv4 address to a MAC address on a local network.

**ICMP** is used for network diagnostics and control messages. The `ping` command commonly uses ICMP to test connectivity.

### Loopback

The IPv4 loopback range is:

```text
127.0.0.0/8
```

The most common address is:

```text
127.0.0.1
```

also known as `localhost`.

---

## OSI Model

The OSI model divides networking into seven layers:

```text
7  Application
6  Presentation
5  Session
4  Transport
3  Network
2  Data Link
1  Physical
```

For NetPractice, the most relevant layers are:

* **Layer 3:** IP addresses, subnetting, routing, routers
* **Layer 2:** MAC addresses, Ethernet, switches
* **Layer 4:** TCP, UDP, and ports

---

## TCP/IP Model

The TCP/IP model groups networking into four main layers:

```text
Application
Transport
Internet
Link
```

Examples include:

```text
Application → HTTP, DNS, SSH
Transport   → TCP, UDP
Internet    → IP, ICMP
Link        → Ethernet, ARP
```

The OSI and TCP/IP models provide different ways of understanding the same networking concepts.

---

## How to Solve a Level

My general approach is:

```text
Read the topology
      ↓
Identify IP addresses and masks
      ↓
Calculate the networks and ranges
      ↓
Check gateways
      ↓
Check router interfaces
      ↓
Check routes and next hops
      ↓
Check connectivity
```

It is important to check that subnets do not overlap and that gateways belong to the correct networks.

---

## Instructions

Run the provided training interface with:

```bash
./run.sh
```

If necessary, it can also be started manually:

```bash
python3 -m http.server 49242
```

Then open:

```text
http://localhost:49242
```

Enter the required login, solve each level, and use **Check again** to validate the configuration.

After completing a level, use **Get my config** to export its configuration.

---

## Submission

There are **10 levels**, so the repository must contain:

```text
10 exported configuration files
```

One configuration file must be provided for each level, and all 10 files must be placed at the **root of the repository**.

---

## Resources

### Topics Studied

* IPv4 addressing
* CIDR and subnet masks
* Subnetting
* Network and broadcast addresses
* Default gateways
* Routing and routing tables
* Routers and switches
* ARP
* ICMP
* Loopback
* OSI model
* TCP/IP model

### References

* RFC 791 — Internet Protocol
* RFC 1918 — Private IPv4 Addresses
* RFC 792 — ICMP
* RFC 826 — ARP
* RFC 4632 — CIDR
* *Computer Networking: A Top-Down Approach* — Kurose & Ross

### AI Usage

AI was used as a learning and documentation assistant to help understand networking concepts such as **subnetting, routing, gateways, ARP, ICMP, OSI, and TCP/IP**, and to help organize this documentation.

The concepts and configurations were reviewed and tested through the NetPractice training interface.
