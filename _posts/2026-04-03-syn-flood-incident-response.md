---
layout: post
title: "My First Incident Response: Analyzing a SYN Flood DoS Attack"
date: 2026-04-03
---

This is my first hands-on incident response write-up, completed as part of the Google Cybersecurity Certificate program. The goal was to analyze a real network attack using Wireshark logs and produce a professional incident report — something I would actually do as a SOC analyst.

## The Scenario

A company's web server started throwing connection timeout errors. Employees couldn't access internal systems. Something was wrong at the network level and it was my job to figure out what.

## What I Found in the Logs

Opening the Wireshark TCP log, the first thing I noticed was normal traffic — legitimate users like `198.51.100.14` successfully completing the TCP three-way handshake and receiving `HTTP 200 OK` responses. Everything looked fine up until around timestamp 3.39.

Then IP `203.0.113.0` showed up.

Within seconds this single source was flooding the server with SYN packets — over 150 incomplete TCP handshake requests in a matter of seconds, all targeting port 443. Unlike legitimate traffic, these connections never completed. The attacker sent the initial SYN but never responded to the server's SYN/ACK, leaving hundreds of half-open connections piling up in the server's TCP backlog.

## How a SYN Flood Works

The TCP handshake has three steps:

1. **SYN** — the client sends a synchronization request to the server
2. **SYN/ACK** — the server acknowledges and reserves resources, opening a port
3. **ACK** — the client confirms, establishing the full connection

In a SYN flood attack the attacker deliberately skips step 3. The server keeps memory buffers open waiting for the final ACK that never comes. Do this at scale and the TCP backlog fills up completely — the server can no longer accept new legitimate connections.

## The Impact

By timestamp 7.33, legitimate users were receiving `HTTP 504 Gateway Timeout` errors. The server was sending `RST/ACK` packets trying to reset stalled connections but couldn't keep up. Business continuity was disrupted, putting revenue and reputation at risk.

## My Recommendations

- **Rate limiting** — limit the number of SYN packets accepted per second from a single IP
- **SYN cookies** — a technique that avoids allocating resources until the handshake is complete
- **Firewall rules** — block the offending IP `203.0.113.0` immediately and implement geo-blocking if needed
- **IDS/IPS monitoring** — set alerts for spikes in half-open connections
- **Incident response plan** — document this attack and use it to improve detection thresholds

## What I Learned

This lab made the TCP handshake click for me in a way that reading about it never did. Seeing the half-open connections pile up in the log, watching the server slow down and eventually stop responding — it made the attack tangible. It also reinforced why network monitoring and baseline behavior analysis are so critical in a SOC environment.

This is exactly the kind of work I want to be doing professionally.

---

*Files: [Incident Report](/assets/projects/cybersecurity-incident-report.pdf) | [Wireshark Log](/assets/projects/wireshark-tcp-log.pdf)*
