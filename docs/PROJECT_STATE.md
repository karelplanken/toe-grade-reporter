# TGR (Talent-Oriented Grade Reporter) - Project State & Architecture

**Last Updated:** January 17, 2026  
**Status:** Phase 1 Planning & Directory Setup

---

## Project Identity

- **Full Name:** Talent-Oriented Grade Reporter (`toe-grade-reporter`)
- **CLI Command:** `tgr`
- **Package Name:** `tgr` (Python, snake_case)
- **Directory Convention:** Hyphens for parent (`toe-grade-reporter/`), snake_case inside for Python (`tgr/`, `tgr_tests/`)
- **Purpose:** Multi-module Canvas LMS grade reporting system with learning outcome mapping

---

## What's Completed (Phase 0)

✅ **Excel Validation Optimization**
- `ExcelFile` class with single-pass reading + caching
- Vectorized pandas validation (55× faster: ~59ms vs ~3200ms)
- Float validation for datapoint weights
- Benchmark harness in `benchmark.py`

✅ **Architecture Decisions**
- SQLite database (local, easy to deploy)
- Sync API initially, async refactor in Phase 3 (isolated to client layer)
- Single-module CLI focus, "sync all" feature designed in but deferred
- Python 3.10+ with mypy --strict type checking
- pytest for testing from day 1

---

## Core Architecture

### Data Model: Organizational Hierarchy

```
Module (e.g., "CEGC")
  ├─ name, abbreviation, description
  └─ 1:N Terms

Term (e.g., "2025-2026-01")  [Academic year - semester format]
  ├─ academic_year, semester, name (human-readable)
  └─ 1:N Courses

Course (e.g., CEGC in 2025-2026-01)  [Module-Term pair on Canvas]
  ├─ module_id (FK)
  ├─ term_id (FK)
  ├─ canvas_course_id (e.g., 123456)
  └─ Scopes all learning outcomes, students, assignments, submissions
```

### Learning Outcomes (Course-Scoped)

```
LearningOutcome (LOC)
  ├─ course_id (FK) → ensures multi-module isolation
  ├─ identifier (e.g., "LOC-1")
  └─ description

SubLearningOutcome (SLOC)
  ├─ course_id (FK)
  ├─ learning_outcome_id (FK)
  ├─ identifier (e.g., "SLOC-1.1")
  └─ description

Datapoint (DP)
  ├─ course_id (FK)
  ├─ name, weight (Float)
  └─ (1:M mapped to assignments via Assignment.datapoint_id)

sloc_datapoint (association table)
  ├─ sub_learning_outcome_id (FK)
  ├─ datapoint_id (FK)
  └─ (many-to-many relationship)
```

### Canvas Data (Course-Scoped)

```
Student
  ├─ course_id (FK) → scoped to course
  ├─ canvas_user_id, name, email, sis_user_id, login_id
  └─ Submission (1:M)

Assignment
  ├─ course_id (FK)
  ├─ canvas_assignment_id
  ├─ datapoint_id (FK, nullable) → maps to learning outcome DP
  ├─ name, due_at, points_possible
  └─ Submission (1:M)

Submission
  ├─ student_id (FK)
  ├─ assignment_id (FK)
  ├─ canvas_submission_id
  ├─ score, submitted_at, workflow_state
  └─ (composed foreign keys imply course scoping)
```

---

## Directory Structure

