# Key Governance Specification

**Version:** 1.0
**Status:** Draft
**Author:** Keep Protocol
**Date:** 2026-02-12
**Parent:** [KEEP Protocol Specification](./KEEP-PROTOCOL.md) -- Section 3.1

---

## Abstract

This document defines the Key Governance model for the KEEP Protocol, extending the base keyholder and wallet structures with two new concepts: **Functional Roles** (what a keyholder can do operationally) and **Wallet Tiers** (the security posture of a wallet). Together, these enable precise governance modeling for multisig custody arrangements -- from a solo holder with a single warm wallet to a corporate treasury with distributed cold vaults.

All fields introduced in this specification are **optional**. Files without functional roles or wallet tiers remain valid under the KEEP Protocol schema.

---

## 1. Functional Roles

A keyholder's existing `role` field describes their **relationship** to the custody arrangement (primary, spouse, child, attorney, custodian, friend, other). The new `functional_role` field describes their **operational capability** -- what they can do with their key in the context of signing and recovery.

### 1.1 Role Definitions

| Functional Role | Description | Capabilities |
|----------------|-------------|--------------|
| `owner` | Primary custody operator. Can initiate transactions and co-sign. | Initiate + Sign |
| `signer` | Authorized co-signer for normal operations. Cannot initiate unilaterally. | Sign only (normal ops) |
| `protector` | Emergency and recovery participant. Signs only during recovery or emergency scenarios. | Sign only (emergency/recovery) |

### 1.2 Role Semantics

**Owner.** An owner can both propose transactions and provide a signature. In a family context, the primary Bitcoin holder is typically an owner. In a corporate context, the CEO and CFO might both be owners. Owners drive day-to-day custody operations.

**Signer.** A signer provides a co-signature when asked but does not initiate transactions. A spouse who co-signs family transactions, or a CFO who countersigns treasury movements, is a signer. Signers participate in normal operations but do not drive them.

**Protector.** A protector holds a key exclusively for emergency and recovery scenarios. They do not participate in normal transaction signing. An attorney holding a key in a bank vault, or an institutional custodian holding a key in cold storage, is a protector. Protectors are the safety net -- their keys are activated only when something goes wrong.

### 1.3 Relationship to Existing `role` Field

The `functional_role` field is independent of the existing `role` field. A keyholder with `role: "attorney"` might have `functional_role: "protector"` (typical) or `functional_role: "signer"` (if the attorney actively co-signs). The two fields answer different questions:

- `role`: Who is this person in the family/organization?
- `functional_role`: What can this person do with their key?

---

## 2. Wallet Tiers

A wallet's `tier` field describes its security posture and signing model.

### 2.1 Tier Definitions

| Tier | Description | Signing Model |
|------|-------------|---------------|
| `warm` | Operational wallet. An owner can sign alone or with minimal co-signing. | Owner can meet threshold alone or with one co-signer |
| `cold` | Long-term storage vault. Distributed signing required -- no single person holds enough keys to meet the threshold. | Multiple independent signers required |

### 2.2 Tier Semantics

**Warm.** A warm wallet is designed for operational use. The signing threshold can be met by a single owner (1-of-1, 1-of-2) or by an owner plus one co-signer (2-of-3 where owner holds 1 key). Warm wallets prioritize accessibility for day-to-day transactions while maintaining basic multisig protection.

**Cold.** A cold wallet (vault) is designed for long-term storage. The signing threshold requires coordination among multiple independent parties. The cold vault hard rule (Section 3) recommends that no single person hold enough keys to meet the threshold unilaterally.

---

## 3. Threshold Rules

### 3.1 General Threshold Rule

For any wallet, the **owners + signers** participating in normal operations should be able to meet or exceed the wallet threshold. Protectors are not counted for normal operations -- they are reserved for emergency/recovery scenarios.

```
owners_count + signers_count >= wallet.threshold
```

This ensures that normal transaction signing does not require activating emergency keys.

