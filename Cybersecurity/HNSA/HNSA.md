# Home Network Security Assessment: Port Scanning & Hardening with Nmap

## Idea

The goal of this project is to map my own home network, discover which devices
are connected, identify the open ports and running services on them, assess the
security risks these represent, and apply at least one concrete hardening
measure. All scanning is performed exclusively on my own network.

> **Note:** All IP addresses, MAC addresses, and device details in this write-up
> have been anonymized. Real values from a live network should not be published.

---

### Step 1: Get IP Address and Subnet Mask

First, run `ip a` to inspect the network interfaces. In the output, the relevant
line looks like this (values replaced with placeholders):

```
inet 10.0.0.42/24 brd 10.0.0.255 scope global dynamic wlan0
```

The `/xx` suffix is the **subnet mask** (in CIDR notation). It defines how many
bits of the address belong to the *network* portion; the remaining bits are
available for the *device* (host) portion.

To describe this formally, let an IPv4 address be written as four blocks. We
label the blocks belonging to the network with $\theta$ and the blocks
identifying the device with $\sigma$:

$$
\text{ip} := \underbrace{\theta_1.\theta_2.\dots.\theta_k}_{\text{network}}\,.\,\underbrace{\sigma_1.\dots.\sigma_m}_{\text{device}}
\qquad k + m = 4
$$

where each block is an integer in the range

$$
0 \leq \theta_j,\ \sigma_j \leq 255 .
$$

For a `/24` mask, the first $k = 3$ blocks are fixed as the network and the last
$m = 1$ block identifies the device:

$$
\underbrace{10.0.0}_{\theta:\ \text{network}}\,.\,\underbrace{42}_{\sigma:\ \text{device}}
$$

Since one free block spans $0$-$255$, a `/24` network contains $2^8 = 256$
possible addresses.

**Values (anonymized):**

| Item          | Value                     |
|---------------|---------------------------|
| IP address    | `10.0.0.42`               |
| Subnet mask   | `/24` (= `255.255.255.0`) |
| Network range | `10.0.0.0/24`             |
| Router        | `10.0.0.1`                |

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

*(to be completed)*

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