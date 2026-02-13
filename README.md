# KEEP Protocol

KEEP (Key governance, Estate integration, Ensured continuity, Professional stewardship) is an open standard for documenting Bitcoin custody operations.

## The Little Shard

A **Little Shard** is a `.keep` file -- the portable, human-readable single source of truth for your Bitcoin custody governance. It's a JSON document that captures everything needed to govern multi-generational Bitcoin holdings: who holds keys, what happens at death or incapacity, how continuity is maintained, and which professionals coordinate the plan.

Your Little Shard is yours. It lives on your device, not on a server. It works with any wallet, any platform, any custody setup. It's the map -- not the treasure.

## Build Your Own Little Shard

The KEEP Protocol is an open standard. Anyone can build tools that create, read, validate, and manage `.keep` files.

**Start now:** Request access to the web portal at **[selfcustodyos.com](https://selfcustodyos.com)** to build your own Little Shard. The portal runs entirely client-side -- your data never leaves your browser.

**Build your own tools:** The [JSON Schema](./little-shard-v2.schema.json) and [whitepaper](./KEEP-PROTOCOL.md) define everything you need to implement KEEP in any language, on any platform. Custody platforms, estate planning software, hardware wallet coordinators, and advisory tools can all speak `.keep`.

## Specification

- [Whitepaper](./KEEP-PROTOCOL.md) -- Full protocol specification (v2.1)
- [JSON Schema](./little-shard-v2.schema.json) -- Machine-readable schema (JSON Schema draft-07)
- [Key Governance](./KEY-GOVERNANCE.md) -- Functional roles and tiered wallet model
- [THAP](./THAP.md) -- Trust Hash Amendment Protocol (connecting .keep files to legal trusts)
- [Threat Model](./THREAT-MODEL.md) -- What KEEP protects against and its limitations

## Examples

Three example Little Shard files demonstrate progressive adoption levels (see [MVK Tiers](./KEEP-PROTOCOL.md#e-minimum-viable-keep-mvk-tiers)):

| Example | Profile | KEEP Score | MVK Level |
|---------|---------|------------|-----------|
| [solo-holder.keep](./examples/solo-holder.keep) | Individual with single-sig Coldcard | 38 | Level 1 |
| [nakamoto-family.keep](./examples/nakamoto-family.keep) | Family with 2-of-3 / 3-of-5 multisig | 72 | Level 2 |
| [corporate-treasury.keep](./examples/corporate-treasury.keep) | Corporate treasury with institutional custody | 89 | Level 3 |

## Validate a .keep File

```bash
npx ajv validate -s little-shard-v2.schema.json -d examples/nakamoto-family.keep
```

## License

MIT
