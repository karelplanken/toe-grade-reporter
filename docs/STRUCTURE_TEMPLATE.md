# TGR Project Structure - Ready for Tomorrow

**This file shows exactly what we'll create in Phase 1 (tomorrow)**

---

## Current State (Today)

```
toe-grade-reporter/
├── .env
├── .env.example
├── .gitignore
├── .python-version
├── .secret
├── .git/
├── .venv/
├── __pycache__/
│
├── main.py                    (empty, waiting)
├── pyproject.toml
├── README.md                  (updated)
├── SESSION_SUMMARY.md         (this session summary)
├── PROJECT_PLAN.md            (6-phase roadmap)
│
├── excel_file.py              (in root, will move)
├── benchmark.py               (in root, will move)
│
├── loc-sloc-dp.xlsx           (test data)
│
├── dump/                      (archived code from Phase 0)
│   ├── excel_file_v1.py
│   ├── benchmark.py
│   ├── performance_analysis.md
│   └── .gitkeep
│
├── original_project/          (reference, do not modify)
│   ├── PROJECT_SUMMARY.md
│   └── README.md
│
├── docs/
│   ├── PROJECT_STATE.md       (new - architecture, decisions)
│   ├── EXECUTION_PLAN.md      (new - step-by-step code)
│   ├── (more docs later)
│
└── (nothing else - waiting for Phase 1)
```

---

## After Phase 1 (Tomorrow's Goal)

```
toe-grade-reporter/
├── .env
├── .env.example
├── .gitignore
├── .python-version
├── .secret
├── .git/
├── .venv/
├── __pycache__/
├── *.pyc
│
├── main.py                    ✅ NEW - CLI entry point
├── pyproject.toml             (updated: add sqlalchemy, typer, pyyaml)
├── README.md                  ✅ UPDATED
├── alembic.ini               ✅ NEW - Alembic config
│
├── tgr/                       ✅ NEW - Main package
│   ├── __init__.py           ✅ NEW
│   ├── config.py             ✅ NEW - Config loader
│   ├── cli.py                ✅ NEW - Typer CLI
│   │
│   ├── database/             ✅ NEW
│   │   ├── __init__.py       ✅ NEW
│   │   ├── models.py         ✅ NEW - SQLAlchemy ORM
│   │   ├── connection.py     ✅ NEW - Session mgmt
│   │   └── migrations/       ✅ NEW (empty for now)
│   │       └── versions/     ✅ NEW
│   │
│   ├── learning_outcomes/    ✅ NEW
│   │   ├── __init__.py       ✅ NEW
│   │   ├── excel_file.py     ✅ MOVED from root + enhanced
│   │   ├── parser.py         ✅ NEW - Data extraction
│   │   └── importer.py       ✅ NEW - DB insertion
│   │
│   ├── canvas/               ✅ NEW (empty, for Phase 3)
│   │   ├── __init__.py       ✅ NEW
│   │   └── (Phase 3 modules)
│   │
│   ├── grading/              ✅ NEW (empty, for Phase 4)
│   │   ├── __init__.py       ✅ NEW
│   │   └── (Phase 4 modules)
│   │
│   ├── reporting/            ✅ NEW (empty, for Phase 5)
│   │   ├── __init__.py       ✅ NEW
│   │   └── (Phase 5 modules)
│   │
│   └── utils/                ✅ NEW
│       ├── __init__.py       ✅ NEW
│       └── helpers.py        ✅ NEW (optional utilities)
│
├── tgr_tests/                ✅ NEW - Test suite
│   ├── __init__.py           ✅ NEW
│   ├── conftest.py           ✅ NEW - Pytest fixtures
│   ├── test_excel_file.py    ✅ NEW - ExcelFile tests
│   ├── test_models.py        ✅ NEW - Model tests
│   ├── test_e2e_import.py    ✅ NEW - Integration test
│   └── fixtures/             ✅ NEW (optional, for test data)
│
├── config/                   ✅ NEW
│   ├── __init__.py           ✅ NEW (empty)
│   ├── config.yaml           ✅ NEW - Logging configuration
│   └── modules.yaml          ✅ NEW - Module registry
│
├── alembic/                  ✅ NEW - Database migrations
│   ├── env.py               ✅ NEW
│   ├── script.py.mako       ✅ NEW
│   └── versions/            ✅ NEW (empty for now)
│
├── logs/                      ✅ NEW (created at runtime)
│   └── .gitkeep             ✅ NEW (empty marker file)
│
├── data/                      ✅ NEW (for Excel files, cache)
│   └── .gitkeep             ✅ NEW
│
├── grade-reports/            ✅ NEW (for generated reports)
│   └── .gitkeep             ✅ NEW
│
├── templates/                ✅ NEW (for Phase 5)
│   └── grade_report.html    ✅ NEW (Phase 5)
│
├── dump/                      ✅ (continue)
│   └── (Phase 0 code)
│
├── original_project/         ✅ (continue)
│   └── (reference)
│
├── docs/                      ✅ (continue + expand)
│   ├── PROJECT_STATE.md
│   ├── EXECUTION_PLAN.md
│   ├── ARCHITECTURE.md       ✅ NEW (optional)
│   ├── DECISIONS.md          ✅ NEW (optional)
│   └── API.md                ✅ NEW (optional)
│
└── (rest same as today)
```

