# TGR Project - Documentation Index

**Last Updated:** January 17, 2026  
**Next Session:** Tomorrow (Phase 1 implementation)

---

## Quick Navigation

### 🎯 Start Here (In Order)

1. **[docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md)** (5 min read)
   - What we did today
   - What happens tomorrow
   - Key decisions made

2. **[docs/PROJECT_STATE.md](docs/PROJECT_STATE.md)** (10 min read)
   - Architecture overview
   - Data model (Module → Term → Course)
   - Configuration & testing strategy

3. **[docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md)** (Reference tomorrow)
   - Step-by-step Phase 1 tasks
   - Complete code snippets (copy-paste ready)
   - Test fixtures and examples

4. **[docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md)** (Reference tomorrow)
   - Exact directories to create
   - Files to add/move
   - Before/after structure

---

### 📚 Reference Documents

**Project Planning:**
- [docs/PROJECT_PLAN.md](docs/PROJECT_PLAN.md) - 6-phase implementation roadmap
- [README.md](README.md) - Quick start guide

**Architecture (Why things are designed this way):**
- [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md) - Decisions & architecture
- [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) - Implementation details

**Implementation (How to build it):**
- [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) - Complete code to write
- [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) - Directories & structure

---

### 🔍 For Tomorrow's Session

**Required Reading Before Starting:**
- ✅ [docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md) - Refresh on decisions
- ✅ [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) - Know what to create

**Open During Work:**
- 📖 [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) - Copy code from here
- 📖 [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md) - Reference models & config

---

## Document Purposes

| File | Purpose | Length | Read When |
|------|---------|--------|-----------|
| [docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md) | What we did today, what's next | 2K words | Start (today) |
| [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md) | Architecture, data model, decisions | 4K words | Before tomorrow |
| [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) | Step-by-step Phase 1 with code | 8K words | During tomorrow |
| [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) | Directory layout, file list | 2K words | During tomorrow |
| [docs/PROJECT_PLAN.md](docs/PROJECT_PLAN.md) | 6-phase roadmap, timeline | 5K words | Reference (not urgent) |
| [README.md](README.md) | Quick start, overview | 1K words | When setting up |

---

## Key Decisions (Summary)

**Database:** SQLite (local, easy)  
**API:** Sync canvasapi initially (async in Phase 3)  
**Testing:** pytest + mypy --strict from day 1  
**Config:** YAML + .env (secure, flexible)  
**Term Format:** 2025-2026-01 (academic year standard)  
**Package:** `tgr` (CLI command: `tgr`)  
**Directory:** `toe-grade-reporter/` (with hyphens)  

---

## Phase 1 Goals (Tomorrow)

✅ Create `tgr/` package structure  
✅ Setup logging, config, database models  
✅ Move ExcelFile to `tgr/learning_outcomes/`  
✅ Add data extraction methods  
✅ Create parser & importer  
✅ Build Typer CLI (`init`, `locs-import`)  
✅ Write pytest fixtures & tests  
✅ Verify everything works end-to-end  

**Deliverable:** `tgr locs-import` command working

---

## Frequently Asked Questions

**Q: Where do I find the code to write?**  
A: [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) has all code snippets ready to copy.

**Q: What directories should I create?**  
A: [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) shows exactly what to create.

**Q: How long will tomorrow take?**  
A: 6-8 hours collaborative. Can break into 2 sessions (morning + afternoon).

**Q: What if I get stuck?**  
A: Reference [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md) for architecture, or ask questions.

**Q: Can I look at the existing code first?**  
A: Yes! The original system is in `original_project/` (reference only).

**Q: What about the later phases?**  
A: [docs/PROJECT_PLAN.md](docs/PROJECT_PLAN.md) covers all 6 phases, but Phase 1 is tomorrow's focus.

---

## File Status

| File | Status | Purpose |
|------|--------|---------|
| SESSION_SUMMARY.md | ✅ Ready | Today's summary |
| docs/PROJECT_STATE.md | ✅ Ready | Architecture reference |
| docs/EXECUTION_PLAN.md | ✅ Ready | Implementation guide |
| STRUCTURE_TEMPLATE.md | ✅ Ready | Directory template |
| PROJECT_PLAN.md | ✅ Ready | 6-phase roadmap |
| README.md | ✅ Updated | Quick start |
| (All other files) | ✅ Clean | Ready for Phase 1 |

---

## How to Use These Docs

### Before Tomorrow (Tonight)
1. Read [docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md) (2-5 min)
2. Skim [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md) (10 min, optional)
3. Get good sleep! ☕

### Start of Tomorrow Session
1. Open [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) → Create directories
2. Open [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) → Follow section by section
3. Copy code from EXECUTION_PLAN into editor
4. Write tests as you go

### If You Get Lost
1. Check [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md) for "why" questions
2. Check [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md) for "how" questions
3. Check [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) for file locations

---

## What NOT to Look At

❌ `dump/` - Archived code from Phase 0  
❌ `original_project/` - Legacy system (reference only)  
❌ `benchmark.py` - Will be moved later  

Focus on `tgr/` package we'll create tomorrow.

---

## Success Criteria

By end of Phase 1 tomorrow, you should have:

- ✅ Runnable `tgr init` command
- ✅ Runnable `tgr locs-import` command  
- ✅ SQLite database with learning outcomes
- ✅ 100% type-safe code (mypy --strict)
- ✅ All tests passing
- ✅ Full logging to file + console

---

## Questions to Ask Yourself

**Before starting tomorrow:**
- Have you read [docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md)?
- Do you understand Module → Term → Course hierarchy?
- Do you know what files to create? (Check [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md))

**During implementation:**
- Should I refer to [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md)?
- Is this code type-safe? (Will mypy --strict pass?)
- Do I have a test for this feature?

---

## Next Steps

1. **Tonight:** Read [docs/SESSION_SUMMARY.md](docs/SESSION_SUMMARY.md)
2. **Tomorrow Morning:** Open [docs/STRUCTURE_TEMPLATE.md](docs/STRUCTURE_TEMPLATE.md) and [docs/EXECUTION_PLAN.md](docs/EXECUTION_PLAN.md)
3. **Tomorrow Afternoon:** Celebrate working `tgr locs-import` command! 🎉

---

**Ready? Let's build something great!** 🚀

See you tomorrow!
