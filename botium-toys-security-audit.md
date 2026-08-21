# Botium Toys — Internal Security Audit

*Internal IT audit conducted using the NIST Cybersecurity Framework (CSF)*

## Background

Botium Toys is a small U.S.-based toy company that runs everything — design, sales, storefront, warehouse, and office — out of a single physical location. As their online sales have expanded to customers both in the U.S. and abroad, the IT department has been stretched to keep that growing digital footprint secure and compliant with the regulations that come with international, card-based e-commerce.

This audit was my chance to step into that gap: evaluate the current security posture, dig into risks and vulnerabilities affecting critical assets, and check compliance against the two regulatory frameworks that matter most here — PCI DSS (since they process online payments) and GDPR (since they have E.U. customers).

## Scope

I reviewed the IT department's assets and infrastructure end to end — employee devices, internet access, and internal network setup. Right away, a pattern emerged: the company has some solid physical and perimeter controls, but almost everything else — the controls that actually protect data once someone's past the front door — is missing.

Of everything assessed, only four controls were confirmed in place: **firewall, physical locks, CCTV, and antivirus software.**

## Assets Reviewed

**Hardware/Physical**
- Desktops, laptops, and smartphones
- Peripherals (mice, headsets, cables, cameras)
- Networking equipment

**Software/Data**
- Accounting systems
- Database, security, and inventory management systems

**Network**
- Internet access
- Internal network

## Findings

### Physical controls — in decent shape

| Control | Status | Notes |
|---|---|---|
| CCTV |  In place | Deters and detects unauthorized physical access across the premises |
| Locks (office/storefront/warehouse) |  In place | Well-implemented, restricts access to company assets |

Physical security here isn't the problem. The gaps are almost entirely on the technical side.

### Technical controls — this is where it gets concerning

| Control | Status | Notes |
|---|---|---|
| Firewall |  In place | First line of defense against unauthorized traffic |
| Intrusion Detection System (IDS) |  Missing | No automated way to catch abnormal or malicious traffic — a real blind spot |
| Encryption |  Missing | Customer and payment data isn't encrypted at rest or in transit |
| Backups |  Missing | No recovery path if ransomware, hardware failure, or data loss hits |
| Password management |  Missing | No formal policy — opens the door to weak, reused, or guessable passwords |

Four out of five technical controls I checked simply aren't there. Individually, each of these gaps is bad. Together, they compound — no encryption plus no IDS means an attacker could both get in *and* walk away with usable data, and nobody would necessarily notice until it's too late.

## Compliance Gaps

**PCI DSS** — Missing encryption, no IDS, no password policy, and no backup process all directly undercut the company's ability to protect cardholder data the way PCI DSS requires.

**GDPR** — No password management, no least-privilege access model, and no disaster recovery plan add up to real gaps in how E.U. customer data is safeguarded.

## Recommendations

1. **Encrypt sensitive data** — at rest and in transit, especially payment and customer info. This is foundational for PCI DSS and cuts exposure significantly.
2. **Deploy an IDS** — to actually see abnormal network activity happening in real time instead of finding out after the fact.
3. **Formalize password policies** — minimum length, complexity, rotation — backed by a real password management system.
4. **Set up automated backups** — with offsite or cloud storage, so a single incident doesn't become an existential one.
5. **Apply least-privilege access + build a disaster recovery plan** — to limit blast radius and support GDPR obligations.

## Conclusion

Botium Toys' current security posture carries a meaningful amount of risk. The gaps aren't obscure edge cases — they're foundational controls that most modern security frameworks treat as table stakes, and their absence leaves the company exposed to data breaches, reputational fallout, and regulatory penalties.

If I had to prioritize, I'd tackle it in this order:

1. **IDS** — visibility into the network first, so you know what you're dealing with.
2. **Encryption** — protect the data itself, in case something does get through.
3. **Password management** — close off the easiest attack vector (credential-based attacks).
4. Backups, least privilege, and disaster recovery follow — lower immediate risk, but necessary to reach full compliance and support the company's continued international growth.

## Framework Used

- NIST Cybersecurity Framework (CSF)
- Referenced regulations: PCI DSS, GDPR