### 3.2 Cold Vault Hard Rule

For cold-tier wallets, the recommended security posture is:

> **No single person should hold enough keys to unilaterally meet the signing threshold.**

This is a governance recommendation, not a schema enforcement. Implementations SHOULD surface a warning (not an error) when a cold vault violates this rule. The warning allows families to make informed decisions while acknowledging that some configurations (e.g., a solo holder bootstrapping into multisig) may temporarily violate this rule.

**Example violation:** A 2-of-3 cold vault where one person holds 2 keys. That person can sign alone, defeating the purpose of distributed cold storage.

**Example compliance:** A 2-of-3 cold vault where three different people each hold 1 key. No single person can sign alone.

### 3.3 Self-Custodied Split Pattern

A common pattern for solo holders transitioning to multisig: the same person holds 2 keys stored in **different physical locations** (e.g., home safe and bank vault). This provides geographic redundancy without requiring additional trusted parties.

In this pattern, the single owner technically holds enough keys to meet a 2-of-3 threshold. This is acceptable for warm-tier wallets. For cold-tier wallets, the cold vault hard rule would flag a warning, encouraging the holder to distribute at least one key to an independent party.

---

## 4. Recovery and Emergency Signing

Recovery is initiated through **governance rules** (the existing `governance_rules[]` structure in the KEEP Protocol), not through role changes. A protector's key is always a protector's key -- it is the governance rule that authorizes its use in a specific scenario.

**Example governance rule for recovery:**

| who | canDo | when | condition |
|-----|-------|------|-----------|
| Protector (Attorney) | Co-sign recovery transaction | Death or incapacity of Owner | Death certificate + 2 witnesses |

The protector does not become an owner or signer during recovery. The governance rule explicitly authorizes their participation under defined conditions. This maintains clean separation between role definitions and operational authorization.

---

## 5. Signing Scenarios

### 5.1 Two-of-Three (2-of-3) Configurations

| Scenario | Key 1 | Key 2 | Key 3 | Can Sign? |
|----------|-------|-------|-------|-----------|
| Normal transaction | Owner | Signer | -- | Yes (Owner + Signer = 2) |
| Owner + Protector emergency | Owner | -- | Protector | Yes (via governance rule) |
| Signer + Protector recovery | -- | Signer | Protector | Yes (via governance rule) |
| Owner alone | Owner | -- | -- | No (threshold = 2) |
| Protector alone | -- | -- | Protector | No (threshold = 2) |

**Typical 2-of-3 family setup:**
- Key 1: Primary holder (Owner)
- Key 2: Spouse (Signer)
- Key 3: Attorney or trusted third party (Protector)

### 5.2 Three-of-Five (3-of-5) Configurations

| Scenario | Key 1 | Key 2 | Key 3 | Key 4 | Key 5 | Can Sign? |
|----------|-------|-------|-------|-------|-------|-----------|
| Normal operations | Owner | Owner | Signer | -- | -- | Yes (2 Owners + 1 Signer = 3) |
| Partial emergency | Owner | -- | Signer | Protector | -- | Yes (via governance rule) |
| Full recovery | -- | -- | Signer | Protector | Protector | Yes (via governance rule) |
| Two owners only | Owner | Owner | -- | -- | -- | No (threshold = 3) |

**Typical 3-of-5 corporate setup:**
- Key 1: CEO (Owner)
- Key 2: CFO (Owner)
- Key 3: Board member (Signer)
- Key 4: Institutional custodian (Protector)
- Key 5: Attorney (Protector)

---

## 6. Example Configurations

### 6.1 Couple (2-of-3)

```
Wallet: Family Vault (cold, 2-of-3)
  Key 1: Partner A (Owner)     -- Home safe
  Key 2: Partner B (Signer)    -- Home safe (separate device)
  Key 3: Attorney (Protector)  -- Bank vault

Normal signing: Partner A + Partner B
Emergency: Partner A + Attorney (via governance rule)
Recovery: Partner B + Attorney (via governance rule)
```

