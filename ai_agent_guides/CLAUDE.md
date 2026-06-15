# Claude Code Project Context

Read this file first. For full details see `AGENTS.md`.

This project converts electricity distribution GIS data into lookup dictionaries and a NetworkX graph.

Module roles:

- `couplateV2.py`: heavy GIS preprocessing. Reads shapefiles, performs spatial joins, exports Excel lookup files.
- `import_dicts.py`: fast loader. Reads lookup Excel files and converts them into Python dicts/DataFrames.
- `graph.py`: graph and trace logic. Loads dictionaries, builds a NetworkX graph, traces feeder/bay/substation connections.

Do not run or refactor `couplateV2.py` unless the task explicitly involves GIS preprocessing or lookup generation.

Preserve:

- `ABB_INT_ID` identity semantics.
- coordinate tuples rounded with `PRECISION = 8`.
- bidirectional `start_end_2_edge_id` mapping.
- `NORMAL_ANA == "KAPALI"` -> `1`, otherwise `0`.
- Excel output names and sheet names unless explicitly asked.
- trace output columns: `from_ayirici`, `to_ayirici`, `length`, `enh_ids`, `anahtar_durumu`.

For graph tasks, inspect `docs/GRAPH_TRACE_ALGORITHM.md` before editing `graph.py`.
For data/schema tasks, inspect `docs/DATA_CONTRACTS.md`.
For performance/refactor tasks, inspect `docs/REFACTORING_ROADMAP.md`.

Prefer small patches. Explain changed files, preserved behavior, test steps, and risks.