```
toe-grade-reporter/
├── tgr/                           # Main Python package
│   ├── __init__.py
│   ├── config.py                  # Config loading (YAML + .env)
│   │
│   ├── database/
│   │   ├── __init__.py
│   │   ├── models.py              # SQLAlchemy ORM (Module, Term, Course, LOC, SLOC, DP, Student, Assignment, Submission)
│   │   ├── connection.py          # Engine + session management
│   │   └── migrations/            # Alembic version control
│   │
│   ├── learning_outcomes/
│   │   ├── __init__.py
│   │   ├── excel_file.py          # Moved from root, added logging + data extraction
│   │   ├── parser.py              # Extract LOCs/SLOCs/DPs from validated Excel
│   │   └── importer.py            # Insert into DB with upsert logic
│   │
│   ├── canvas/
│   │   ├── __init__.py
│   │   ├── client.py              # Canvas API client (sync for now)
│   │   ├── fetchers.py            # Student/assignment/submission fetchers
│   │   └── cache.py               # Optional response caching (Phase 3)
│   │
│   ├── grading/
│   │   ├── __init__.py
│   │   ├── calculator.py          # Per-student grade calc
│   │   └── aggregator.py          # LOC/SLOC aggregation (Phase 4)
│   │
│   ├── reporting/
│   │   ├── __init__.py
│   │   ├── generator.py           # HTML generation (Phase 5)
│   │   ├── visualizations.py      # Matplotlib donut charts (Phase 5)
│   │   └── exporters.py           # PDF export via Playwright (Phase 5)
│   │
│   ├── cli.py                     # Typer CLI entry point
│   │
│   └── utils/
│       ├── __init__.py
│       └── helpers.py             # Utility functions
│
├── tgr_tests/                     # Test suite (mirroring tgr/)
│   ├── __init__.py
│   ├── conftest.py                # Pytest fixtures, test DB setup
│   ├── test_excel_file.py
│   ├── test_models.py
│   ├── test_parser.py
│   ├── test_importer.py
│   ├── fixtures/
│   │   ├── sample_locs.xlsx       # Test Excel file
│   │   ├── mock_canvas_api.py     # Mock Canvas responses
│   │   └── test_data.py           # Fixture factories
│   └── integration/
│       └── test_e2e_workflow.py   # End-to-end tests
│
├── templates/
│   └── grade_report.html          # Jinja2 template
│
├── config/
│   ├── config.yaml                # Logging + app settings
│   └── modules.yaml               # Module registry (Module code → name → abbreviation)
│
├── alembic/
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
│
├── logs/                          # Created at runtime
├── data/                          # Excel files, cache
├── grade-reports/                 # Generated reports
│
├── main.py                        # CLI entry: imports tgr.cli
├── pyproject.toml                 # Dependencies
├── alembic.ini                    # Alembic config
├── config.yaml                    # Logging + app config
├── .env                           # Secrets (Canvas URL, API key)
├── .env.example                   # Template
├── .python-version                # Python 3.10+
├── .gitignore
├── README.md                      # Quick start guide
└── docs/
    ├── ARCHITECTURE.md            # Detailed design
    ├── DECISIONS.md               # Architecture Decision Records
    ├── DEVELOPMENT.md             # Developer guide
    └── API.md                     # Module interfaces
```

---

## Term Representation

**Format:** `{academic_year}-{semester_number}`  
**Examples:**
- `2025-2026-01` (Fall 2025 / Autumn semester)
- `2025-2026-02` (Spring 2026 / Spring semester)

**Database Fields:**
```python
Term
  - id (PK)
  - academic_year (2025)  # Start year of academic year
  - semester (1 or 2)      # 01 = first, 02 = second
  - name ("2025-2026-01")  # Composite display name
  - human_name ("Fall 2025")  # For UI
  - start_date, end_date
  - created_at
```

---

## CLI Commands

**Basic Pattern:** `tgr [command] [subcommand] [options]`

### Core Commands (Phase 1-3)

```bash
# Database setup
tgr init                                    # Create DB, run migrations

# Module management
tgr module list                             # List all modules
tgr module add CEGC "Clean Energy..."       # Add module to registry

# Term management
tgr term list                               # List terms
tgr term add 2025-2026-01 "Fall 2025"       # Add term

# Course setup (links Module + Term + Canvas)
tgr course setup \
  --module CEGC \
  --term 2025-2026-01 \
  --canvas-id 123456

# Import learning outcomes from Excel
tgr locs import \
  --course CEGC/2025-2026-01 \
  --file loc-sloc-dp.xlsx

# Sync Canvas data
tgr sync \
  --course CEGC/2025-2026-01 \
  --what students|assignments|submissions|all

# Generate reports (Phase 5)
tgr report generate \
  --course CEGC/2025-2026-01 \
  --student-id 12345

tgr report generate \
  --course CEGC/2025-2026-01 \
  --all

# Future: Sync all modules
# tgr sync --all
```

---

## Configuration Files

### `.env` (Secrets - NOT committed)
```bash
CANVAS_URL=https://canvas.example.com
CANVAS_API_TOKEN=your_api_token_here
DATABASE_URL=sqlite:///tgr.db
LOG_LEVEL=INFO
```

