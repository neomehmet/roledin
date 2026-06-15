# Gemini CLI Project Context

Use `AGENTS.md` as the main instruction source.

High-level pipeline:

```text
GIS shapefiles -> couplateV2.py -> Excel lookup files -> import_dicts.py -> dict/DataFrame -> graph.py -> NetworkX trace outputs
```

When editing:

- Use `couplateV2.py` only for GIS/spatial join/dictionary export changes.
- Use `import_dicts.py` only for Excel reading and type conversion changes.
- Use `graph.py` for graph construction and trace algorithm changes.

Important invariants:

- `ABB_INT_ID` is the primary ID.
- Coordinates are rounded tuples with precision 8.
- HAT edge IDs drive physical cable length calculation.
- Path internals should preferably be `list[int]`; string conversion should happen only at export.
- Tek bara and 2.5 bara trace paths must be handled separately.