### 6.2 Solo Holder (1-of-1 warm + 2-of-3 cold)

```
Wallet 1: Spending (warm, 1-of-1)
  Key 1: Holder (Owner) -- Mobile or hardware wallet

Wallet 2: Savings (cold, 2-of-3)
  Key 1: Holder (Owner)       -- Home safe
  Key 2: Holder (Owner)       -- Bank safe deposit box [self-custodied split]
  Key 3: Trusted friend (Protector) -- Their home safe

Normal signing (Wallet 1): Holder signs alone
Normal signing (Wallet 2): Holder uses keys from 2 locations
Recovery (Wallet 2): Holder + Friend (via governance rule)
```

### 6.3 Family with Children (3-of-5)

```
Wallet: Family Trust Vault (cold, 3-of-5)
  Key 1: Parent A (Owner)       -- Home safe
  Key 2: Parent B (Owner)       -- Office safe
  Key 3: Adult Child (Signer)   -- Their home safe
  Key 4: Attorney (Protector)   -- Law firm vault
  Key 5: Advisor (Protector)    -- Institutional custody

Normal signing: Parent A + Parent B + Adult Child
Emergency: Parent A + Adult Child + Attorney
Full recovery: Adult Child + Attorney + Advisor
```

### 6.4 Corporate Treasury (2-of-3 warm + 3-of-5 cold)

```
Wallet 1: Operating Fund (warm, 2-of-3)
  Key 1: CEO (Owner)           -- Office safe
  Key 2: CFO (Owner)           -- Office safe
  Key 3: Board member (Protector) -- Bank vault

Wallet 2: Strategic Reserve (cold, 3-of-5)
  Key 1: CEO (Owner)           -- Office safe
  Key 2: CFO (Owner)           -- Office safe
  Key 3: Board member (Signer) -- Bank vault
  Key 4: Custodian (Protector) -- Institutional vault
  Key 5: Attorney (Protector)  -- Law firm vault

Normal ops (Fund): CEO + CFO
Normal ops (Reserve): CEO + CFO + Board member
Emergency freeze: CEO or CFO (governance rule)
Recovery: Board member + Custodian + Attorney
```

### 6.5 Business Partners (2-of-3)

```
Wallet: Partnership Vault (cold, 2-of-3)
  Key 1: Partner A (Owner)      -- Their office safe
  Key 2: Partner B (Owner)      -- Their office safe
  Key 3: Attorney (Protector)   -- Law firm vault

Normal signing: Partner A + Partner B
Dispute resolution: Either partner + Attorney (via governance rule)
```

---

## 7. Schema Reference

### 7.1 FunctionalRole (enum)

```json
{
  "type": "string",
  "enum": ["owner", "signer", "protector"]
}
```

Added as an optional `functional_role` property on the `KeyHolder` definition.

### 7.2 WalletTier (enum)

```json
{
  "type": "string",
  "enum": ["warm", "cold"]
}
```

Added as an optional `tier` property on the `Wallet` definition.

### 7.3 THAP Canonical Fields

The following fields are added to the THAP canonical field set:

- `keyholders[].functional_role` -- included when present
- `wallets[].tier` -- included when present

These fields affect the THAP hash only when they are set. Files without these fields produce the same THAP hash as before, maintaining backward compatibility.

---

## 8. Backward Compatibility

Both `functional_role` and `tier` are optional fields. Existing `.keep` files without these fields remain fully valid and produce identical THAP hashes. Applications implementing this specification SHOULD:

1. Display functional role and tier when present
2. Allow users to set these fields on existing keyholders and wallets
3. Not require these fields for any validation or scoring
4. Surface cold vault hard rule warnings only when `tier: "cold"` is explicitly set

---

*This document is released under the MIT License. It is a companion specification to the [KEEP Protocol](./KEEP-PROTOCOL.md).*
