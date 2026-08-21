# Security Incident Report — DNS Redirect / Spoofed Website Analysis

*Packet-log analysis of a suspicious DNS resolution and traffic redirect*

## Section 1: Network Protocols Involved

The two protocols at play here are **HTTP** and **DNS**, both of which operate at the application layer (Layer 4 of the TCP/IP model — or Layer 7 if you're mapping it to OSI).

Looking at the log, the IP address `203.0.113.22` resolves to `yummyrecipesforme.com` — this domain-to-IP mapping is what shows up in the DNS records referenced in the log.

## Section 2: Documenting the Incident

Here's the sequence of events as it plays out in the traffic log:

1. **Initial connection request** — the source machine (`your.machine`) sends a connection request (`Flags [S]`) from local port `36086` to `yummyrecipesforme.com` on its HTTP port. (The `.http` suffix in the log is just shorthand for port 80, the standard HTTP port.)

2. **Handshake acknowledged** — the destination responds, acknowledging the connection request (`Flags [S.]`) — the second step of the TCP three-way handshake.

3. **Normal communication** — traffic flows normally between the source and destination for about two minutes, based on the timestamps in the log (starting around 14:18). During this window, the browser sends a `GET` request to pull data from the site — all looking like standard, expected web traffic so far.

4. **A second DNS request — where things turn** — the source machine reaches out to the DNS server again (`your.machine.52444 > dns.google.domain`), but this time the resolution returns a *different* IP address: `192.0.2.172`, tied to a different domain: `greatrecipesforme.com`.

5. **Traffic redirected to the new (spoofed) site** — the connection shifts entirely to this new destination:
   - Outgoing: `your.machine.56378 > greatrecipesforme.com.http`
   - Incoming: `greatrecipesforme.com.http > your.machine.56378`

   Notably, the source port changes again (`.56378`) with this redirect — consistent with a fresh connection being established to the new destination rather than a continuation of the original session.

**What this points to:** the pattern here — a legitimate-looking domain suddenly resolving to a different IP/domain mid-session, paired with near-identical naming (`yummyrecipesforme.com` vs. `greatrecipesforme.com`) — is a strong indicator of a **DNS spoofing or typosquatting-style redirect**, where a user is silently routed from a trusted site to a lookalike one, likely for phishing or credential harvesting purposes.

## Section 3: Remediation — Brute Force Attacks

With how accessible AI-assisted tools and precomputed password tables (rainbow tables) have become, brute-force attacks are easier to pull off than ever — and a lot of small companies still don't treat password security as a priority, which leaves them exposed to breaches and the regulatory fines that can follow.

The most effective remediation is straightforward: **enforce strong password policies.** That means requiring long, complex passwords that aren't tied to the user's name, email, or other easily-guessable personal details. On top of that, **MFA should be mandatory** — even if an attacker manages to crack or guess a password, MFA adds a second barrier that a brute-force attack alone can't get past.

## Key Takeaway

This incident is a good example of why DNS traffic deserves just as much scrutiny as the initial connection — a session can look completely legitimate at first handshake and still get silently redirected partway through. Combined with weak password hygiene, that kind of redirect becomes a much easier path to credential theft. Strong password policies plus MFA won't stop a spoofing attempt from happening, but they significantly limit what an attacker can do with credentials captured through one.
