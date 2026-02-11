# KEEP Protocol Specification

**Version:** 2.0.0
**Status:** Draft
**Authors:** Bitcoin Command
**Created:** 2026-02-10
**License:** MIT

---

## Table of Contents

1. [Abstract](#1-abstract)
2. [The Problem](#2-the-problem)
3. [The KEEP Framework](#3-the-keep-framework)
4. [The .keep File Format](#4-the-keep-file-format)
5. [THAP -- Trust Hash Amendment Protocol](#5-thap----trust-hash-amendment-protocol)
6. [Integrity and Audit](#6-integrity-and-audit)
7. [Role-Scoped Views](#7-role-scoped-views)
8. [Security Model](#8-security-model)
9. [Comparison with Existing Solutions](#9-comparison-with-existing-solutions)
10. [Implementation](#10-implementation)
11. [Future Work](#11-future-work)
12. [Appendix](#12-appendix)
    - [A. JSON Schema](#a-json-schema)
    - [B. Example File](#b-example-file)
    - [C. THAP Canonical Field Reference](#c-thap-canonical-field-reference)
    - [D. Glossary](#d-glossary)
    - [E. Minimum Viable KEEP (MVK) Tiers](#e-minimum-viable-keep-mvk-tiers)

---

## 1. Abstract

KEEP (Key governance, Estate integration, Ensured continuity, Professional stewardship) is an open framework and portable file format for documenting Bitcoin custody governance. A `.keep` file is a self-contained JSON document that captures the complete operational picture of a Bitcoin custody arrangement across four pillars: who holds keys, what happens at death or incapacity, how continuity is maintained through drills and reviews, and which professionals coordinate the plan. The file is the single source of truth -- local-only, human-readable, vendor-neutral, and optionally encrypted. KEEP aims to do for custody operations what BIP-39 did for seed phrase generation: establish a common standard that any tool can implement.

---

## 2. The Problem

Bitcoin custody is a solved technical problem. Multisignature wallets, hardware signing devices, Shamir secret sharing, and output descriptors provide robust mechanisms for securing private keys. What remains unsolved is **governance** -- the human and operational layer that determines whether keys can actually be recovered and used when it matters.

### 2.1 The Governance Gap

When a Bitcoin holder dies, becomes incapacitated, or is otherwise unable to act, the technical security of their setup becomes an obstacle rather than a protection. The question is no longer "are the keys safe?" but rather:

- **Who** holds which keys, and where are they stored?
- **What** legal instruments authorize access to digital assets?
- **How** do heirs, attorneys, and fiduciaries coordinate a recovery?
- **When** was the setup last tested, and does the documentation reflect reality?

No existing standard addresses these questions. Each family, advisor, and custodian invents their own ad hoc documentation -- spreadsheets, notes in password managers, letters left in safes. This fragmentation means that when recovery is needed, critical information is scattered across multiple parties, formats, and locations.

### 2.2 The Multi-Stakeholder Coordination Problem

A typical Bitcoin estate plan involves at least four categories of participants:

- **Key holders** who control signing devices
- **Legal professionals** (attorneys, trustees) who execute estate instruments
- **Financial professionals** (CPAs, advisors) who manage tax and compliance
- **Heirs** who ultimately receive the assets

Each participant sees a fragment of the whole picture. The attorney knows the trust structure but not the key locations. The CPA knows the tax obligations but not the governance rules. The heir knows they are a beneficiary but not how to initiate a recovery. There is no shared document format that gives each party exactly the information they need -- and nothing more.

### 2.3 The Drift Problem

Even well-documented custody setups degrade over time. Keys are rotated without updating documentation. Heirs move and change contact information. Legal instruments are amended. Hardware devices are replaced. Without a mechanism for detecting when documentation has diverged from reality, families discover the gap only during an emergency -- when it is too late to fix.

---

## 3. The KEEP Framework

KEEP organizes custody governance into four pillars. A custody arrangement is considered complete only when all four pillars are addressed. The acronym reflects the structure:

```
K -- Key Governance
E -- Estate Integration
E -- Ensured Continuity
P -- Professional Stewardship
```

### 3.1 K -- Key Governance

Key Governance documents the technical custody structure and the human policies that govern it.

**Wallets.** Each wallet entry records the multisig quorum (M-of-N threshold), total key count, output descriptor (per BIP-380), hosting platform (if any), creation date, and optional label.

**Keyholders.** Each keyholder entry records the individual's name, role (primary holder, spouse, child, attorney, custodian, friend, or other), storage type (hardware wallet, paper, vault, digital, custodian, or mobile), physical location of the key, jurisdiction, age of the key in days, and whether the key is further sharded via Shamir secret sharing. If sharded, a nested shard configuration records the k-of-m threshold and the identities of shard holders.

**Governance Rules.** Rules are expressed as structured policy statements: *who* can do *what*, *when*, under *what condition*. Each rule has an explicit status (active, paused, or pending), an optional risk level, and execution tracking (last executed timestamp, execution count). Examples:

| who | canDo | when | condition |
|-----|-------|------|-----------|
| Spouse | co-sign transactions | anytime | primary holder approval |
| Attorney | initiate recovery | death or incapacity | death certificate + 2 witnesses |
| Trustee | distribute to heirs | after probate | court order or trust terms |

**Redundancy Metrics.** The 3-3-3 rule provides a minimum threshold for custody resilience: at least 3 signing devices, stored across at least 3 geographic locations, controlled by at least 3 distinct people. The redundancy object tracks device count, location count, person count, geographic distribution, and whether the 3-3-3 rule is satisfied.

### 3.2 E -- Estate Integration

Estate Integration connects the technical custody structure to the legal and succession framework.

**Heirs.** Each heir entry records name, relationship, allocation percentage, and whether the heir is also a keyholder. Contact information is optional and scoped -- it is stripped from role-scoped exports to professionals who do not need it.

**Charter.** The family charter is a governance document that records the family's mission statement regarding Bitcoin custody, guiding principles (as an ordered list), and review frequency (quarterly or annual). The charter provides the "why" behind custody decisions and anchors future policy discussions.

**Legal Documents.** The legal section tracks the existence and status of critical instruments: will, trust, and letter of instruction. Additional fields record the trust name, jurisdiction, whether Bitcoin is explicitly referenced in legal documents, whether RUFADAA (Revised Uniform Fiduciary Access to Digital Assets Act) authorization has been filed, trustee names, attorney contact information, and review dates (last and next).

### 3.3 E -- Ensured Continuity

Ensured Continuity addresses the ongoing operational health of the custody arrangement.

**Drills.** Recovery drills are practice runs of the recovery process. Each drill entry records the type (recovery, signing, verification, or education), participants, success/failure outcome, duration, notes, and any issues discovered. Drills are the primary mechanism for validating that documentation matches reality.

**Continuity Configuration.** The continuity config sets the expected cadence for check-ins and drills (monthly, quarterly, or annual), tracks the dates of the last and next scheduled events, defines life event triggers that should prompt an out-of-cycle review (e.g., marriage, divorce, new child, relocation), and optionally specifies a notification lead time in days.

**Key Rotations.** Each rotation entry records the timestamp, which keys were rotated, the reason for rotation, and the new output descriptor if applicable. The rotation log provides an audit trail of key lifecycle events.

### 3.4 P -- Professional Stewardship

Professional Stewardship documents the advisory team and the state of heir education.

**Professional Network.** Three professional contact slots are defined: advisor (Bitcoin estate planning specialist), attorney, and CPA. Each contact records name, firm, email, and phone. Not all slots need to be filled, but the KEEP Score reflects gaps.

**Education Status.** Tracks whether heirs have been trained on the recovery process, the date of last training, the next scheduled review, the location of training materials, and which specific heirs have completed training. An untrained heir is a single point of failure regardless of how well the rest of the plan is constructed.

---

## 4. The .keep File Format

### 4.1 Design Principles

The `.keep` file follows three core principles drawn from Bitcoin's own design philosophy:

**File-first.** The file IS the source of truth, not a database export or API response. Any application that reads `.keep` files is a viewer/editor for this file. The file can exist without any application.

**Local-only.** All operations -- creation, editing, validation, encryption, decryption -- happen client-side. No server ever sees plaintext file contents. This is a zero-knowledge architecture by design, not by policy.

**Human-readable.** The file is JSON. It can be opened in any text editor, diffed with standard tools, and version-controlled with Git. There is no proprietary binary format.

### 4.2 File Metadata

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | yes | Semantic version, currently `"2.0.0"` |
| `created_at` | string (ISO 8601) | yes | File creation timestamp |
| `last_modified` | string (ISO 8601) | yes | Last modification timestamp |
| `file_hash` | string | no | SHA-256 hash for file integrity |
| `family_name` | string | yes | Identifying name for the custody arrangement |

### 4.3 Top-Level Structure

The `LittleShardFile` is the canonical type name for a `.keep` document. Its top-level structure maps directly to the four KEEP pillars plus cross-cutting integrity fields:

```
LittleShardFile
|
|-- version                    "2.0.0"
|-- created_at                 ISO 8601 timestamp
|-- last_modified              ISO 8601 timestamp
|-- file_hash?                 SHA-256 integrity hash
|-- family_name                string
|
|-- [K] Key Governance
|   |-- wallets[]              Wallet configurations
|   |-- keyholders[]           Key holder records
|   |-- governance_rules[]     Policy statements
|   |-- redundancy             RedundancyMetrics object
|
|-- [E] Estate Integration
|   |-- heirs[]                Heir records
|   |-- charter                Charter object
|   |-- legal                  LegalDocuments object
|
|-- [E] Ensured Continuity
|   |-- drills[]               Drill records
|   |-- continuity             ContinuityConfig object
|   |-- rotations[]            Key rotation records
|
|-- [P] Professional Stewardship
|   |-- professionals          ProfessionalNetwork object
|   |-- education              EducationStatus object
|
|-- Integrity (cross-cutting)
|   |-- keep_score             KEEPScore object
|   |-- thap                   ThapIntegrity object
|   |-- event_log[]            EventLogEntry records
|
|-- risk_analysis?             Optional computed risk data
```

### 4.4 File Extension and MIME Type

The canonical file extension is `.keep`. Implementations SHOULD also accept `.keepnexus`, `.shard`, and `.json` on import for backward compatibility. The MIME type is `application/json`.

### 4.5 Versioning

The version field uses semantic versioning. The current version is `2.0.0`. Implementations MUST accept files where the version starts with `"1."` or `"2."`. Files with a `"1."` prefix use the legacy format (camelCase field names, different structure) and SHOULD be migrated to v2 on import.

**Version detection heuristic:** If a file contains the field `familyName` (camelCase) but not `family_name` (snake_case), it is a v1 file and requires migration.

---

## 5. THAP -- Trust Hash Amendment Protocol

THAP provides a lightweight mechanism for detecting when a custody arrangement has changed since it was last reviewed. It does not prevent changes -- it detects them.

### 5.1 Canonical Fields

The THAP hash is computed over a specific subset of fields from the `.keep` file. These fields represent the structural elements of the custody arrangement that, if changed, indicate the arrangement has been modified and should be reviewed.

The canonical object is constructed as follows:

```json
{
  "familyName": <family_name>,
  "wallets": [{ "id", "label", "threshold", "total_keys", "platform" }],
  "keyholders": [{ "id", "name", "role", "storage_type", "location" }],
  "heirs": [{ "id", "name", "relationship", "allocation", "isKeyHolder" }],
  "charter": { "mission", "principles", "reviewFrequency" },
  "legal": {
    "trust_name", "jurisdiction", "bitcoin_in_docs",
    "rufadaa_filed", "trustee_names"
  },
  "governance_rules": [{ "id", "who", "canDo", "when", "condition", "status" }],
  "professionals": {
    "advisor": <name>, "advisorFirm": <firm>, "advisorEmail": <email>,
    "attorney": <name>, "cpa": <name>
  },
  "continuity": { "drill_frequency", "checkin_frequency" }
}
```

Fields not in this list (e.g., `event_log`, `drills`, `keep_score`, `risk_analysis`, timestamps, contact details) are excluded because they represent activity data rather than structural configuration.

### 5.2 Hash Computation

1. Construct the canonical object from the current `.keep` file state.
2. Serialize to JSON using `JSON.stringify` (default serialization, no custom replacer).
3. Encode the JSON string to UTF-8 bytes.
4. Compute SHA-256 digest using the Web Crypto API: `crypto.subtle.digest('SHA-256', encoded)`.
5. Convert the resulting `ArrayBuffer` to a lowercase hexadecimal string.

The result is a 64-character hex string.

### 5.3 History Tracking

The `thap` object in the `.keep` file contains:

- `current_hash` -- the most recently computed THAP hash.
- `history` -- an append-only array of `{ hash, timestamp, note? }` entries.

Each time the THAP hash is recomputed and differs from `current_hash`, the old hash is appended to the history array with a timestamp and optional note describing the change. This provides a lightweight change detection trail without storing full diffs.

### 5.4 Drift Detection

An implementation SHOULD recompute the THAP hash on file load and compare it to `current_hash`. If they differ, the file has been modified outside the current application (or by a process that did not update the hash). The implementation SHOULD surface this as a warning to the user, not as a blocking error.

---

## 6. Integrity and Audit

### 6.1 Event Log

The event log is an append-only array of structured entries that record significant actions taken on or with the `.keep` file. Each entry contains:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | Unique identifier |
| `timestamp` | string (ISO 8601) | yes | When the event occurred |
| `event_type` | string | yes | Machine-readable event category |
| `description` | string | yes | Human-readable description |
| `actor` | string | no | Who performed the action |
| `metadata` | object | no | Arbitrary key-value data |
| `hash` | string | no | Hash of the previous entry |

Standard event types include: `file_created`, `file_migrated`, `vault_configured`, `keyholder_added`, `drill_completed`, `governance_updated`, `thap_recalculated`, `file_exported`, `scoped_share_generated`.

### 6.2 Hash Chain

Each event log entry MAY include a `hash` field containing a hash of the previous entry. This creates a tamper-evidence chain: if any entry in the log is modified or deleted, the chain breaks at that point. Implementations SHOULD compute this hash when appending entries and MAY verify the chain on file load.

The hash is computed by serializing the previous `EventLogEntry` object to JSON and applying a hash function. The reference implementation uses a fast non-cryptographic hash for performance; implementations requiring stronger guarantees MAY substitute SHA-256.

### 6.3 KEEP Score

The KEEP Score is a computed metric (0-100) that reflects the completeness of the custody arrangement across five component dimensions:

| Component | Range | What it measures |
|-----------|-------|-----------------|
| `security` | 0-100 | Key storage quality, encryption, hardware wallet usage |
| `redundancy` | 0-100 | Geographic/device/person distribution, 3-3-3 rule |
| `liveness` | 0-100 | Recent drills, active check-in schedule |
| `legal_readiness` | 0-100 | Will/trust status, RUFADAA filing, document completeness |
| `education` | 0-100 | Heir training completion, knowledge transfer |

The score object also includes a `trend` direction (`improving`, `stable`, or `declining`), a `calculated_at` timestamp, and a `recommendations` array of actionable improvement suggestions.

**Pillar mapping.** The KEEP Score components map to the four pillars:

- **K (Key Governance):** Wallets configured, keyholders assigned, governance rules set.
- **E (Estate):** Heirs designated, trust or will documented, charter mission defined.
- **E (Continuity):** Drills recorded, drill frequency set, last drill completed.
- **P (Professional):** Advisor assigned, attorney assigned, CPA assigned.

A pillar is considered "configured" when all of its checklist items are satisfied.

### 6.4 Proof of Control

A Proof of Control is a derived document that demonstrates a family's custody structure and governance posture WITHOUT exposing sensitive operational details. It is designed to be shared with third parties (e.g., auditors, insurance providers, institutional counterparties) who need to verify that a custody arrangement exists without seeing the details.

**Included in Proof of Control:**
- Wallet quorum summaries (e.g., "2-of-3 | Theya")
- Governance rule summaries (who/canDo/condition/status)
- Continuity evidence (drill count, last drill date, frequencies)
- Legal summary (has trust, has will, jurisdiction, RUFADAA status)
- Integrity seal (THAP hash, event log count, generation timestamp)

**Excluded from Proof of Control:**
- Key storage locations and types
- Wallet balances or amounts
- Attorney/professional contact details
- Personal contact information
- Full event log contents

---

## 7. Role-Scoped Views

The `.keep` file contains the complete custody record. When sharing with professional advisors, the principle of minimum necessary disclosure applies: each professional role receives only the fields required for their function.

### 7.1 Scoping Rules

All scoped exports include the following base fields: `version`, `family_name`, `created_at`, `last_modified`, `thap`, `keep_score`.

**Attorney scope:**

| Included | Excluded |
|----------|----------|
| Wallets (full) | Keyholder locations (redacted to `[redacted]`) |
| Keyholders (locations stripped) | Heir contact information |
| Legal documents (full) | Full event log (replaced with count summary) |
| Governance rules (full) | Professional network |
| Family charter (full) | Drills and continuity config |
| Heirs (name + allocation only) | Education status |
| THAP hash | Risk analysis |
| Event log summary (count only) | |

**CPA scope:**

| Included | Excluded |
|----------|----------|
| CPA professional contact | Full wallet details and descriptors |
| Full event log | Keyholders |
| Wallet summary (label + quorum) | Legal documents |
| Heir allocations (name + %) | Governance rules |
| THAP hash | Charter |
| KEEP Score | Heir contact info |
| | Drills and continuity config |
| | Education status |

**Advisor scope:**

| Included | Excluded |
|----------|----------|
| Full shard data | Keyholder locations (redacted to `[redacted]`) |
| All wallets and governance | |
| All heirs and estate info | |
| All professionals and continuity | |
| THAP hash and KEEP Score | |

### 7.2 Export Metadata

Each scoped export is stamped with a fresh `last_modified` timestamp at generation time. Implementations SHOULD also log a `scoped_share_generated` event to the source file's event log, recording the role and recipient.

---

## 8. Security Model

### 8.1 Architecture: Client-Side Only

The `.keep` file is designed for a zero-knowledge architecture. All cryptographic operations occur in the user's browser or local environment. No server component is required for any file operation. The reference implementation uses the Web Crypto API, which is available in all modern browsers and provides hardware-accelerated cryptographic primitives.

### 8.2 What the .keep File Is NOT

The `.keep` file is a **map**, not the **treasure**. It documents where keys are stored, who holds them, and how they should be used. It does NOT contain:

- Private keys or seed phrases
- Extended public keys (xpubs) or derivation paths (unless the user explicitly places them in the descriptor field)
- Wallet balances (the `balance_sats` field is optional and informational)
- Passwords or PINs for signing devices

Compromise of a `.keep` file reveals the custody structure (which devices exist, who holds them, where they are stored) but does not grant the ability to sign transactions. An attacker with a `.keep` file still needs physical access to the signing devices and knowledge of their PINs/passphrases.

### 8.3 Optional Encryption

Files MAY be encrypted at rest using AES-256-GCM with the following parameters:

| Parameter | Value |
|-----------|-------|
| Algorithm | AES-GCM |
| Key length | 256 bits |
| Key derivation | PBKDF2 |
| PBKDF2 iterations | 100,000 |
| PBKDF2 hash | SHA-256 |
| Salt | 16 random bytes (per document) |
| IV | 12 random bytes (per document) |

The encryption workflow:

1. User provides a password.
2. A random 16-byte salt is generated.
3. The password is imported as PBKDF2 key material.
4. An AES-256-GCM key is derived using PBKDF2 with 100,000 iterations.
5. A random 12-byte initialization vector (IV) is generated.
6. The JSON-serialized `.keep` file is encrypted using AES-GCM.
7. The encrypted ciphertext, salt, and IV are stored together.

Decryption reverses this process. The salt and IV are stored alongside the ciphertext; only the password is required from the user.

### 8.4 Threat Model

| Threat | Mitigation |
|--------|-----------|
| Server compromise | No server required; all operations are local |
| File theft (unencrypted) | Reveals structure but not keys; encrypt sensitive files |
| File theft (encrypted) | AES-256-GCM; 100k PBKDF2 iterations resist brute force |
| Metadata leakage | File contains no private keys, no balances by default |
| Drift / stale documentation | THAP hash detects structural changes |
| Tampered event log | Hash chain provides tamper evidence |
| Over-sharing with professionals | Role-scoped exports enforce minimum disclosure |

---

## 9. Comparison with Existing Solutions

| Feature | KEEP | Casa Covenant | Unchained Vaults | Ledger Recover |
|---------|------|--------------|-----------------|----------------|
| **Key management** | Documented (any vendor) | Proprietary multisig | Proprietary multisig | Proprietary SSS |
| **Estate planning** | Full pillar (heirs, charter, legal) | Basic inheritance | Letter of instruction | None |
| **Continuity (drills)** | Built-in drill tracking | None | None | None |
| **Professional coordination** | Role-scoped exports | None | Limited | None |
| **Vendor lock-in** | None (portable JSON) | Casa platform | Unchained platform | Ledger hardware |
| **Governance rules** | Structured who/what/when/condition | None | None | None |
| **Integrity tracking** | THAP + hash-chained event log | None | None | None |
| **Offline capable** | Fully offline | Requires server | Requires server | Requires server |
| **Open standard** | Yes (this specification) | No | No | No |

The key differentiator: existing solutions address a single pillar (Key management). KEEP addresses all four pillars (Key governance, Estate, Continuity, Professional coordination) in a portable, vendor-neutral format. A `.keep` file can document a Casa multisig setup, an Unchained vault, a Ledger-based arrangement, or any combination -- the format is agnostic to the underlying custody technology.

---

## 10. Implementation

### 10.1 Reference Implementation

The reference implementation is **[Keep Nexus](https://selfcustodyos.com)**, a Next.js 14 application written in TypeScript. It provides a browser-based editor for `.keep` files with the terminal-style aesthetic common to Bitcoin tooling. All operations run client-side with zero server dependency. Request access at [selfcustodyos.com](https://selfcustodyos.com). The source code is organized as follows:

- `lib/keep-core/data-model.ts` -- Canonical TypeScript type definitions for all interfaces
- `lib/keep-core/little-shard.ts` -- File creation, validation, import/export, encryption, migration
- `lib/thap/hash.ts` -- THAP canonical field extraction and SHA-256 computation
- `lib/keep-core/keep-score-v2.ts` -- Pillar completeness calculation
- `lib/proof-of-control/generate.ts` -- Proof of Control packet assembly
- `lib/shard/scope.ts` -- Role-scoped export generation
- `lib/context/FamilySetup.tsx` -- React Context wrapping `LittleShardFile` for UI state

### 10.2 Creating a .keep File

A minimal valid `.keep` file can be created by providing only a `family_name`. All other fields are initialized to safe defaults:

- Empty arrays for `wallets`, `keyholders`, `governance_rules`, `heirs`, `drills`, `rotations`
- Default `redundancy` with all counts at 0 and `passes_3_3_3_rule: false`
- Empty `charter` with annual review frequency
- `legal` with all booleans set to false and review dates set to creation date + 365 days
- `continuity` with quarterly check-in and drill frequencies
- Empty `professionals` object
- `education` with `heirs_trained: false` and empty `trained_heirs` array
- `keep_score` initialized to 25 (reflecting minimal configuration)
- Empty `thap` (hash computed on first save)
- Initial `event_log` entry of type `file_created`

### 10.3 Validating a .keep File

Validation produces a `ValidationResult` with three components:

- **errors** (array of strings) -- conditions that make the file invalid
- **warnings** (array of strings) -- conditions that indicate incomplete configuration
- **version_compatible** (boolean) -- whether the version is recognized

**Error conditions:**
- Missing `version` field
- Missing `family_name` field
- Missing `keep_score` object
- Missing `wallets` array
- Missing `keyholders` array
- `keep_score.value` outside 0-100 range
- Wallet `threshold` exceeds `total_keys`
- Wallet `threshold` less than 1
- Shard `threshold` exceeds shard `total`
- Duplicate keyholder IDs
- Unrecognized version prefix (not `"1."` or `"2."`)

**Warning conditions:**
- Empty wallets array
- Empty keyholders array
- `passes_3_3_3_rule` is false
- No drills recorded
- No will AND no trust on file

A file is considered valid if the errors array is empty. Warnings are informational and do not prevent loading.

### 10.4 Version Migration (v1 to v2)

Files using the v1 format are detected by the presence of `familyName` (camelCase) without `family_name` (snake_case). Migration maps v1 fields to v2 structure:

- `familyName` becomes `family_name`
- `vaults[].multisig` becomes `wallets[]`
- `multisig.keys[]` becomes `keyholders[]`
- `heirs[]` contact fields (`email`, `phone`) are nested under `contact`
- `trust` fields become `legal` fields with explicit boolean flags
- `drillHistory` becomes `drills[]`
- `auditTrail` becomes `event_log[]`
- `thapHash` / `thapHistory` become `thap.current_hash` / `thap.history`
- `captainSettings` advisor fields become `professionals.advisor`
- `taxSettings` CPA fields become `professionals.cpa`

A `file_migrated` event is appended to the event log recording the migration.

---

## 11. Future Work

### 11.1 BIP Proposal

Submit the KEEP framework and `.keep` file format as a Bitcoin Improvement Proposal (BIP) for formal standardization within the Bitcoin development community.

### 11.2 Multi-Shard Coordination

Enable multiple family members to each maintain their own `.keep` shard representing their view of the custody arrangement. A coordination protocol would detect conflicts between shards and surface discrepancies (e.g., heir A's shard lists a different allocation than the primary holder's shard).

### 11.3 Nostr Integration

Publish THAP hashes to Nostr relays as timestamped attestations. This provides a public, decentralized record that a custody arrangement existed in a particular state at a particular time, without revealing any details of the arrangement itself.

### 11.4 Hardware Wallet Descriptor Import

Import output descriptors directly from hardware wallet coordinators (Sparrow Wallet, Specter Desktop, Nunchuk) to auto-populate the `wallets` section of a `.keep` file, reducing manual data entry and transcription errors.

### 11.5 Collaborative Signing Integration

Hooks into multi-party signing coordinators (e.g., Nunchuk, Sparrow) to verify that the key holder assignments in the `.keep` file match the actual cosigner set of the described wallets.

### 11.6 Standardized Drill Protocols

Define machine-readable drill templates for common recovery scenarios (single key loss, holder death, geographic disaster, custodian failure) that can be loaded into any KEEP-compatible application and executed with guided steps.

---

## 12. Appendix

### A. JSON Schema

The formal JSON Schema (draft-07) for the `.keep` file format is published alongside this document:

- **Schema file:** `little-shard-v2.schema.json`

Validate a `.keep` file against the schema:

```bash
npx ajv validate -s little-shard-v2.schema.json -d examples/nakamoto-family.keep
```

### B. Example File

A complete example `.keep` file using fictional data is provided:

- **Example file:** `examples/nakamoto-family.keep`

### C. THAP Canonical Field Reference

The following fields are included in the THAP hash computation. All other fields in the `.keep` file are excluded.

```
familyName          <- family_name
wallets[].id
wallets[].label
wallets[].threshold
wallets[].total_keys
wallets[].platform
keyholders[].id
keyholders[].name
keyholders[].role
keyholders[].storage_type
keyholders[].location
heirs[].id
heirs[].name
heirs[].relationship
heirs[].allocation
heirs[].isKeyHolder
charter.mission
charter.principles
charter.reviewFrequency
legal.trust_name
legal.jurisdiction
legal.bitcoin_in_docs
legal.rufadaa_filed
legal.trustee_names
governance_rules[].id
governance_rules[].who
governance_rules[].canDo
governance_rules[].when
governance_rules[].condition
governance_rules[].status
professionals.advisor.name
professionals.advisor.firm
professionals.advisor.email
professionals.attorney.name
professionals.cpa.name
continuity.drill_frequency
continuity.checkin_frequency
```

### D. Glossary

| Term | Definition |
|------|-----------|
| **.keep file** | A JSON document conforming to the LittleShardFile v2.0 schema |
| **KEEP** | Key governance, Estate integration, Ensured continuity, Professional stewardship |
| **THAP** | Trust Hash Amendment Protocol -- SHA-256 based drift detection |
| **Little Shard** | The informal name for a `.keep` file |
| **KEEP Score** | A composite 0-100 metric reflecting custody arrangement completeness |
| **Proof of Control** | A derived document proving custody structure without sensitive details |
| **3-3-3 Rule** | Minimum redundancy: 3 devices, 3 locations, 3 people |
| **Role-Scoped Export** | A filtered copy of the `.keep` file appropriate for a specific professional role |
| **Pillar** | One of the four KEEP framework categories (K, E, E, P) |
| **Governance Rule** | A structured policy: who / canDo / when / condition / status |
| **RUFADAA** | Revised Uniform Fiduciary Access to Digital Assets Act |
| **Output Descriptor** | A BIP-380 string describing a wallet's script and key derivation |
| **Drift** | Divergence between documented custody arrangement and actual state |

### E. Minimum Viable KEEP (MVK) Tiers

Not everyone needs all 19 required fields filled out meaningfully on day one. MVK defines progressive adoption tiers so people can start small and grow their custody governance over time.

#### Level 1: Emergency Recovery (The Bare Minimum)

The absolute minimum to prevent total loss if the holder is hit by a bus tomorrow.

**Required fields that must have real data (not just defaults):**
- `family_name`
- `wallets` (at least 1 with `threshold`/`total_keys`)
- `keyholders` (at least 1 with `name`/`role`/`storage_type`/`location`)
- `heirs` (at least 1 with `name`/`relationship`)
- `legal.has_letter_of_instruction = true` (or `has_trust`/`has_will`)

This gets you from "my family has zero idea" to "someone can figure out what exists and where."

**Approximate KEEP Score at this level:** ~25-35

#### Level 2: Governed Custody (Operational Readiness)

The setup is documented, tested, and professionally supported.

**Everything in Level 1, plus:**
- `governance_rules` (at least 1 active rule)
- `redundancy` with `passes_3_3_3_rule` checked (even if false — you've assessed it)
- `charter` with `mission` defined
- `legal` with `jurisdiction` and `bitcoin_in_docs`
- `drills` (at least 1 completed)
- `professionals` (at least 1 contact: advisor OR attorney)
- `continuity` with frequencies set

This is where most serious holders should aim. Your setup is documented, you've tested it, and at least one professional knows it exists.

**Approximate KEEP Score at this level:** ~55-70

#### Level 3: Institutional Grade (Full KEEP Compliance)

All pillars fully configured, actively maintained, professionally coordinated.

**Everything in Level 2, plus:**
- `passes_3_3_3_rule = true`
- All 3 professional slots filled
- `education.heirs_trained = true`
- `charter` with `principles` and review completed
- `legal` with RUFADAA filed, trust established
- Multiple drill types completed
- THAP history with 2+ entries (proving ongoing review)

**Approximate KEEP Score at this level:** ~85-100

---

**Implementation Note:** Implementations MAY surface MVK level as a user-facing indicator alongside the KEEP Score. The levels are advisory, not enforced by the schema — all fields remain technically required to maintain forward compatibility.

**Example file mappings:**
- `solo-holder.keep` → Level 1 (score: 38)
- `nakamoto-family.keep` → Level 2 (score: 72)
- `corporate-treasury.keep` → Level 3 (score: 89)

---

*This document is released under the MIT License. Contributions, implementations, and feedback are welcome.*
