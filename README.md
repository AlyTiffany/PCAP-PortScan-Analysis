# 🔍 PCAP Analysis: Network Port Scan Investigation

## Overview

In this lab, I analyzed a PCAP file using Wireshark to investigate network reconnaissance activity.

The objectives of the investigation were to:

- Identify the source of the scanning activity
- Determine which hosts were targeted
- Determine how many destination ports were scanned
- Analyze TCP SYN and SYN-ACK traffic
- Identify hosts with Remote Desktop Protocol (RDP) accessible

---

## 🛠️ Tools Used

- **Wireshark**
- TCP/IP traffic analysis
- Wireshark display filters

---

## Investigation

### Identifying the Scanning Host

Analysis of the packet capture identified the following host as the source of the scanning activity:

**Scanning Host:** `192.168.1.212`

The scan primarily targeted:

- `192.168.1.101`
- `192.168.1.102`
- `192.168.1.103`
- `192.168.1.104`

Traffic involving `192.168.1.1` was also observed.

The scanning host probed **20 distinct destination ports**.

---

### Identifying SYN Scan Traffic

To identify initial TCP connection attempts, I filtered for packets where the SYN flag was set and the ACK flag was not set:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 0
```

After identifying the scanning host, I further narrowed the traffic:

```wireshark
ip.src == 192.168.1.212 && tcp.flags.syn == 1 && tcp.flags.ack == 0
```

The first SYN associated with the scanning activity was:

| Field | Value |
|---|---|
| **Source** | `192.168.1.212` |
| **Destination** | `192.168.1.102` |
| **Timestamp** | `Feb 2, 2024 07:30:36.410417000 MST` |

---

### Identifying Open Ports

A TCP SYN scan can identify open ports based on the response from the target.

| Response | Interpretation |
|---|---|
| `SYN → SYN-ACK` | Port is open |
| `SYN → RST` | Port is closed |
| No response / ICMP response | Port may be filtered or unreachable |

To identify SYN-ACK responses, I used:

```wireshark
tcp.flags.syn == 1 && tcp.flags.ack == 1
```

The first SYN-ACK response observed was:

| Field | Value |
|---|---|
| **Packet** | `#26` |
| **Source** | `192.168.1.104` |
| **Timestamp** | `Feb 2, 2024 07:40:36.410737000 MST` |

The SYN-ACK response indicates that the targeted TCP port was open and accepting connections.

---

## RDP Discovery

Remote Desktop Protocol commonly operates over **TCP port 3389**.

To identify hosts responding positively to RDP connection attempts, I filtered for SYN-ACK responses originating from TCP/3389:

```wireshark
tcp.srcport == 3389 && tcp.flags.syn == 1 && tcp.flags.ack == 1
```

Two hosts responded with SYN-ACK packets from TCP/3389:

- `192.168.1.102`
- `192.168.1.104`

This indicates that **RDP was accessible on both systems at the time of the capture**.

---

## 📊 Key Findings

| Finding | Result |
|---|---|
| **Scanning Host** | `192.168.1.212` |
| **Primary Target Hosts** | 4 |
| **Distinct Destination Ports Scanned** | 20 |
| **First SYN Target** | `192.168.1.102` |
| **First SYN Timestamp** | `Feb 2, 2024 07:30:36.410417000 MST` |
| **First SYN-ACK** | Packet `#26` |
| **Hosts with RDP Accessible** | `192.168.1.102`, `192.168.1.104` |

---

## Analysis

The packet capture demonstrates network reconnaissance originating from `192.168.1.212`. The host probed multiple systems and destination ports to identify accessible services.

An important distinction during the investigation was separating **ports that were scanned** from **ports confirmed to be open**. A SYN packet sent to a destination port demonstrates that the port was probed, while a SYN-ACK response provides evidence that the destination was accepting TCP connections on that port.

The discovery of SYN-ACK responses from TCP/3389 on `192.168.1.102` and `192.168.1.104` confirmed that RDP was accessible on both hosts.
