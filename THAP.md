# Trust Hash Amendment Protocol (THAP)

**Version:** 1.0
**Status:** Draft Specification
**Author:** Keep Protocol
**Date:** 2026-02-11

## Abstract

Trust Hash Amendment Protocol (THAP) is a cryptographic amendment protocol that connects a `.keep` governance file to a legal trust instrument, allowing Bitcoin custody governance to evolve without requiring formal trust amendments for every operational change. THAP establishes a cryptographic hash chain that maintains verifiable lineage from an initial trust-anchored state through all subsequent governance changes, enabling attorneys and trustees to verify custody governance integrity while reducing the cost and friction of maintaining operational alignment between legal instruments and actual custody practices.

## The Problem

Trust documents are static legal instruments. Bitcoin custody operations are necessarily dynamic. Every time a family rotates a hardware wallet seed, adds a keyholder, updates signing thresholds, or modifies governance rules, the operational reality of their custody setup diverges from what is documented in their trust agreement.

This creates an untenable dilemma:

**Option A: Formally amend the trust for every change.**
Professional trust amendments cost $500-2000+ per amendment, require attorney review and drafting, take weeks to execute, and create significant friction. This discourages families from making prudent operational changes to their custody setup. The result: stale governance, unrotated keys, outdated emergency procedures.

**Option B: Update custody operations without amending the trust.**
This leaves the trust document perpetually out of sync with reality. In a dispute or emergency, executors and courts face uncertainty about which version of the governance rules is authoritative. Did the trust intend for the custody setup documented five years ago, or the setup currently in use?

Both options are dangerous. Option A creates friction that discourages good security hygiene. Option B undermines the legal certainty that trusts are designed to provide.

**The core insight:** Most custody governance changes are *operational amendments* to an existing governance framework, not *structural changes* to the trust's intent. Adding a new hardware wallet, rotating a seed phrase, or updating drill frequency are implementations of the trust's broader custody mandate — they don't change the beneficiaries, trustees, or distribution rules. These operational changes shouldn't require formal trust amendments, but they do need a mechanism to maintain legal certainty and verifiable provenance.

THAP solves this by establishing a cryptographic chain of custody for governance changes.

## How THAP Works

### The Trust-Hash Anchor

When a trust is executed, it references the `.keep` governance file by its THAP hash. This becomes the **anchor hash** — the cryptographic fingerprint of the custody governance state at the moment the trust becomes legally binding.

Example: A family finalizes their Bitcoin Family Trust on March 15, 2026. Their estate attorney includes language that references "the KEEP governance file identified by SHA-256 hash `3a7f8e2c...`" The hash `3a7f8e2c...` is computed from their initial custody setup: 2-of-3 multisig, three hardware wallets held by Mom, Dad, and Adult Child, with specific emergency procedures and governance rules.

This anchor hash serves as the cryptographic link between the legal instrument and the operational governance file.

### Canonical Fields

THAP hashes only the **structural governance fields** that represent the trust's custody mandate. These are the fields that define *how* Bitcoin custody is organized, *who* holds keys, *what* rules govern operations, and *when* actions require review.

The canonical object includes:

**Family Context:**
- `familyName` — identifies the family unit

**Custody Structure:**
- `wallets[]` — array of wallet configurations
  - `id` — unique wallet identifier
  - `label` — human-readable wallet name
  - `threshold` — required signatures
  - `total_keys` — total keys in multisig
  - `platform` — wallet software/hardware type
  - `tier` — wallet security posture: warm or cold (optional, included when present)

**Key Holders:**
- `keyholders[]` — individuals with key custody
  - `id` — unique keyholder identifier
  - `name` — keyholder name
  - `role` — family role (parent, heir, advisor)
  - `storage_type` — hardware/paper/metal
  - `location` — custody location (not address)
  - `functional_role` — operational capability: owner, signer, or protector (optional, included when present)

**Beneficiaries:**
- `heirs[]` — inheritance distribution
  - `id` — unique heir identifier
  - `name` — heir name
  - `relationship` — family relationship
  - `allocation` — inheritance percentage
  - `isKeyHolder` — whether heir holds keys

**Governance Framework:**
- `charter` — family governance principles
  - `mission` — custody mission statement
  - `principles` — governance principles array
  - `reviewFrequency` — review cadence
- `governance_rules[]` — specific operational rules
  - `id` — unique rule identifier
  - `who` — role subject to rule
  - `canDo` — permitted action
  - `when` — timing/trigger condition
  - `condition` — additional constraints
  - `status` — active/suspended

**Legal Integration:**
- `legal` — trust and legal structure
  - `trust_name` — name of legal trust
  - `jurisdiction` — governing jurisdiction
  - `bitcoin_in_docs` — whether Bitcoin is explicitly mentioned
  - `rufadaa_filed` — digital asset access authorization status
  - `trustee_names` — array of trustee names

