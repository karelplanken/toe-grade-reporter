# TGR - Talent-Oriented Grade Reporter

CLI app for Canvas LMS grade reporting with learning outcome mapping, multi-module support, and automated HTML/PDF report generation.

## Project Status

**Phase:** 0-1 Foundation (In Progress)  
**Latest:** Excel validation optimized (55× faster), project architecture finalized

## Documentation

**👉 [START HERE: DOCUMENTATION_INDEX.md](docs/DOCUMENTATION_INDEX.md)** - Navigation guide to all docs

**Main references:**
- **[docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md)** - Today's summary, what's next
- **[docs/PROJECT_STATE.md](docs/PROJECT_STATE.md)** - Architecture, data model, decisions
- **[docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md)** - Step-by-step Phase 1 implementation
- **[docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md)** - Exact directory structure to create
- **[docs/PROJECT_PLAN.md](docs/PROJECT_PLAN.md)** - 6-phase implementation roadmap

## What's Done

✅ **Excel Validation** (Phase 0)
- `ExcelFile` class with single-pass sheet caching
- Vectorized pandas validation (55× faster: ~59ms vs ~3200ms)
- Float validation for datapoint weights
- Benchmark harness

✅ **Architecture** (Phase 0)
- Multi-module, multi-term design
- SQLite database schema
- Typer CLI command structure
- pytest + mypy --strict from day 1

## Quick Start (When Ready)

```bash
# Setup
python -m venv .venv
source .venv/bin/activate
pip install -e .

# Run CLI
tgr init
tgr locs-import --course CEGC/2025-2026-01 --file loc-sloc-dp.xlsx
```

## Development

```bash
# Type checking
mypy tgr/ --strict

# Tests
pytest tgr_tests/ -v --cov=tgr

# Linting (optional)
ruff check tgr/
```

## Key Technologies

- **SQLAlchemy:** ORM for relational data
- **Typer:** CLI framework
- **pytest:** Testing
- **pandas:** Excel processing
- **Jinja2:** HTML templating (Phase 5)
- **Playwright:** PDF generation (Phase 5)
