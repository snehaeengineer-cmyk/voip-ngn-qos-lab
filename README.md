# VoIP & NGN Network Design with DiffServ QoS

Two Cisco Packet Tracer / Asterisk lab projects from my M.Sc. Communication Systems & Networks coursework at TH Köln, demonstrating SIP/RTP signalling analysis, VoIP service deployment, and DiffServ-based QoS for latency-sensitive traffic.

**Stack:** Cisco IOS (routers/switches), Asterisk, Wireshark, iPerf, SIP/RTP

---

## Results at a glance

QoS makes or breaks VoIP call quality once a link gets congested. These labs measured that effect directly rather than just asserting it.

### 1. DiffServ QoS impact on a congested 128 kbps link

| Condition | VoIP (PCMA) throughput | Competing (iPerf) traffic | Outcome |
|---|---|---|---|
| **Best Effort (no QoS)** | 10.11 kbps (7.9% of link) | 117.89 kbps (92.1% of link) | VoIP starved — degraded audio |
| **With DiffServ (EF/AF12 marking)** | 85.6 kbps (full allocation, guaranteed) | Capped at 41.72 kbps | Excellent QoE — no echo, minimal delay |

DiffServ reserved **94.16 kbps** total for VoIP (85.6 kbps RTP media + 8.56 kbps SIP signalling, 10% overhead), leaving 33.84 kbps for best-effort traffic on the 128 kbps link.

### 2. Codec throughput comparison (measured via Wireshark I/O graphs)

| Codec | Sampling rate | Application throughput | Data-link throughput |
|---|---|---|---|
| PCMA (G.711) | 8000 Hz | 64 kbps | 85.6 kbps |
| G.722 | 16000 Hz | 64 kbps | 85.6 kbps |
| iLBC | 8000 Hz | 15.2 kbps | 27.4 kbps |

iLBC trades audio quality for bandwidth resilience — it degrades less under congestion but sounds worse even when the link is clear.

### 3. Quality of Experience vs. bandwidth (measured delay, perceptual rating)

| Codec | Link bandwidth | Delay | QoE rating |
|---|---|---|---|
| PCMA | 128 kbps | 0.75–0.95 s | 10/10 — clear, minimal echo |
| PCMA | 64 kbps | 4.19–5.92 s | 5/10 — noticeable echo |
| PCMA | 32 kbps | 4.41–9.7 s | Poor — unusable for real conversation |
| iLBC | 128 kbps | 0.71–0.90 s | 8/10 |
| iLBC | 32 kbps | 0.45–1.10 s | 9/10 — best low-bandwidth option |

### 4. SIP call setup latency

Measured directly from Wireshark capture (time between first `INVITE` and `180 Ringing`):

```
Call setup latency = 23.0500 − 11.35565 = 11.694 ms
```

---

## What was built

### Lab A — VoIP-enabled enterprise LAN with DiffServ QoS
- 3-LAN topology (Asterisk server, VoIP hardphones/softphones, routers, switches) with NAT and static routing
- SIP trunking for cross-network call routing
- DiffServ domain: VoIP media classified as **Expedited Forwarding (DSCP 46)**, SIP signalling as **Assured Forwarding AF12 (DSCP 12)**, everything else **Best Effort (DSCP 0)**
- ACL-based traffic classification + policy maps for ingress DSCP marking and egress bandwidth allocation
- Validated under a constrained 128 kbps serial link with parallel iPerf-generated congestion

📄 [Full report with topology diagrams & Cisco config](./diffserv-qos-report.pdf)

### Lab B — Next-Generation Network: SIP proxy/peer-to-peer, NAT traversal, multi-domain interconnect
- VoIP service provider network with Asterisk, hard/softphones, NAT, static routing
- SIP registration and authentication (challenge-response) analysis via Wireshark
- Peer-to-peer RTP vs. proxy-mode RTP comparison, including NAT traversal behavior (RFC 3581 symmetric response routing)
- Demonstrated NAT mapping dependency: calls **failed** when initiated from outside a NAT'd network (empty translation table) and **succeeded** when initiated from inside it
- Cross-team SIP trunk interconnection with live transcoding between PCMA and iLBC codecs

📄 [Full report with call-flow diagrams & Cisco config](./ngn-team1-report.pdf)

---

## Key takeaways

- DSCP-based traffic marking and policy-map bandwidth reservation reliably protect VoIP quality even when a link is at 92%+ congestion from competing traffic.
- Codec choice is a real engineering tradeoff: PCMA/G.722 sound best but need ~3x the bandwidth of iLBC and degrade hard under congestion; iLBC is the right call for constrained links.
- NAT silently breaks inbound VoIP unless a translation entry already exists — a call must originate from behind the NAT, or you need explicit port forwarding / SBC-style handling.
- SIP signalling (via `rport`/RFC 3581) can survive NAT independently of RTP media — a common real-world cause of "call connects but no audio."

---

## Files

- `diffserv-qos-report.pdf` — full conference-style writeup of Lab A (topology, DSCP config, throughput tables, Wireshark I/O graphs)
- `ngn-team1-report.pdf` — full writeup of Lab B (NAT, peer-to-peer SIP, transcoding, cross-team interconnect)

*Coursework completed as part of M.Sc. Communication Systems & Networks, TH Köln, under Prof. Dr. A. Grebe.*
