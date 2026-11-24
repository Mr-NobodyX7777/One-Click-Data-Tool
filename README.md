# OneClick Data Tool 

Native Python/PySide6 desktop tool: **Import → Understand → Clean → Visualize → Export**.
Offline-first. Dark mode is on by default and can be changed from the View menu.

## Run the project

**Windows:** Unzip, then double-click `Start OneClick Data Tool.bat`. This launcher
requires Python installed and downloads dependencies on first use. Subsequent launches
are offline. For development use:

```bash
python -m pip install -r requirements.txt
python main.py
```

**Distribute a genuine one-file Windows application:** On a Windows BUILD computer
with Python, run `build_windows.bat` and distribute **only**
`dist/OneClickDataTool.exe`. Recipient PCs do not need Python or a setup process.
PyInstaller output must be validated on Windows; this source ZIP does not contain a built EXE.

## Supported operations

- Import CSV / TSV / TXT, Excel (`.xlsx`), JSON / JSONL, Parquet, SQLite, XML (tabular
  repeating records), and DuckDB (`.duckdb`). DuckDB tables/views are opened read-only.
  For ambiguous `.db` files, choose DuckDB explicitly from the Format dropdown.
- XML auto-detects repeated sibling record elements. For unusual documents you may enter
  an XPath-compatible record path such as `.//item`. XML currently supports up to 100 MB
  of source data, stores extracted fields as text and does not permit DTD/entities.
- Audit full-data statistics, missing cells, duplicates and Data Health Score; export a
  portable **HTML report** from `File → Export HTML quality report` after an audit completes.
- Interactive cleaning with explanations, confirmation, history, undo/redo (up to eight
  retained undo versions) and **Reset to original**.
- Save and reopen entire working sessions as portable **`.ocdt`** project archives via
  `File → Save project` / `File → Open project`. Archive includes immutable original,
  every retained undo/redo table, metadata and cleaning history. Loading re-runs audits;
  plots and temporary audit caches are not serialized.
- Chart recommendation/validation and PNG export; cleaned export to CSV, Parquet,
  Excel, JSON and JSON Lines. The original source is never overwritten.

## Behaviours worth knowing (v0.3.1)

- Empty CSV cells are always treated as missing. Words such as `NA`/`null` are also treated as missing by default;
  the import tells you when that changed text values (edit the list in the import options).
- CSV export prefixes risky text cells (starting with `=`, `@`, ...) with `'` so spreadsheets never run them as formulas.
- JSON export writes NaN/Infinity as `null`. A column named `rowid` is renamed `rowid_1` on import.

## Limitations / planned work

- SQLite, Excel, XML and the first DuckDB-file connector materialize data into memory;
  for very large inputs prefer CSV/Parquet. XML is capped at 100 MB.
- Portable projects can be large: full surviving version snapshots are stored.
- This edition does **not** provide shared multi-user MySQL storage. Its DuckDB engine
  is per desktop session, not a central database. MySQL server integration is a
  separate architectural feature.
- SQL workspace, joins, advanced transforms, autosave/recovery and large-data performance
  validation remain future work.
- A Windows EXE must be built and smoke-tested **on Windows**. Packaging cannot
  be verified just by compiling Python files.

## Tests

```bash
python -m pytest tests/ -v
```

`tests/test_v02_features.py` covers the CSV fallback, undo trimming, XML, DuckDB,
project round-trip and HTML reports. Run with an offscreen Qt platform where needed.

## Design

- The normal DuckDB engine lives in a private temporary folder and cleans up on exit.
  `.ocdt` projects are an explicit portable snapshot, not a persistent engine.
- Sources are opened read-only, and exports never overwrite originals.
- Internal logging: see `app/errors.py`.

## Development

```
python -m venv .venv
.venv\Scripts\activate        # Windows  (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt pytest
python main.py                # run the app
QT_QPA_PLATFORM=offscreen python -m pytest tests -q   # run tests (set the variable with `set` on Windows)
```

Normal users just double-click `Start OneClick Data Tool.bat`; it creates `.venv` and installs the libraries on first run.
Dependencies are downloaded from PyPI on the user's machine and are not stored in this repository (see `THIRD_PARTY_NOTICES.md`).


## Development / GitHub

- Tests: `QT_QPA_PLATFORM=offscreen python -m pytest tests -q` (set the variable with `$env:QT_QPA_PLATFORM="offscreen"` in PowerShell).
- CI: `.github/workflows/tests.yml` runs the tests on Ubuntu and Windows for Python 3.11 and 3.12 on every push and pull request.
- Dependabot (`.github/dependabot.yml`) proposes monthly dependency updates.
- `.gitignore` keeps `.venv`, build output, logs and data files (`*.csv`, `*.xlsx`, ...) out of the repository.
- The repository contains no third-party libraries; they are installed from PyPI on each user's machine. See `THIRD_PARTY_NOTICES.md`.
- `LICENSE` is a proprietary "all rights reserved" notice suitable for a private repo; replace it if you choose an open-source licence.
