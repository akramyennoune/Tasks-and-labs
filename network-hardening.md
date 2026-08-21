# Security Risk Assessment — Hardening Recommendations

*Recommendations aligned with the NIST Cybersecurity Framework*

## Overview

With breaches becoming more frequent — and increasingly sophisticated thanks to the growing accessibility of AI-powered attack tools — hardening our systems and network isn't optional anymore, it's foundational. This report lays out three practical techniques we should implement to strengthen our security posture and stay aligned with frameworks like NIST.

The three techniques:

1. Multi-factor authentication (MFA)
2. Network access privilege
3. Port filtering

Below, I'll walk through why each one matters and how it fits into the bigger picture.

## Recommendations

Applying these three techniques builds a much stronger security foundation, reduces the risk of a reputation-damaging breach, and keeps us aligned with frameworks like NIST. Here's the breakdown:

### 1. Multi-Factor Authentication (MFA)

MFA is one of the most important controls we can put in place. Instead of relying on a single password, authentication is verified through a combination of factors: **something you know** (a password), **something you have** (a phone or security key), and **something you are** (biometrics like a fingerprint). Requiring more than one of these makes it dramatically harder for identity theft or unauthorized access attempts to succeed — even if one factor (like a password) gets compromised, the attacker still hits a wall.

### 2. Network Access Privilege

This is about controlling *who and what* gets to talk to our network — limiting, permitting, or outright blocking specific users, IP addresses, or MAC addresses based on whether they're authorized. Done properly, this significantly reduces our exposure to social engineering attempts and brute-force attacks, since unauthorized devices simply never get far enough to try.

### 3. Port Filtering

Port filtering controls which ports are open and available for communication, which in turn controls what traffic can even reach the network in the first place. By selectively enabling or disabling ports, we cut off a lot of the easy entry points that attackers and unknown threat vectors rely on to get in undetected.

## Beyond These Three

These three techniques are essential, but they're not the whole picture — no system or network is ever completely secure, no matter how well it's hardened. That's why these controls need to be paired with **penetration testing**: proactively searching for and exploiting vulnerabilities ourselves before an attacker gets the chance to. Hardening tells you what protections are in place; pentesting tells you whether they actually hold up.

## Bottom Line

MFA, network access privilege, and port filtering form a solid core of defense-in-depth — each one closes off a different avenue of attack (identity, network access, and traffic flow, respectively). Combined with regular pentesting to validate that these controls are actually working as intended, this gives us a security posture that's both proactive and continuously verified, rather than just "set and forget."
