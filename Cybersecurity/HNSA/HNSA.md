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

Use nmap to gather information about your home network in order to determine
weaknesses.

```
sudo nmap -sV -sC -O -p- <target address (e.g. router)>
```

- `sudo` -> root access for the command
- `nmap` -> call the nmap tool
- `-sV` flag -> service/version detection: identify which service and which version runs on each open port
- `-sC` flag -> run nmap's default scripts to gather extra info (titles, certificates, headers, ...)
- `-O` flag -> OS detection: guess the target's operating system (needs root)
- `-p-` flag -> scan all 65535 ports instead of only the top 1000

> All identifying values below (public IP, IPv6 address, MAC address, internal
> device IDs) have been **redacted**. Only the analysis is real.

**Target overview:**

| Item              | Value (anonymized)                       |
|-------------------|------------------------------------------|
| Host              | `10.0.0.1` (router / gateway)            |
| Identified device | AVM FRITZ!Box (home router)              |
| MAC vendor        | AVM (`AA:BB:CC:DD:EE:FF`)                 |
| OS (detected)     | Linux kernel 4.15 – 5.19                 |
| Scan duration     | ~35 s                                    |

**Open ports and services:**

| Port      | Service        | Notes                                           |
|-----------|----------------|-------------------------------------------------|
| 53/tcp    | DNS            | Router acts as local DNS resolver               |
| 80/tcp    | HTTP           | Web admin interface (FRITZ!Box), cleartext      |
| 443/tcp   | HTTPS (TLS)    | Web admin interface over TLS; self-signed cert  |
| 5060/tcp  | SIP            | VoIP / telephony (FRITZ!OS)                      |
| 8089/tcp  | tcpwrapped     | AVM internal service                            |
| 8181–8189 | HTTP / various | AVM internal services (management/UPnP-like)    |
| 49000/tcp | (AVM)          | TR-064 style management interface               |
| 49443/tcp | HTTPS (TLS)    | AVM management over TLS                          |
| others    | unknown        | Additional AVM-specific services                |

**Observations:**

- **Only expected, vendor-legitimate services are present.** Every open port
  maps to a known FRITZ!OS function (DNS, web UI, VoIP, AVM management). Nothing
  foreign or unexpected is exposed — a good baseline result.
- **The web interface ships with strong security headers.** The HTTP response
  includes `X-Frame-Options: SAMEORIGIN` (clickjacking protection),
  `X-Content-Type-Options: nosniff` (anti MIME-sniffing), a restrictive
  `Content-Security-Policy` (XSS mitigation), and a `Referrer-Policy`. This
  indicates a security-conscious default configuration.
- **HTTP (port 80) is open in cleartext.** The admin interface is reachable over
  unencrypted HTTP as well as HTTPS. Best practice is to serve the admin UI over
  HTTPS only (or strictly redirect HTTP → HTTPS). → candidate for Step 4.
- **The TLS certificate is self-signed / device-specific.** Normal for consumer
  routers, but it explains the browser certificate warning when opening the web
  UI. Low risk on the LAN.

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