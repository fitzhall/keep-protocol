# KEEP Protocol

KEEP (Key governance, Estate integration, Ensured continuity, Professional stewardship) is an open standard for documenting Bitcoin custody operations. The `.keep` file format provides a portable, human-readable JSON structure that captures everything needed to govern multi-generational Bitcoin holdings.

## Specification

- [Whitepaper](./KEEP-PROTOCOL.md) -- Full protocol specification
- [JSON Schema](./little-shard-v2.schema.json) -- Machine-readable schema (JSON Schema draft-07)
- [Example File](./examples/nakamoto-family.keep) -- Fictional but realistic `.keep` file

## Validate a .keep File

```bash
npx ajv validate -s little-shard-v2.schema.json -d examples/nakamoto-family.keep
```

## License

MIT