### `.env.example` (Template - committed)
```bash
CANVAS_URL=https://canvas.example.com
CANVAS_API_TOKEN=
DATABASE_URL=sqlite:///tgr.db
LOG_LEVEL=INFO
```

### `config.yaml` (App config - committed)
```yaml
version: 1

formatters:
  file:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    datefmt: '%d-%b-%y %H:%M:%S'
  console:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    datefmt: '%Y-%m-%d %H:%M:%S'

handlers:
  file:
    class: logging.handlers.RotatingFileHandler
    level: DEBUG
    formatter: file
    filename: logs/tgr.log
    maxBytes: 10485760  # 10MB
    backupCount: 5
    delay: False
  console:
    class: logging.StreamHandler
    level: INFO
    formatter: console
    stream: ext://sys.stdout

root:
  level: DEBUG
  handlers: [file, console]

loggers:
  tgr.database:
    level: INFO
  tgr.canvas:
    level: INFO
  tgr.learning_outcomes:
    level: DEBUG
```

### `config/modules.yaml` (Module registry - committed)
```yaml
modules:
  - code: CEGC
    name: "Clean Energy and Green Chemistry"
    abbreviation: "CEGC"
    
  - code: HIST
    name: "History"
    abbreviation: "HIST"
    
  # ... 9 modules total
```

---

## Testing Strategy

**Framework:** pytest  
**Type Checking:** mypy --strict  
**Structure:** Mirror source (`tgr_tests/test_*.py` ↔ `tgr/*.py`)

### Fixtures (Shared in `conftest.py`)
- Test database (in-memory SQLite)
- Sample Excel file with LOCs/SLOCs/DPs
- Mock Canvas API responses
- Temporary directories for reports

### Test Phases
1. **Unit tests** (Phase 1-2): Models, parsers, calculators
2. **Integration tests** (Phase 3): Database + Canvas sync
3. **E2E tests** (Phase 5): Full workflow (import → sync → report)

---

## Development Workflow (Tomorrow)

### Morning: Foundation
1. Create package structure (`tgr/`, `tgr_tests/`)
2. Add logging config + integrate with ExcelFile
3. Create database models + migrations
4. Write first tests for ExcelFile

### Afternoon: CLI & Import
5. Add data extraction methods to ExcelFile
6. Create Typer CLI skeleton
7. Implement `tgr locs import` command
8. Test end-to-end: Excel → Database

---

## Key Decisions Logged

✅ **Single-responsibility modules:** Each module does one thing  
✅ **Type hints everywhere:** mypy --strict from day 1  
✅ **Tests alongside code:** pytest from day 1  
✅ **Multi-module architecture:** Design now for 9+ modules, one at a time for CLI  
✅ **Academic year term format:** 2025-2026-01, 2025-2026-02  
✅ **SQLite locally:** Easy to backup, move, inspect  
✅ **Defer async/multi-course:** Easy to add later, isolated to client layer  
✅ **Config via YAML + .env:** Flexible, human-readable, secure  

---

## Next Session: Starting Phase 1

**Objective:** Working `tgr init` and `tgr locs import` commands

**Files to Create:**
- `tgr/__init__.py` (package init)
- `tgr/config.py` (config loader)
- `tgr/database/models.py` (SQLAlchemy ORM)
- `tgr/database/connection.py` (session management)
- `tgr/learning_outcomes/excel_file.py` (moved + enhanced)
- `tgr/learning_outcomes/parser.py` (data extraction)
- `tgr/learning_outcomes/importer.py` (DB insert)
- `tgr/cli.py` (Typer commands)
- `tgr_tests/conftest.py` (pytest fixtures)
- `tgr_tests/test_excel_file.py` (first tests)
- `config.yaml` (logging config)
- `config/modules.yaml` (module registry)
- Alembic migrations

**Deliverable:** `tgr locs import` works end-to-end

---

## Questions Resolved ✅

- [x] Directory naming: `toe-grade-reporter/` with `tgr/` package
- [x] CLI command: `tgr`
- [x] Term format: Academic year `2025-2026-01`
- [x] Database: SQLite local
- [x] Multi-module: Design for it, single-module CLI focus
- [x] Canvas mapping: New course per semester per module
- [x] Testing: pytest + mypy --strict from day 1
- [x] Config: YAML + .env
- [x] Development: Together, step-by-step

---

**Ready for tomorrow!** 🚀