---

## Files by Category

### Python Modules (Code)
```
NEW: tgr/config.py                    (100 lines)
NEW: tgr/cli.py                       (80 lines)
NEW: tgr/database/models.py           (250 lines)
NEW: tgr/database/connection.py       (40 lines)
NEW: tgr/learning_outcomes/parser.py  (50 lines)
NEW: tgr/learning_outcomes/importer.py (150 lines)
MOVED: tgr/learning_outcomes/excel_file.py (450 lines + additions)
NEW: main.py                          (10 lines)
```

### Test Modules
```
NEW: tgr_tests/conftest.py            (80 lines)
NEW: tgr_tests/test_excel_file.py     (60 lines)
NEW: tgr_tests/test_models.py         (30 lines)
NEW: tgr_tests/test_e2e_import.py     (40 lines)
```

### Configuration Files
```
NEW: config/config.yaml               (YAML logging config)
NEW: config/modules.yaml              (YAML module registry)
NEW: .env.example                     (Template)
UPDATED: .env                         (Secrets - user filled)
```

### Special Files
```
NEW: alembic.ini                      (Alembic configuration)
NEW: logs/.gitkeep                    (Empty marker)
NEW: data/.gitkeep                    (Empty marker)
NEW: grade-reports/.gitkeep           (Empty marker)
UPDATED: pyproject.toml               (Add dependencies)
UPDATED: main.py                      (CLI entry point)
```

---

## Commands to Execute Tomorrow (in order)

```bash
# Part A: Create directories
mkdir -p tgr/database tgr/learning_outcomes tgr/canvas tgr/grading tgr/reporting tgr/utils
mkdir -p tgr_tests/fixtures
mkdir -p config
mkdir -p alembic/versions
mkdir -p logs data grade-reports templates

# Part B: Create __init__.py files (empty)
touch tgr/__init__.py
touch tgr/database/__init__.py
touch tgr/learning_outcomes/__init__.py
touch tgr/canvas/__init__.py
touch tgr/grading/__init__.py
touch tgr/reporting/__init__.py
touch tgr/utils/__init__.py
touch tgr_tests/__init__.py
touch config/__init__.py

# Part C: Create marker files for git
touch logs/.gitkeep
touch data/.gitkeep
touch grade-reports/.gitkeep

# Part D: Move existing file
mv excel_file.py tgr/learning_outcomes/

# Part E: Create YAML config files (content provided in EXECUTION_PLAN)
# vim config/config.yaml
# vim config/modules.yaml

# Part F: Create Python files (content provided in EXECUTION_PLAN)
# vim tgr/config.py
# ... etc
```

---

## Size Estimate

- **Total new lines of code:** ~1000 LOC
- **Total new test code:** ~200 LOC
- **Total configuration:** ~100 lines YAML
- **Estimated time:** 6-8 hours (collaborative, including testing)

---

## What NOT to Change

- ❌ Do NOT modify `original_project/`
- ❌ Do NOT modify `dump/` (archived code)
- ❌ Do NOT delete `benchmark.py` or old `excel_file.py` until moved
- ❌ Do NOT commit `.env` secrets to git
- ❌ Do NOT modify `.python-version`

---

## What to Verify at End of Phase 1

```bash
# 1. Structure exists
ls -la tgr/
ls -la tgr_tests/
ls -la config/

# 2. Tests pass
pytest tgr_tests/ -v

# 3. Type check
mypy tgr/ --strict

# 4. CLI works
python main.py --help
python main.py init

# 5. Full workflow
python main.py locs-import --course CEGC/2025-2026-01 --file loc-sloc-dp.xlsx
```

---

## Git Workflow

Suggested commits tomorrow:

```bash
git commit -m "Phase 1: Create package structure and config"
git commit -m "Phase 1: Add database models and connection"
git commit -m "Phase 1: Add Excel parser and importer"
git commit -m "Phase 1: Add Typer CLI with init and locs-import"
git commit -m "Phase 1: Add comprehensive test suite"
```

---

**Ready for tomorrow! Reference this file when creating directories.** ✅