**Professional Network:**
- `professionals` — advisors and service providers
  - `advisor` — Bitcoin custody advisor (name, firm, email)
  - `attorney` — estate attorney (name, firm)
  - `cpa` — tax advisor (name, firm)

**Continuity Procedures:**
- `continuity` — operational continuity settings
  - `drill_frequency` — emergency drill cadence
  - `checkin_frequency` — regular review cadence

**Why these fields?**

These represent the **structural decisions** that a trust document would care about: who holds keys, how signing works, what rules govern operations, who inherits, what professionals are involved, and how continuity is maintained. They define the *governance framework*.

**What's excluded?**

- `event_log[]` — operational activity history
- `drills[]` — drill execution records
- `keep_score` — completeness metrics
- `created_at`, `updated_at` — timestamps
- `current_hash`, `history[]` — THAP metadata itself

These are excluded because they represent **operational history and metadata**, not governance structure. The event log records *what happened*; the canonical fields define *what should happen*. A trust doesn't need to be amended every time a drill is executed or a check-in is completed — those are implementations of the governance framework, not changes to it.

### Hash Computation

THAP uses a deterministic, reproducible hashing process:

1. **Construct canonical object** — Extract only the canonical fields listed above from the current `.keep` file
2. **Serialize** — `JSON.stringify(canonicalObject)` with no custom replacers or formatting
3. **Encode** — Convert JSON string to UTF-8 byte array
4. **Hash** — Compute SHA-256 using Web Crypto API (`crypto.subtle.digest('SHA-256', bytes)`)
5. **Encode output** — Convert hash buffer to lowercase hexadecimal string (64 characters)

**Implementation requirements:**
- JSON serialization must be deterministic (same object always produces same string)
- No whitespace formatting or pretty-printing
- Field order is preserved by object construction order
- SHA-256 via Web Crypto API for browser compatibility
- Output is always lowercase hex, no `0x` prefix

**Example:**
```javascript
const canonicalObject = {
  familyName: "Smith Family",
  wallets: [{ id: "w1", label: "Primary Vault", threshold: 2, total_keys: 3, platform: "Sparrow" }],
  // ... other canonical fields
};

const jsonString = JSON.stringify(canonicalObject);
const bytes = new TextEncoder().encode(jsonString);
const hashBuffer = await crypto.subtle.digest('SHA-256', bytes);
const hashArray = Array.from(new Uint8Array(hashBuffer));
const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
// Result: "3a7f8e2c1b4d5f6e8a9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a"
```

### History Chain

The `.keep` file maintains a hash chain:

```typescript
{
  current_hash: "3a7f8e2c...", // Current governance state hash
  history: [
    {
      hash: "9d2e5b8f...",     // Previous hash (trust anchor)
      timestamp: "2026-03-15T14:30:00Z",
      note: "Initial trust execution"
    },
    {
      hash: "2f1c4a7e...",     // Hash after first amendment
      timestamp: "2026-06-20T09:15:00Z",
      note: "Added hardware wallet for Adult Child, updated to 2-of-3"
    },
    {
      hash: "7b3d9e1a...",     // Hash after second amendment
      timestamp: "2026-09-10T16:45:00Z",
      note: "Rotated primary vault seed, updated keyholder locations"
    }
  ]
}
```

**Protocol:**

1. **On initial creation:** Compute hash, set as `current_hash`, initialize `history` as empty array
2. **On governance change:**
   - Append current hash to `history[]` with timestamp and descriptive note
   - Recompute hash from new canonical state
   - Update `current_hash`
3. **On file load:** Recompute hash and compare to stored `current_hash` (drift detection)

**Properties:**
- History is append-only — entries are never deleted or modified
- History is chronologically ordered
- Each entry describes *what changed* between that hash and the next
- The chain traces from anchor hash forward to current state
- Breaks in the chain indicate tampering or file corruption

## Legal Integration Pattern

### Trust Language

THAP requires specific language in the trust instrument to establish the hash anchor and authorize the amendment protocol.

**Recommended trust language:**

> **Bitcoin Custody Governance**
>
> The Grantor's Bitcoin custody operations shall be governed in accordance with the KEEP governance file (the "Governance File"), initially identified by SHA-256 hash `[ANCHOR_HASH]`, as maintained and amended per the Trust Hash Amendment Protocol (THAP) as documented at [protocol reference URL].
>
> Amendments to custody governance documented via THAP shall not require formal trust amendment, provided:
> (a) The current Governance File's hash chain maintains unbroken cryptographic lineage from the anchor hash specified above;
> (b) Amendments are limited to operational custody matters including but not limited to: key rotation, keyholder changes, wallet configuration, signing procedures, drill schedules, and governance rules implementation; and
> (c) Amendments do not modify beneficiary allocations, trustee appointments, jurisdictional provisions, or distribution timing absent formal trust amendment.
>
> The Trustee shall verify Governance File integrity annually by validating the THAP hash chain from the anchor hash to the current state. In the event of a hash chain break or unresolved discrepancy, the most recent verifiable Governance File state shall govern until the Trustee determines the correct current state.

