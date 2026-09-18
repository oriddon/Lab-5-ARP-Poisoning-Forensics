# SBT-DF203 Lab 5: ARP Poisoning Forensics

## Overview

This repository contains my work for **SBT-DF203: Basic Networking Skills for Digital Forensics – Lab 5: ARP Poisoning Forensics**.

The practical focused on understanding normal ARP behaviour, examining ARP request and reply traffic, checking IP-to-MAC address relationships, preserving packet-capture evidence, and reviewing a supplied ARP capture for possible poisoning indicators.

All work was carried out in an authorized and isolated Kali Linux virtual environment.

## Objectives

The lab covered the following tasks:

- Observe normal ARP request and reply behaviour.
- Capture and inspect ARP traffic.
- Identify ARP opcodes, sender IP addresses, sender MAC addresses, and target addresses.
- Preserve original PCAP evidence and work from a separate analysis copy.
- Verify evidence integrity using SHA-256 hashing.
- Build an ARP timeline from the supplied capture.
- Check for conflicting IP-to-MAC mappings.
- Identify whether ARP replies were solicited or unsolicited.
- Restore the lab environment after analysis.

## Lab Environment

- **Operating System:** Kali Linux
- **Virtualization:** VMware Workstation
- **Primary Interface:** `eth0`
- **Tools:** Wireshark, TShark, Scapy, net-tools, standard Linux networking utilities
- **Evidence Files:** `arp.pcap`, `normal_arp.pcapng`, `arp_working.pcap`

During the final restoration stage, the Kali VM was connected to the isolated `192.168.40.0/24` network using `192.168.40.128/24`.

## Repository Structure

```text
SBT-DF203-Lab5/
├── evidence/
│   ├── arp.pcap
│   └── normal_arp.pcapng
├── working/
│   └── arp_working.pcap
├── reports/
│   ├── arp_capture_hashes.txt
│   ├── normal_arp_fields.tsv
│   ├── arp_replies.tsv
│   ├── ip_mac_claims.txt
│   ├── partE_all_arp.tsv
│   ├── partE_arp_timeline.tsv
│   ├── partE_ip_mac_summary.txt
│   ├── arp_table_initial.txt
│   ├── arp_table_after_ping.txt
│   ├── arp_table_restored.txt
│   ├── process_check.txt
│   ├── interfaces.txt
│   ├── routes.txt
│   └── routes_final.txt
├── screenshots/
├── scripts/
└── README.md
```

## Evidence Integrity

The original ARP capture was preserved in the `evidence` directory, while a separate working copy was used for analysis.

The SHA-256 hash recorded for both the original and working copy was:

```text
342a75dc002d090cc7fd108994b6c0c9c8eaa3962cf642159b4507d5615adc3e
```

The matching hashes confirmed that the working copy remained identical to the preserved original evidence.

## Normal ARP Behaviour

A normal ARP capture was generated and saved as:

```text
evidence/normal_arp.pcapng
```

The capture showed the expected request-and-reply pattern.

- ARP request: opcode `1`
- ARP reply: opcode `2`
- Typical ARP request destination: `ff:ff:ff:ff:ff:ff`

In the normal capture, the gateway and Kali exchanged ARP traffic so that their IP and MAC address information could be learned.

## Supplied ARP Capture Analysis

The supplied evidence was analyzed from:

```text
working/arp_working.pcap
```

The capture contained eight ARP frames.

The main request-and-reply sequence was:

| Frame | Opcode | Sender IP | Sender MAC | Target IP | Interpretation |
|---|---:|---|---|---|---|
| 3 | 1 | `136.160.215.15` | `00:50:56:86:cb:fc` | `136.160.215.194` | ARP request |
| 4 | 2 | `136.160.215.194` | `00:50:56:86:02:65` | `136.160.215.15` | Solicited ARP reply |
| 5 | 1 | `136.160.215.194` | `00:50:56:86:02:65` | `136.160.215.15` | ARP request |
| 6 | 2 | `136.160.215.15` | `00:50:56:86:cb:fc` | `136.160.215.194` | Solicited ARP reply |

Frame 4 followed the request in Frame 3, while Frame 6 followed the request in Frame 5. Both replies were therefore treated as solicited replies.

## IP-to-MAC Findings

The ARP claim summary showed the following mappings:

```text
136.160.215.1    -> 00:1b:17:00:0a:30
136.160.215.194  -> 00:50:56:86:02:65
136.160.215.15   -> 00:50:56:86:cb:fc
```

The observed mappings remained consistent throughout the analyzed ARP frames.

No single IP address was observed using more than one MAC address, and no single MAC address was observed claiming both `136.160.215.15` and `136.160.215.194`.

Based on the ARP frames examined, the supplied capture did not provide conclusive evidence of an ARP-poisoning event.

## Restoration

After completing the analysis, the lab environment was checked and restored.

A process check was performed using:

```bash
ps aux | grep -E '[a]rp.py|[s]capy'
```

No matching ARP-poisoning or Scapy process was found.

The learned neighbour entries were then cleared with:

```bash
sudo ip neigh flush all
```

The ARP/neighbor table and final routing state were checked afterward to confirm that the environment had returned to a normal isolated state.

## Key Takeaways

This lab helped demonstrate that ARP analysis should be based on the actual packet sequence rather than assumptions.

A reply by itself is not enough to prove poisoning. The analyst should check:

- whether a matching request appeared first;
- whether an IP address changes between different MAC addresses;
- whether one MAC claims multiple important IP addresses;
- whether repeated or unexpected ARP replies appear;
- whether endpoint ARP tables support the packet-level findings.

## Limitation

A final `controlled_arp_poison.pcapng` file was not available for the final analysis. The main forensic findings were therefore based on the supplied `arp.pcap`, the normal ARP capture, and the exported analysis results.

## Ethical and Authorization Notice

This work was completed only in an authorized training environment.

ARP poisoning, traffic interception, and similar techniques should never be performed on third-party, public, school, office, hotel, home, or production networks without explicit permission.

## Course Information

- **Course:** SBT-DF203 – Basic Networking Skills for Digital Forensics
- **Lab:** Lab 5 – ARP Poisoning Forensics
- **Student:** Athanasius Alekwe
- **Instructor:** Aminu Idris
