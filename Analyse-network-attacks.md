# Cybersecurity Incident Report — SYN Flood DDoS Attack

*Traffic-log analysis of a network interruption caused by a SYN flood*

## Section 1: Identifying the Attack Type

The server was unable to keep up with an overwhelming number of SYN requests, which flooded it and caused a cascade of failed TCP connections. Based on the pattern in the logs, this points to a **SYN flood** — a specific form of **Distributed Denial of Service (DDoS) attack.**

The logs show the source IP `203.0.113.0` sending a large volume of failed SYN requests into the network. What makes this a *distributed* denial of service rather than a plain DoS is the use of a botnet — many devices working together to overwhelm the target — as opposed to a DoS attack, which relies on a single device. That distinction matters operationally too: a DDoS is much harder to block with a simple IP ban, since the traffic isn't coming from one predictable source.

Notably, the logs show a completely normal, successful three-way handshake *before* the attack began. The network was functioning as expected right up until the flood started, at which point everything went down.

## Section 2: How the Attack Disrupted the Website

### Before the attack — a normal handshake

The three-way handshake completed successfully and looked exactly as it should:

1. **SYN** — the source IP (`198.51.100.23`) sent a TCP request to the destination (`192.0.2.1`), initiating the connection.
2. **SYN/ACK** — the destination replied, acknowledging the request and synchronizing back.
3. **ACK** — the source IP finalized the handshake with an acknowledgment.

After that, a `GET` request went through normally — this was legitimate traffic, and the connection worked exactly as intended.

### After the attack began — the flood

Shortly after, the logs show a sudden surge of SYN packets, this time originating from `203.0.113.0` — an IP with no prior legitimate presence on the system. Instead of completing handshakes like the earlier traffic, these SYN requests just kept coming, tying up server resources waiting on connections that were never meant to complete.

The effect was gradual but decisive: the server slowed under the load and eventually went down entirely, taking the organization's operations offline. Without backups or a way to quickly restore state, this kind of outage isn't just an inconvenience — it can mean real, lasting operational and financial damage while systems are rebuilt from scratch.

## Recommendations

To prevent this type of attack going forward:

- **Next-generation firewall (NGFW)** — to inspect traffic more intelligently than a traditional firewall and catch flood patterns before they overwhelm the server.
- **Port filtering** — to limit which ports are reachable and reduce the attack surface available to flood.
- **Network segmentation** — so that even if one segment is hit, critical systems elsewhere on the network stay isolated and operational.
- **IDS/IPS** — to actively detect and block anomalous traffic patterns like a SYN flood as the first line of defense, rather than finding out only after the server has already gone down.

## Key Takeaway

The giveaway here wasn't the SYN packets themselves — those are a completely normal part of every TCP handshake. It was the *volume*, paired with a source IP that had no established legitimate history on the network. That's the pattern to watch for: not just "are SYN requests happening" but "is the number of incomplete handshakes spiking from an unfamiliar source." Catching that shift early, via IDS/IPS and traffic monitoring, is what turns a potential outage into a blocked attempt.
