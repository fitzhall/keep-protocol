# KEEP Protocol

KEEP (Key governance, Estate integration, Ensured continuity, Professional stewardship) is an open standard for documenting Bitcoin custody operations. The `.keep` file format provides a portable, human-readable JSON structure that captures everything needed to govern multi-generational Bitcoin holdings.

## Specification

- [Whitepaper](./KEEP-PROTOCOL.md) -- Full protocol specification
- [JSON Schema](./little-shard-v2.schema.json) -- Machine-readable schema (JSON Schema draft-07)
- [Threat Model](./THREAT-MODEL.md) -- What KEEP protects against and its limitations

## Examples

Three example `.keep` files demonstrate progressive adoption levels (see [MVK Tiers](./KEEP-PROTOCOL.md#e-minimum-viable-keep-mvk-tiers)):

| Example | Profile | KEEP Score | MVK Level |
|---------|---------|------------|-----------|
| [solo-holder.keep](./examples/solo-holder.keep) | Individual with single-sig Coldcard | 38 | Level 1 |
| [nakamoto-family.keep](./examples/nakamoto-family.keep) | Family with 2-of-3 / 3-of-5 multisig | 72 | Level 2 |
| [corporate-treasury.keep](./examples/corporate-treasury.keep) | Corporate treasury with institutional custody | 89 | Level 3 |

## Reference Implementation

**[Keep Nexus](https://selfcustodyos.com)** is the reference implementation of the KEEP Protocol — a browser-based editor for creating, validating, and managing `.keep` files. All operations run client-side with zero server dependency. Request access at [selfcustodyos.com](https://selfcustodyos.com).

## Validate a .keep File

```bash
npx ajv validate -s little-shard-v2.schema.json -d examples/nakamoto-family.keep
```

## License

MIT