**Key elements:**

1. **Anchor hash specification** — The trust must include the specific 64-character hash that serves as the starting point
2. **Protocol reference** — Links to the THAP specification for attorneys to verify methodology
3. **Scope limitation** — Clearly defines what can be amended via THAP vs. what requires formal trust amendment
4. **Verification requirement** — Establishes trustee duty to validate hash chain
5. **Dispute resolution** — Provides fallback procedure if hash chain is broken

### Attorney Verification Workflow

When an attorney needs to verify that a current `.keep` file is a legitimate evolution of the trust-anchored governance state:

**Step 1: Obtain current governance file**
The trustee or family provides the attorney with the current `.keep` file (typically via a role-scoped export that includes only the attorney's relevant fields).

**Step 2: Compute current hash**
Attorney uses the THAP implementation (reference implementation available at `lib/thap/hash.ts` in Keep Nexus, or any standards-compliant implementation) to compute the hash of the current file's canonical fields.

**Step 3: Verify hash integrity**
Compare the computed hash to the `current_hash` stored in the file. If they don't match, the file has been modified outside the protocol (drift detected) and requires investigation.

**Step 4: Walk the chain**
Starting from `current_hash`, walk backward through `history[]` to verify:
- Each entry has a hash, timestamp, and note
- Timestamps are chronologically ordered
- The oldest hash in the chain matches the anchor hash from the trust document

**Step 5: Review amendments**
Examine the notes in each history entry to understand what changed. Verify that changes fall within the scope of operational amendments (key rotation, keyholder updates, governance rules) and not structural changes (beneficiary changes, trustee appointments) that would require formal trust amendment.

**Step 6: Issue verification**
If the chain is unbroken and amendments are within scope, attorney can certify that the current governance state is a legitimate evolution of the trust-anchored state.

**Example verification:**

```
Trust executed 2026-03-15 with anchor hash: 9d2e5b8f...

Current .keep file provided 2026-12-01:
  current_hash: 3a7f8e2c...

  history:
    [0] 9d2e5b8f... | 2026-03-15 | "Initial trust execution"
    [1] 2f1c4a7e... | 2026-06-20 | "Added hardware wallet for Adult Child"
    [2] 7b3d9e1a... | 2026-09-10 | "Rotated primary vault seed"

Verification:
✓ Anchor hash matches trust document
✓ Hash chain unbroken (3 entries)
✓ Amendments within operational scope
✓ Computed hash matches current_hash

Result: VERIFIED — Current governance file is legitimate evolution of trust-anchored state.
```

### When You DO Need a Trust Amendment

THAP does not eliminate trust amendments entirely. It handles operational governance changes, but formal trust amendments are still required for:

**Beneficiary Changes:**
- Adding or removing heirs
- Modifying inheritance allocations
- Changing distribution conditions

**Trustee Changes:**
- Appointing new trustees
- Removing existing trustees
- Modifying trustee powers

**Structural Legal Changes:**
- Changing governing jurisdiction
- Converting trust type (revocable to irrevocable, etc.)
- Modifying trust term or termination conditions

**Material Distribution Changes:**
- Changing distribution timing (immediate vs. staggered)
- Adding or removing distribution conditions
- Modifying spend-down provisions

**Scope rule of thumb:**
If the change affects *who gets what and when*, you need a formal amendment. If the change affects *how the Bitcoin is held and accessed*, THAP handles it.

**When in doubt:**
If there's any question whether a change requires formal amendment, consult the estate attorney. THAP is designed to handle clear operational changes; edge cases should default to formal amendment for maximum legal certainty.

## Drift Detection

Beyond its primary role as a legal amendment protocol, THAP provides a secondary benefit: **drift detection**.

**The drift problem:**
`.keep` files may be edited outside the application (via text editor, manual JSON modification, or buggy import/export). These modifications bypass the THAP update process, breaking hash integrity.

**Detection mechanism:**
On every file load, recompute the canonical hash and compare to `current_hash`. If they don't match:

```
⚠️ HASH MISMATCH DETECTED

Expected (stored):  3a7f8e2c1b4d5f6e8a9c0d1e2f3a4b5c...
Computed (actual):  7f2a9d4e6b1c3f5a8e0d2b4c6e8a0c2d...

The file has been modified outside the THAP protocol.
This may indicate:
- Manual editing of the .keep file
- Import from untrusted source
- File corruption
- Software bug

Review recent changes and recompute hash if changes are intentional.
```

**Resolution:**
If drift is detected and changes are intentional, the user can acknowledge the changes and trigger a THAP update, which will:
1. Move the old (stored) hash to history with a note "Drift detected, hash recomputed after manual edit on [date]"
2. Update `current_hash` to the newly computed value
3. Resume normal THAP operation

**Security note:**
Drift detection is not a security mechanism against malicious tampering — it's a hygiene check. A sophisticated attacker who understands THAP could modify the file and recompute the hash. THAP is designed for legitimate use cases where the family and attorney need to verify governance lineage, not for adversarial scenarios.

## Implementation Notes

**Cryptographic primitives:**
- Use Web Crypto API (`crypto.subtle.digest`) for browser compatibility
- SHA-256 is the only supported hash function
- Do not use Node.js `crypto` module directly (not available in browser)

**Serialization requirements:**
- `JSON.stringify()` must be deterministic
- No custom `replacer` functions
- No spacing or formatting parameters
- Field order in canonical object construction determines serialization order

**Hash computation timing:**
- Compute on every save operation (after user confirms changes)
- Compute on every load operation (for drift detection)
- Do not compute on every render (performance impact)

**History management:**
- Append only — never delete or modify history entries
- Include meaningful notes that describe what changed (not just "updated")
- Timestamp in ISO 8601 format (UTC)
- History array can grow indefinitely (trim very old entries at user discretion)

**Reference implementation:**
A complete, standards-compliant implementation is available in the Keep Nexus application at `lib/thap/hash.ts`. This implementation includes:
- Canonical object construction
- Deterministic JSON serialization
- SHA-256 computation via Web Crypto API
- Hex encoding
- Drift detection on load
- History chain management

**Example integration:**

```typescript
import { computeTHAPHash, detectDrift } from '@/lib/thap/hash';
import type { LittleShardFile } from '@/lib/keep-core/data-model';

async function saveGovernanceFile(file: LittleShardFile) {
  const newHash = await computeTHAPHash(file);

  if (file.current_hash !== newHash) {
    // Governance changed, update THAP
    const updatedFile = {
      ...file,
      history: [
        ...file.history,
        {
          hash: file.current_hash,
          timestamp: new Date().toISOString(),
          note: "User provided description of what changed"
        }
      ],
      current_hash: newHash,
      updated_at: new Date().toISOString()
    };

    return updatedFile;
  }

  return file;
}

async function loadGovernanceFile(file: LittleShardFile) {
  const drift = await detectDrift(file);

  if (drift) {
    console.warn('THAP drift detected:', drift);
    // Prompt user to review changes
  }

  return file;
}
```

## Security Considerations

**THAP is not:**
- A signing protocol (no private keys involved)
- A multi-party verification system (single hash, not threshold signatures)
- A Byzantine fault-tolerant consensus mechanism
- Protection against malicious file modification by someone with write access

**THAP is:**
- A provenance protocol (proves lineage from anchor to current state)
- A drift detection mechanism (detects unintentional modifications)
- A legal integration bridge (connects trust documents to living operational files)
- A verification tool for attorneys and trustees (validates governance evolution)

**Threat model:**
THAP assumes the family and their attorney are honest actors who want to maintain verifiable governance records. It is not designed to prevent a sophisticated attacker with file write access from tampering with the `.keep` file.

**Recommended practices:**
- Store `.keep` files in version-controlled repositories (Git) for additional provenance
- Maintain offline backups of `.keep` files at each major governance change
- Attorney should maintain independent copy of anchor-hash `.keep` file
- Review THAP history during annual trust reviews

## Conclusion

Trust Hash Amendment Protocol provides a practical, cryptographically-verifiable bridge between static legal instruments and dynamic Bitcoin custody operations. By establishing a hash-anchored chain of governance changes, THAP enables families to maintain good custody hygiene without incurring prohibitive legal costs, while giving attorneys and trustees a mechanism to verify that current custody operations reflect the trust's intent.

THAP reduces the cost of custody governance from $500-2000 per formal trust amendment to near-zero for operational changes, while improving legal certainty through cryptographic proof of governance lineage. The protocol is simple enough for estate attorneys to verify manually, yet robust enough to support complex custody operations over decades.

For Bitcoin families navigating the intersection of multi-generational wealth preservation and adversarial security requirements, THAP offers a path forward: evolve custody operations as needed, maintain legal certainty, and prove governance integrity when it matters.

---

**References:**
- KEEP Protocol: [keepnexus.com/protocol](https://keepnexus.com/protocol)
- Reference Implementation: Keep Nexus `lib/thap/hash.ts`
- SHA-256: FIPS 180-4
- Web Crypto API: W3C Recommendation

**License:**
This specification is released under MIT License for implementation by any custody solution provider.
