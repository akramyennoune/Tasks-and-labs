# Incident Report Analysis — DoS Attack via ICMP Flood

*Incident analysis structured around the NIST Cybersecurity Framework (Identify, Protect, Detect, Respond, Recover)*

## Summary

This morning the organization experienced an attack that took down the internal network for roughly two hours. The system stopped responding after being hit with a massive flood of incoming packets, and all signs point to a deliberate attempt to disrupt operations rather than a random spike in traffic.

Digging into it, we found that a malicious actor sent a flood of ICMP pings into the network through a firewall that had never been properly configured to handle rate limiting on that kind of traffic. With nothing in place to catch or throttle it, the flood was enough to overwhelm the network — a classic denial-of-service (DoS) attack.

## Identify

When the network services team audited the systems, the cause became clear pretty quickly: an incoming flood of ICMP packets, consistent with a "ping of death" style attack, had overwhelmed the server to the point where it simply stopped responding. This wasn't a subtle intrusion — it was a volume-based attack that took advantage of the fact that nothing was filtering or rate-limiting ICMP traffic at the perimeter.

## Protect

To close the gap that let this happen, the network security team rolled out several changes:

- **Firewall rate limiting** — a new rule to cap the rate of incoming ICMP packets, so a flood like this can't overwhelm the network the same way again.
- **Source IP verification** — checking incoming ICMP packets for spoofed source addresses, which also helps guard against smurf attacks and on-path (man-in-the-middle) attacks.
- **Network monitoring** — software to flag abnormal traffic patterns as they happen, rather than finding out after the fact.
- **IDS/IPS** — deployed to filter out ICMP traffic that shows suspicious characteristics before it ever reaches the network.

## Detect

Looking forward, preventing this category of incident — DoS and DDoS attacks especially — comes down to two things: proper firewall rules with port filtering, and network segmentation. Segmentation in particular limits how far an attack like this can spread even if it does get through, since it keeps critical systems isolated from the parts of the network most exposed to external traffic.

## Respond

Once the incident was identified, the response focused on containment and hardening: port filtering, multi-factor authentication (MFA), and new firewall rules specifically aimed at stopping ICMP floods, on top of the IDS/IPS systems already in place.

The bigger lesson here was a reminder that no network or system is ever fully airtight — even a small, easily-overlooked gap (like an unconfigured firewall rule) can be exploited to real effect. That's part of why regular penetration testing matters: it's a way to proactively find these weaknesses before an attacker does.

## Recover

The incident management team brought the network back online by:

1. Blocking all incoming ICMP packets at the firewall
2. Taking non-critical network services offline temporarily to reduce load and exposure
3. Restoring critical services once the threat was contained

The two-hour compromise was resolved successfully, with all critical services back up and running.

## Key Takeaway

A single unconfigured firewall setting was enough to bring down the network for two hours. The fixes here — rate limiting, IP verification, monitoring, IDS/IPS, and segmentation — all point at the same underlying principle: perimeter defenses need to be actively configured and tested, not just installed and assumed to be working. Regular pentesting and traffic monitoring are what catch these gaps before they turn into incidents.
