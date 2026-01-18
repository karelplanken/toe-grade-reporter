# TGR - Phase 1 Checklist for Tomorrow

**Print this out or bookmark it!** ✅

---

## Pre-Session (Before Starting)

- [ ] Reread [docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md) (2 min)
- [ ] Have [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) open
- [ ] Have [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) open
- [ ] Coffee/tea ready ☕
- [ ] Have `loc-sloc-dp.xlsx` file available
- [ ] Confirm Python 3.10+ with `python --version`

---

## Phase 1 Tasks

### Part A: Setup & Config (30 min)

**A1: Create Base Package Structure**
- [ ] Create `tgr/` directory structure
  - [ ] `tgr/__init__.py`
  - [ ] `tgr/database/__init__.py`
  - [ ] `tgr/learning_outcomes/__init__.py`
  - [ ] `tgr/canvas/__init__.py`
  - [ ] `tgr/grading/__init__.py`
  - [ ] `tgr/reporting/__init__.py`
  - [ ] `tgr/utils/__init__.py`
- [ ] Create `tgr_tests/__init__.py`
- [ ] Create `config/__init__.py`
- [ ] Create marker directories:
  - [ ] `mkdir logs data grade-reports templates`
  - [ ] Add `.gitkeep` files to empty dirs

**A2: Create Configuration Files**
- [ ] `config/config.yaml` (copy from EXECUTION_PLAN.md)
- [ ] `config/modules.yaml` (copy from EXECUTION_PLAN.md, add all 9 modules)
- [ ] `.env.example` (copy from EXECUTION_PLAN.md)

**A3: Create tgr/config.py**
- [ ] Copy code from EXECUTION_PLAN Part A3
- [ ] Verify imports work
- [ ] Test with `python -c "from tgr.config import Config"`

---

### Part B: Database Models (1 hour)

**B1: Create tgr/database/models.py**
- [ ] Copy complete models code from EXECUTION_PLAN.md
- [ ] All 8 models:
  - [ ] Module
  - [ ] Term
  - [ ] Course
  - [ ] LearningOutcome
  - [ ] SubLearningOutcome
  - [ ] Datapoint
  - [ ] Student
  - [ ] Assignment
  - [ ] Submission
- [ ] Association table: `sloc_datapoint`
- [ ] All relationships defined
- [ ] Run `mypy tgr/database/models.py --strict` ✅

**B2: Create tgr/database/connection.py**
- [ ] Copy code from EXECUTION_PLAN Part B2
- [ ] Test with `python -c "from tgr.database.connection import init_db"`

---

### Part C: Excel Data Extraction (1 hour)

**C1: Move and Enhance ExcelFile**
- [ ] Move `excel_file.py` → `tgr/learning_outcomes/excel_file.py`
- [ ] Add 4 data extraction methods (from EXECUTION_PLAN Part C1):
  - [ ] `get_learning_outcomes()`
  - [ ] `get_sub_learning_outcomes()`
  - [ ] `get_datapoints()`
  - [ ] `get_sloc_datapoint_mappings()`
- [ ] Test: `python -c "from tgr.learning_outcomes.excel_file import ExcelFile"`

**C2: Create tgr/learning_outcomes/parser.py**
- [ ] Copy code from EXECUTION_PLAN Part C2
- [ ] Verify imports and types

**C3: Create tgr/learning_outcomes/importer.py**
- [ ] Copy code from EXECUTION_PLAN Part C3
- [ ] Verify all imports
- [ ] Run `mypy tgr/learning_outcomes/importer.py --strict` ✅

---

### Part D: CLI Commands (1 hour)

**D1: Create tgr/cli.py**
- [ ] Copy code from EXECUTION_PLAN Part D1
- [ ] Two commands:
  - [ ] `init` - Create database
  - [ ] `locs-import` - Import Excel
- [ ] Run `mypy tgr/cli.py --strict` ✅

**D2: Create main.py**
- [ ] Copy code from EXECUTION_PLAN Part D2
- [ ] Test: `python main.py --help`

**D3: Update pyproject.toml**
- [ ] Add dependencies:
  - [ ] sqlalchemy
  - [ ] typer
  - [ ] pyyaml
  - [ ] python-dotenv
  - [ ] pytest (dev)
- [ ] Run `pip install -e .`

---

### Part E: Testing (1.5 hours)

**E1: Create tgr_tests/conftest.py**
- [ ] Copy fixtures from EXECUTION_PLAN Part E1
- [ ] Verify all fixtures work:
  - [ ] test_db_engine
  - [ ] test_session
  - [ ] sample_module
  - [ ] sample_term
  - [ ] sample_course
  - [ ] sample_excel_file

