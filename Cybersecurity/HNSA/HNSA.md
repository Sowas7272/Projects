# Home Network Security Assessment: Port Scanning & Hardening with Nmap

## Idea

The goal of this project is to map my own home network, discover which devices
are connected, identify the open ports and running services on them, assess the
security risks these represent, and apply at least one concrete hardening
measure. All scanning is performed exclusively on my own network.

> **Note:** All IP addresses, MAC addresses, and device details in this write-up
> have been anonymized. Real values from a live network should not be published.

---

### Step 1: Get IP Address, Subnet Mask and Gateway

An **IP address** is the numeric address that uniquely identifies a device
within a network so that data can be sent to and from it.

**1a — Find your IP address and subnet mask.** Run `ip a` to inspect the network
interfaces. The relevant line looks like this (values replaced with
placeholders):

```
inet 10.0.0.42/24 brd 10.0.0.255 scope global dynamic wlan0
```

The `/xx` suffix is the **subnet mask** (in CIDR notation). It defines how many
bits of the address belong to the *network* portion; the remaining bits are
available for the *device* (host) portion.

To describe this formally, an IPv4 address consists of exactly four blocks
(octets):

$$
\text{ip} := \theta_1.\theta_2.\theta_3.\theta_4, \qquad \theta_j \in \{0, 1, \dots, 255\}
$$

The subnet mask determines where the address is split into its *network* part
and its *device* (host) part. A `/24` mask cuts the address after the third
block: the first three blocks identify the network, the last block identifies
the device.

$$
\underbrace{\theta_1.\theta_2.\theta_3}_{\text{network}}\,.\,\underbrace{\theta_4}_{\text{device}}
$$

Concretely, for `10.0.0.42/24`:

$$
\underbrace{10.0.0}_{\text{network}}\,.\,\underbrace{42}_{\text{device}}
$$

Since one free block spans $0$–$255$, a `/24` network contains $2^8 = 256$
possible addresses.

**1b — Find the default gateway (the router).** The **default gateway** is the
device (the router) through which all traffic leaving the local network is
sent, which makes it the key target of this assessment. Find it by running
`ip route` and reading the address after `default via`:

```
default via 10.0.0.1 dev wlan0 proto dhcp src 10.0.0.42 metric 600
```

Here `10.0.0.1` is the router. In most home networks the gateway is the `.1`
of the range, but `ip route` gives the reliable answer.

**Values (anonymized):**

| Item          | Value                     |
|---------------|---------------------------|
| IP address    | `10.0.0.42`               |
| Subnet mask   | `/24` (= `255.255.255.0`) |
| Network range | `10.0.0.0/24`             |
| Gateway/Router| `10.0.0.1`                |

---

### Step 2: Host Discovery

Before scanning ports, find out which devices are actually online in the
network. A host-discovery (ping) scan lists the reachable hosts in the range
derived above:

```
nmap -sn 10.0.0.0/24
```

- `-sn` tells nmap to do host discovery **only** (no port scan yet).

The key analytical step: map every discovered IP to a real device (phone,
laptop, printer, smart TV, router, IoT device). Anything that cannot be
identified is itself an interesting finding.

| IP            | Device (identified) | Notes         |
|---------------|---------------------|---------------|
| `10.0.0.1`    | Router              | gateway       |
| `10.0.0.42`   | My machine          | scanner host  |
| ...           | ...                 | ...           |

---

### Step 3: Port & Service Scan

*(to be completed)*

---

### Step 4: Risk Analysis

oppen Port → Applications in action → running Versions → Comparssin with DB for Exploits or something equivlant → riskassment 

---

### Step 5: Hardening

*(to be completed)*

---

### Conclusion & Learnings

*(to be completed)*

---

## AI Usage in This Project

In the interest of transparency, here is how AI (Claude) was used during this
project:

- **Concept explanations:** I used AI to explain networking fundamentals such as
  subnet masks, CIDR notation (`/24`), the loopback interface, and IPv4 vs. IPv6.
  I worked through these concepts myself and applied them to my own network.
- **Command guidance:** AI helped me confirm the correct tooling and package
  names for my Linux distribution and explained the purpose of individual nmap
  flags. The scans themselves were run and interpreted by me.
- **Write-up support:** AI corrected Markdown and LaTeX syntax, fixed typos, and
  helped structure this document. The findings, analysis, and hardening
  decisions are my own.
- **What I did without AI:** Running the actual scans on my network, identifying
  my own devices, assessing the real risks, and deciding on and applying the
  hardening measures.

The goal was to use AI as a learning aid and editor, not as a replacement for
doing and understanding the work.