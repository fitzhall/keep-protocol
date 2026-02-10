# KEEP Protocol Threat Model

## What KEEP Protects Against

**Lost key holder** — Death, incapacity, or disappearance of a signer would normally orphan funds. KEEP's estate integration pillar ensures legal documents, backup signers, and recovery procedures are documented and regularly tested.

**Documentation drift** — Setup changes but documentation doesn't keep pace. THAP (Trust Hash Amendment Protocol) detects when the live custody setup no longer matches the documented configuration, alerting you to stale documentation before it causes operational failure.

**Fragmented knowledge** — Each professional (attorney, accountant, custodian) sees only fragments of the custody picture. KEEP provides role-scoped exports so professionals get exactly what they need, while you maintain the complete view.

**Single point of failure** — Over-reliance on one person, device, or location. KEEP's redundancy metrics and 3-3-3 rule (3 locations, 3 device types, 3 people) surface dangerous concentration before it becomes critical.

**Unauthorized access by professionals** — Sharing too much with service providers. Role-scoped exports redact sensitive fields (locations, device details, personal identifiers) while providing enough context for professionals to do their jobs.

**Succession failure** — Heirs can't find keys, don't know procedures, or lack legal authority. Estate pillar documents recovery procedures, legal instruments, and trains heirs through continuity drills.

## What KEEP Does NOT Protect Against

**Private key compromise** — KEEP is a documentation layer. It does NOT store private keys, seed phrases, or signing material. A compromised key is still compromised regardless of documentation quality.

**Physical theft of signing devices** — KEEP documents where devices are stored, but doesn't prevent physical theft. Use appropriate physical security (safes, secure locations, tamper-evident packaging).

**$5 wrench attacks / coercion** — Documentation won't help if someone is pointing a weapon at you. KEEP may actually increase risk if an attacker obtains your .keep file and learns your full custody structure.

**Bugs in wallet software or hardware** — KEEP documents which software/hardware you use, but can't fix vulnerabilities in those systems. Follow best practices for wallet security independently.

**Nation-state level adversaries** — If a sophisticated state actor is targeting you, KEEP's documentation layer won't materially change the threat landscape. Consider operational security far beyond documentation.

**Social engineering targeting keyholders directly** — A keyholder fooled into signing a fraudulent transaction won't be saved by better documentation. KEEP can't replace fundamental transaction verification hygiene.

## The .keep File as Attack Surface

An unencrypted `.keep` file is a **map to the treasure**. It reveals:
- Custody structure (multisig thresholds, quorum rules)
- Device types and locations (hardware wallets, geographic distribution)
- People involved (key holders, heirs, professionals)
- Recovery procedures and governance rules

This is valuable reconnaissance for a targeted attacker. An attacker with your .keep file knows exactly who to target, what devices to look for, and which locations to surveil.

**Mitigations:**
- **AES-256-GCM encryption** with PBKDF2 (100,000 iterations) for .keep files at rest
- **Local-only storage** — never sync unencrypted .keep files to cloud services
- **Role-scoped exports** — redact sensitive fields when sharing with professionals
- **Access control** — treat .keep files like hardware wallet backups (encrypted, controlled access, secure locations)

**Recommendation:** Always encrypt .keep files at rest. Store the encryption passphrase separately from the file itself. Consider splitting highly sensitive custody configurations across multiple encrypted files with different access requirements.

## Common Failure Modes

**1. Key holder death**
Estate integration pillar ensures legal documents (wills, trusts, powers of attorney) reference Bitcoin holdings, backup signers are documented, and recovery procedures are tested. Event log tracks life events that should trigger review.

**2. Device rot / hardware failure**
Continuity drills catch this before it's critical. Regular signing tests (quarterly recommended) verify devices still work and keyholders remember procedures. Device rotation tracked in event log.

**3. Divorce**
Life event triggers prompt review of custody structure. Estate pillar tracks which legal documents need updating. Key governance pillar surfaces if ex-spouse is still a signer or heir.

**4. Jurisdiction change**
Continuity configuration tracks legal jurisdiction. Moving states or countries triggers review of estate documents, tax implications, and professional network. Event log provides audit trail.

**5. Professional turnover**
Professional network section documents attorneys, accountants, and custodians. Event log tracks when professionals change. Role-scoped exports mean new professionals can be onboarded without full custody structure exposure.

**6. Stale documentation**
THAP drift detection compares documented setup hash against current reality. If setup changes but .keep file doesn't update, hash mismatch alerts you to documentation staleness.

**7. Over-reliance on single custodian**
Redundancy metrics surface dangerous concentration: all devices in one location, single device type, sole keyholder. 3-3-3 rule (3 locations, 3 device types, 3 people) provides clear sufficiency threshold.

## Responsible Disclosure

If you discover a vulnerability in the KEEP Protocol specification or the reference implementation (Keep Nexus), please email **security@selfcustodyos.com** with details. For non-sensitive issues, open a GitHub issue at the appropriate repository.

We request 90 days for coordinated disclosure to allow time for fixes and user notification.