**E2: Create tgr_tests/test_excel_file.py**
- [ ] Copy tests from EXECUTION_PLAN Part E2
- [ ] All 5 tests:
  - [ ] test_excel_file_validation
  - [ ] test_get_learning_outcomes
  - [ ] test_get_sub_learning_outcomes
  - [ ] test_get_datapoints
  - [ ] test_get_sloc_datapoint_mappings
- [ ] Run `pytest tgr_tests/test_excel_file.py -v` ✅

**E3: Create tgr_tests/test_models.py**
- [ ] Copy tests from EXECUTION_PLAN Part E3
- [ ] Run `pytest tgr_tests/test_models.py -v` ✅

---

### Part F: Integration Test (30 min)

**F1: Create tgr_tests/test_e2e_import.py**
- [ ] Copy test from EXECUTION_PLAN Part F1
- [ ] Run `pytest tgr_tests/test_e2e_import.py -v` ✅

---

## Verification Checklist

### Code Quality
- [ ] `mypy tgr/ --strict` passes (no errors)
- [ ] `pytest tgr_tests/ -v` all green
- [ ] `pytest tgr_tests/ --cov=tgr` shows coverage
- [ ] No print statements (use logging instead)
- [ ] All functions have docstrings
- [ ] All functions type-hinted

### Functionality
- [ ] `python main.py --help` shows commands
- [ ] `python main.py init` creates database (tgr.db)
- [ ] `python main.py locs-import --help` shows options
- [ ] `python main.py locs-import --course CEGC/2025-2026-01 --file loc-sloc-dp.xlsx` works
- [ ] Database file created with all tables
- [ ] Learning outcomes inserted correctly

### Logging
- [ ] Log file created at `logs/tgr.log`
- [ ] Console output visible during import
- [ ] Log level set correctly (DEBUG in file, INFO in console)

### Files & Structure
- [ ] All directories created (check `ls -la tgr/`)
- [ ] No orphaned files in root (except main.py, config.yaml, etc.)
- [ ] .gitignore updated for `*.db`, `logs/`, `tgr.db`

---

## Common Issues & Solutions

**Problem:** `ModuleNotFoundError: No module named 'tgr'`  
**Solution:** Run `pip install -e .` from project root

**Problem:** `FileNotFoundError: config.yaml not found`  
**Solution:** Verify `config/config.yaml` exists and has correct path in `tgr/config.py`

**Problem:** Tests fail with "in-memory database error"  
**Solution:** Ensure `Base.metadata.create_all()` is called in test fixture

**Problem:** `mypy` errors about type hints  
**Solution:** Check imports - use `from typing import Optional, List` at top of file

**Problem:** Import errors in test files  
**Solution:** Ensure `tgr_tests/__init__.py` exists (can be empty)

---

## Timing Guide

| Part | Task | Est. Time |
|------|------|-----------|
| A | Setup & Config | 30 min |
| B | Database Models | 60 min |
| C | Excel Extraction | 60 min |
| D | CLI Commands | 60 min |
| E | Testing | 90 min |
| F | Integration Test | 30 min |
| **Total** | **All parts** | **6-8 hours** |

*Can split into 2 sessions (morning + afternoon)*

---

## Success Criteria

At end of Phase 1, you should have:

- ✅ `tgr/` package fully structured
- ✅ Logging working (file + console)
- ✅ Database models defined (9 models)
- ✅ Excel validation + parsing working
- ✅ Typer CLI with 2 commands
- ✅ 10+ tests, all passing
- ✅ 100% type-safe (mypy --strict)
- ✅ Full end-to-end workflow: `Excel → Parse → Import → Database`

---

## Git Commits (Optional But Recommended)

```bash
git add -A
git commit -m "Phase 1.A: Create package structure and config"
git commit -m "Phase 1.B: Add SQLAlchemy models and database connection"
git commit -m "Phase 1.C: Add Excel parser and importer"
git commit -m "Phase 1.D: Add Typer CLI with init and locs-import"
git commit -m "Phase 1.E: Add comprehensive pytest fixtures and tests"
git commit -m "Phase 1: Phase 1 complete - tgr locs-import working end-to-end"
```

---

## Quick Reference Links

- **Code to copy:** [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md)
- **File structure:** [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md)
- **Architecture:** [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md)
- **Questions?** [docs/DOCUMENTATION_INDEX.md](docs/DOCUMENTATION_INDEX.md)

---

## Final Reminder

> You're not just writing code. You're building a modular, professional-grade system that can scale to 9 modules, 500+ students, and multiple features. 

**Take your time. Ask questions. Test thoroughly.** ✅

---

## Tomorrow Morning Prep

1. ☕ Make coffee
2. 📖 Read [docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md)
3. 🎯 Open [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md)
4. 📋 Print (or bookmark) this checklist
5. 🚀 Start with Part A!

---

**Let's build TGR! 🚀**

You've got this! 💪
