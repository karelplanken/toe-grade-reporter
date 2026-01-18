# TGR - Execution Plan for Phase 1 (Tomorrow)

**Phase 1 Goal:** Complete database foundation + Excel import pipeline  
**Deliverable:** `tgr locs import` working end-to-end  
**Time Estimate:** 6-8 hours (flexible, collaborative)

---

## Pre-Session Checklist

Before we start tomorrow:

- [ ] Have `loc-sloc-dp.xlsx` ready for testing
- [ ] Confirm Python version: `python --version` (should be 3.10+)
- [ ] Backup current workspace (if nervous about directory rename)
- [ ] Review [docs/PROJECT_STATE.md](docs/PROJECT_STATE.md) for context

---

## Session Structure

### Part A: Setup & Config (30 min)

#### A1. Create Base Package Structure

**What we'll do:**
- Create `tgr/` package with `__init__.py`
- Create subdirectories: `database/`, `learning_outcomes/`, `canvas/`, `grading/`, `reporting/`, `utils/`
- Create `tgr_tests/` directory
- Create `config/` and `logs/` directories
- Move `excel_file.py` from root → `tgr/learning_outcomes/`

**Files to create:**
```
tgr/__init__.py                     (empty or package metadata)
tgr/database/__init__.py            (empty)
tgr/learning_outcomes/__init__.py   (empty)
tgr/canvas/__init__.py              (empty)
tgr/grading/__init__.py             (empty)
tgr/reporting/__init__.py           (empty)
tgr/utils/__init__.py               (empty)
tgr_tests/__init__.py               (empty)
config/__init__.py                  (empty)
logs/.gitkeep                       (empty file for git)
```

#### A2. Create Configuration Files

**Files to create:**

**`config/config.yaml`** (App logging + settings config)
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
    maxBytes: 10485760
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

**`config/modules.yaml`** (Module registry)
```yaml
modules:
  - code: CEGC
    name: "Clean Energy and Green Chemistry"
    abbreviation: "CEGC"
  - code: HIST
    name: "History"
    abbreviation: "HIST"
  # ... add remaining 7 modules
```

**`.env.example`** (Template - committed to git)
```bash
CANVAS_URL=https://canvas.example.com
CANVAS_API_TOKEN=your_api_token_here
DATABASE_URL=sqlite:///tgr.db
LOG_LEVEL=INFO
```

**Update `.env`** (Secrets - NOT committed)
- Copy `.env.example` if it doesn't exist
- Add your actual Canvas credentials

#### A3. Create config.py

**File:** `tgr/config.py`

```python
"""Configuration loader for TGR."""

import logging
import logging.config
import os
from pathlib import Path
from typing import Optional

from dotenv import load_dotenv

# Load .env
load_dotenv()

class Config:
    """Application configuration."""
    
    # Canvas
    CANVAS_URL: str = os.getenv("CANVAS_URL", "https://canvas.example.com")
    CANVAS_API_TOKEN: str = os.getenv("CANVAS_API_TOKEN", "")
    
    # Database
    DATABASE_URL: str = os.getenv("DATABASE_URL", "sqlite:///tgr.db")
    
    # Paths
    PROJECT_ROOT: Path = Path(__file__).parent.parent
    CONFIG_DIR: Path = PROJECT_ROOT / "config"
    LOG_DIR: Path = PROJECT_ROOT / "logs"
    DATA_DIR: Path = PROJECT_ROOT / "data"
    REPORTS_DIR: Path = PROJECT_ROOT / "grade-reports"
    
    # Logging
    LOG_LEVEL: str = os.getenv("LOG_LEVEL", "INFO")
    LOGGING_CONFIG_FILE: Path = CONFIG_DIR / "config.yaml"
    
    # Module registry
    MODULES_FILE: Path = CONFIG_DIR / "modules.yaml"


def setup_logging() -> None:
    """Load logging config from YAML."""
    import yaml
    
    config_file = Config.LOGGING_CONFIG_FILE
    if not config_file.exists():
        raise FileNotFoundError(f"Logging config not found: {config_file}")
    
    with open(config_file) as f:
        log_config = yaml.safe_load(f)
    
    logging.config.dictConfig(log_config)


def get_logger(name: str) -> logging.Logger:
    """Get a named logger."""
    return logging.getLogger(name)


# Ensure directories exist
Config.LOG_DIR.mkdir(exist_ok=True)
Config.DATA_DIR.mkdir(exist_ok=True)
Config.REPORTS_DIR.mkdir(exist_ok=True)

# Setup logging on import
setup_logging()
```

**TODO:** Add validation for Canvas credentials

---

### Part B: Database Models (1 hour)

#### B1. Create SQLAlchemy Models

**File:** `tgr/database/models.py`

We'll write this **together**, step-by-step:

```python
"""SQLAlchemy ORM models for TGR."""

from datetime import datetime
from typing import List

from sqlalchemy import Column, ForeignKey, Integer, String, Float, DateTime, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import relationship

Base = declarative_base()

# Association table for many-to-many: SLOC ↔ Datapoint
sloc_datapoint = Table(
    "sloc_datapoint",
    Base.metadata,
    Column("sub_learning_outcome_id", Integer, ForeignKey("sub_learning_outcome.id"), primary_key=True),
    Column("datapoint_id", Integer, ForeignKey("datapoint.id"), primary_key=True),
)


class Module(Base):
    """Represents an educational module (e.g., CEGC, HIST)."""
    
    __tablename__ = "module"
    
    id: int = Column(Integer, primary_key=True)
    code: str = Column(String(50), unique=True, nullable=False)  # e.g., "CEGC"
    name: str = Column(String(255), nullable=False)
    abbreviation: str = Column(String(50))
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    terms: List["Term"] = relationship("Term", back_populates="module")
    courses: List["Course"] = relationship("Course", back_populates="module")


class Term(Base):
    """Represents an academic term (e.g., 2025-2026-01)."""
    
    __tablename__ = "term"
    
    id: int = Column(Integer, primary_key=True)
    academic_year: int = Column(Integer, nullable=False)  # e.g., 2025
    semester: int = Column(Integer, nullable=False)  # 1 or 2
    name: str = Column(String(50), unique=True, nullable=False)  # e.g., "2025-2026-01"
    human_name: str = Column(String(100))  # e.g., "Fall 2025"
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    courses: List["Course"] = relationship("Course", back_populates="term")


class Course(Base):
    """Represents a Module in a Term on Canvas (e.g., CEGC in Fall 2025)."""
    
    __tablename__ = "course"
    
    id: int = Column(Integer, primary_key=True)
    module_id: int = Column(Integer, ForeignKey("module.id"), nullable=False)
    term_id: int = Column(Integer, ForeignKey("term.id"), nullable=False)
    canvas_course_id: int = Column(Integer, unique=True, nullable=False)
    canvas_course_name: str = Column(String(255))
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    module: Module = relationship("Module", back_populates="courses")
    term: Term = relationship("Term", back_populates="courses")
    learning_outcomes: List["LearningOutcome"] = relationship("LearningOutcome", back_populates="course")
    sub_learning_outcomes: List["SubLearningOutcome"] = relationship("SubLearningOutcome", back_populates="course")
    datapoints: List["Datapoint"] = relationship("Datapoint", back_populates="course")
    students: List["Student"] = relationship("Student", back_populates="course")
    assignments: List["Assignment"] = relationship("Assignment", back_populates="course")


class LearningOutcome(Base):
    """Learning Outcome (LOC) - scoped to course."""
    
    __tablename__ = "learning_outcome"
    
    id: int = Column(Integer, primary_key=True)
    course_id: int = Column(Integer, ForeignKey("course.id"), nullable=False)
    identifier: str = Column(String(50), nullable=False)  # e.g., "LOC-1"
    description: str = Column(String(500))
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    course: Course = relationship("Course", back_populates="learning_outcomes")
    sub_learning_outcomes: List["SubLearningOutcome"] = relationship("SubLearningOutcome", back_populates="learning_outcome")


class SubLearningOutcome(Base):
    """Sub-Learning Outcome (SLOC) - scoped to course."""
    
    __tablename__ = "sub_learning_outcome"
    
    id: int = Column(Integer, primary_key=True)
    course_id: int = Column(Integer, ForeignKey("course.id"), nullable=False)
    learning_outcome_id: int = Column(Integer, ForeignKey("learning_outcome.id"), nullable=False)
    identifier: str = Column(String(50), nullable=False)  # e.g., "SLOC-1.1"
    description: str = Column(String(500))
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    course: Course = relationship("Course", back_populates="sub_learning_outcomes")
    learning_outcome: LearningOutcome = relationship("LearningOutcome", back_populates="sub_learning_outcomes")
    datapoints: List["Datapoint"] = relationship("Datapoint", secondary=sloc_datapoint, back_populates="sub_learning_outcomes")


class Datapoint(Base):
    """Datapoint (DP) - scoped to course."""
    
    __tablename__ = "datapoint"
    
    id: int = Column(Integer, primary_key=True)
    course_id: int = Column(Integer, ForeignKey("course.id"), nullable=False)
    name: str = Column(String(100), nullable=False)
    weight: float = Column(Float, nullable=False)  # Float, as per requirement
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    course: Course = relationship("Course", back_populates="datapoints")
    sub_learning_outcomes: List["SubLearningOutcome"] = relationship("SubLearningOutcome", secondary=sloc_datapoint, back_populates="datapoints")
    assignments: List["Assignment"] = relationship("Assignment", back_populates="datapoint")


class Student(Base):
    """Student - scoped to course."""
    
    __tablename__ = "student"
    
    id: int = Column(Integer, primary_key=True)
    course_id: int = Column(Integer, ForeignKey("course.id"), nullable=False)
    canvas_user_id: int = Column(Integer, nullable=False)
    name: str = Column(String(255), nullable=False)
    email: str = Column(String(255))
    sis_user_id: str = Column(String(50))
    login_id: str = Column(String(255))
    sortable_name: str = Column(String(255))
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    course: Course = relationship("Course", back_populates="students")
    submissions: List["Submission"] = relationship("Submission", back_populates="student")


class Assignment(Base):
    """Assignment - scoped to course."""
    
    __tablename__ = "assignment"
    
    id: int = Column(Integer, primary_key=True)
    course_id: int = Column(Integer, ForeignKey("course.id"), nullable=False)
    canvas_assignment_id: int = Column(Integer, nullable=False)
    datapoint_id: int = Column(Integer, ForeignKey("datapoint.id"), nullable=True)
    name: str = Column(String(255), nullable=False)
    due_at: datetime = Column(DateTime, nullable=True)
    points_possible: float = Column(Float)
    submission_types: str = Column(String(100))
    graded_submissions_exist: bool = Column(bool, default=False)
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    course: Course = relationship("Course", back_populates="assignments")
    datapoint: Datapoint = relationship("Datapoint", back_populates="assignments")
    submissions: List["Submission"] = relationship("Submission", back_populates="assignment")


class Submission(Base):
    """Submission - student's work on assignment."""
    
    __tablename__ = "submission"
    
    id: int = Column(Integer, primary_key=True)
    student_id: int = Column(Integer, ForeignKey("student.id"), nullable=False)
    assignment_id: int = Column(Integer, ForeignKey("assignment.id"), nullable=False)
    canvas_submission_id: int = Column(Integer, nullable=False)
    score: float = Column(Float, nullable=True)
    submitted_at: datetime = Column(DateTime, nullable=True)
    workflow_state: str = Column(String(50))
    created_at: datetime = Column(DateTime, default=datetime.utcnow)
    
    # Relationships
    student: Student = relationship("Student", back_populates="submissions")
    assignment: Assignment = relationship("Assignment", back_populates="submissions")
```

**Action Items During This Part:**
- I'll type-hint everything (mypy --strict ready)
- We'll review relationships together
- Discuss composite uniqueness constraints (e.g., `canvas_user_id` + `course_id`)

#### B2. Create Database Connection

**File:** `tgr/database/connection.py`

```python
"""Database connection and session management."""

from contextlib import contextmanager
from typing import Generator

from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

from tgr.config import Config
from tgr.database.models import Base


# Engine
engine = create_engine(
    Config.DATABASE_URL,
    connect_args={"check_same_thread": False} if "sqlite" in Config.DATABASE_URL else {},
    echo=False  # Set to True to see SQL queries
)

# Session factory
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)


def init_db() -> None:
    """Create all tables."""
    Base.metadata.create_all(bind=engine)


def get_session() -> Session:
    """Get a database session."""
    return SessionLocal()


@contextmanager
def session_context() -> Generator[Session, None, None]:
    """Context manager for database sessions."""
    session = SessionLocal()
    try:
        yield session
        session.commit()
    except Exception:
        session.rollback()
        raise
    finally:
        session.close()
```

---

### Part C: Excel Data Extraction (1 hour)

#### C1. Enhance ExcelFile with Data Extraction

**File:** `tgr/learning_outcomes/excel_file.py` (move from root + enhance)

We'll add four methods to the existing `ExcelFile` class:

```python
def get_learning_outcomes(self) -> list[dict]:
    """
    Extract learning outcomes from 'locs' sheet.
    
    Returns:
        [{'identifier': 'LOC-1', 'description': '...'}]
    """
    df = self._read_sheet('locs')
    return df.rename(columns={'loc': 'identifier'}).to_dict('records')


def get_sub_learning_outcomes(self) -> list[dict]:
    """
    Extract sub-learning outcomes from 'slocs' sheet.
    
    Returns:
        [{'loc': 'LOC-1', 'identifier': 'SLOC-1.1', 'description': '...'}]
    """
    df = self._read_sheet('slocs')
    return df.rename(columns={'sloc': 'identifier'}).to_dict('records')


def get_datapoints(self) -> list[dict]:
    """
    Extract datapoints from 'dps' sheet.
    
    Returns:
        [{'name': 'dp1', 'weight': 1.5}]
    """
    df = self._read_sheet('dps')
    return df.to_dict('records')


def get_sloc_datapoint_mappings(self) -> list[tuple[str, str]]:
    """
    Extract SLOC-DP mappings from sloc sheets.
    
    Returns:
        [('SLOC-1.1', 'dp1'), ('SLOC-1.1', 'dp2'), ...]
    """
    df_slocs = self._read_sheet('slocs')
    sloc_names = df_slocs['sloc'].tolist()
    
    mappings = []
    for sloc_name in sloc_names:
        df_sloc = self._read_sheet(sloc_name)
        for dp_name in df_sloc['dp']:
            mappings.append((sloc_name, dp_name))
    
    return mappings
```

#### C2. Create Parser Module

**File:** `tgr/learning_outcomes/parser.py`

```python
"""Parse Excel data into dictionaries for ORM insertion."""

import logging
from typing import NamedTuple

from tgr.learning_outcomes.excel_file import ExcelFile

logger = logging.getLogger(__name__)


class ParsedLearningOutcomeData(NamedTuple):
    """Parsed learning outcome data."""
    
    learning_outcomes: list[dict]
    sub_learning_outcomes: list[dict]
    datapoints: list[dict]
    sloc_datapoint_mappings: list[tuple[str, str]]


def parse_excel_file(excel_file: ExcelFile) -> ParsedLearningOutcomeData:
    """
    Parse Excel file into structured data.
    
    Args:
        excel_file: Validated ExcelFile instance
        
    Returns:
        ParsedLearningOutcomeData with all extracted data
    """
    logger.info("Parsing learning outcomes from Excel")
    
    locs = excel_file.get_learning_outcomes()
    slocs = excel_file.get_sub_learning_outcomes()
    dps = excel_file.get_datapoints()
    mappings = excel_file.get_sloc_datapoint_mappings()
    
    logger.info(f"Parsed {len(locs)} LOCs, {len(slocs)} SLOCs, {len(dps)} DPs")
    
    return ParsedLearningOutcomeData(
        learning_outcomes=locs,
        sub_learning_outcomes=slocs,
        datapoints=dps,
        sloc_datapoint_mappings=mappings,
    )
```

#### C3. Create Importer Module

**File:** `tgr/learning_outcomes/importer.py`

```python
"""Import parsed learning outcome data into database."""

import logging
from sqlalchemy.orm import Session

from tgr.database.models import LearningOutcome, SubLearningOutcome, Datapoint, Course
from tgr.learning_outcomes.parser import ParsedLearningOutcomeData

logger = logging.getLogger(__name__)


def import_learning_outcomes(
    course: Course,
    parsed_data: ParsedLearningOutcomeData,
    session: Session,
) -> None:
    """
    Import parsed learning outcome data into database.
    
    Uses upsert logic for idempotency (import can be run multiple times).
    
    Args:
        course: Course instance to import into
        parsed_data: ParsedLearningOutcomeData from parser
        session: SQLAlchemy session
    """
    logger.info(f"Importing learning outcomes into course {course.id}")
    
    # Insert LOCs
    for loc_data in parsed_data.learning_outcomes:
        loc = session.query(LearningOutcome).filter_by(
            course_id=course.id,
            identifier=loc_data['identifier']
        ).first()
        
        if loc is None:
            loc = LearningOutcome(
                course_id=course.id,
                identifier=loc_data['identifier'],
                description=loc_data.get('description', '')
            )
            session.add(loc)
            logger.debug(f"Created LOC: {loc_data['identifier']}")
        else:
            # Update existing
            loc.description = loc_data.get('description', '')
            logger.debug(f"Updated LOC: {loc_data['identifier']}")
    
    session.flush()  # Ensure LOCs exist before SLOCs
    
    # Insert SLOCs
    for sloc_data in parsed_data.sub_learning_outcomes:
        loc = session.query(LearningOutcome).filter_by(
            course_id=course.id,
            identifier=sloc_data['loc']
        ).first()
        
        if loc is None:
            raise ValueError(f"LOC {sloc_data['loc']} not found in course")
        
        sloc = session.query(SubLearningOutcome).filter_by(
            course_id=course.id,
            identifier=sloc_data['identifier']
        ).first()
        
        if sloc is None:
            sloc = SubLearningOutcome(
                course_id=course.id,
                learning_outcome_id=loc.id,
                identifier=sloc_data['identifier'],
                description=sloc_data.get('description', '')
            )
            session.add(sloc)
            logger.debug(f"Created SLOC: {sloc_data['identifier']}")
        else:
            sloc.learning_outcome_id = loc.id
            sloc.description = sloc_data.get('description', '')
            logger.debug(f"Updated SLOC: {sloc_data['identifier']}")
    
    session.flush()
    
    # Insert DPs
    for dp_data in parsed_data.datapoints:
        dp = session.query(Datapoint).filter_by(
            course_id=course.id,
            name=dp_data['name']
        ).first()
        
        if dp is None:
            dp = Datapoint(
                course_id=course.id,
                name=dp_data['name'],
                weight=dp_data['weight']
            )
            session.add(dp)
            logger.debug(f"Created DP: {dp_data['name']}")
        else:
            dp.weight = dp_data['weight']
            logger.debug(f"Updated DP: {dp_data['name']}")
    
    session.flush()
    
    # Link SLOCs to DPs
    for sloc_name, dp_name in parsed_data.sloc_datapoint_mappings:
        sloc = session.query(SubLearningOutcome).filter_by(
            course_id=course.id,
            identifier=sloc_name
        ).first()
        
        dp = session.query(Datapoint).filter_by(
            course_id=course.id,
            name=dp_name
        ).first()
        
        if sloc and dp and dp not in sloc.datapoints:
            sloc.datapoints.append(dp)
            logger.debug(f"Linked {sloc_name} → {dp_name}")
    
    session.commit()
    logger.info(f"Learning outcomes import complete for course {course.id}")
```

---

### Part D: CLI Commands (1 hour)

#### D1. Create Typer CLI

**File:** `tgr/cli.py`

```python
"""Typer CLI for TGR."""

import logging
from pathlib import Path
from typing import Optional

import typer
from sqlalchemy.orm import Session

from tgr.database.connection import get_session, init_db, SessionLocal
from tgr.database.models import (
    Assignment,
    Course,
    Datapoint,
    LearningOutcome,
    Module,
    Student,
    SubLearningOutcome,
    Term,
    Submission,
    sloc_datapoint,
)
from tgr.learning_outcomes.excel_file import ExcelFile
from tgr.learning_outcomes.parser import parse_excel_file
from tgr.learning_outcomes.importer import import_learning_outcomes

logger = logging.getLogger(__name__)

app = typer.Typer(help="TGR - Talent-Oriented Grade Reporter")


@app.command()
def init() -> None:
    """Initialize database and create tables."""
    typer.echo("Initializing database...")
    init_db()
    typer.echo("✅ Database initialized")


@app.command()
def course_reset(
    course: str = typer.Option(..., help="Course identifier (e.g., CEGC/2025-2026-01)"),
    drop_students: bool = typer.Option(False, help="Also delete students for this course"),
) -> None:
    """Wipe all course-scoped data so a term can be reloaded cleanly."""
    typer.echo(f"Resetting course data for {course} ...")

    # Parse course identifier (MODULE/TERM)
    parts = course.split("/")
    if len(parts) != 2:
        typer.echo(f"❌ Invalid course format: {course} (expected: MODULE/TERM)", err=True)
        raise typer.Exit(1)

    module_code, term_code = parts

    session = SessionLocal()

    try:
        course_obj = session.query(Course).join(Module).join(Term).filter(
            Module.code == module_code,
            Term.name == term_code,
        ).first()

        if not course_obj:
            typer.echo(f"❌ Course not found: {course}", err=True)
            raise typer.Exit(1)

        # Delete in dependency-safe order
        deleted_submissions = session.query(Submission).filter_by(course_id=course_obj.id).delete()
        deleted_assignments = session.query(Assignment).filter_by(course_id=course_obj.id).delete()
        session.query(sloc_datapoint).filter_by(course_id=course_obj.id).delete()
        deleted_datapoints = session.query(Datapoint).filter_by(course_id=course_obj.id).delete()
        deleted_slocs = session.query(SubLearningOutcome).filter_by(course_id=course_obj.id).delete()
        deleted_locs = session.query(LearningOutcome).filter_by(course_id=course_obj.id).delete()

        if drop_students:
            deleted_students = session.query(Student).filter_by(course_id=course_obj.id).delete()
        else:
            deleted_students = 0

        session.commit()

        typer.echo(
            "✅ Reset complete | "
            f"LOCs:{deleted_locs} SLOCs:{deleted_slocs} DPs:{deleted_datapoints} "
            f"Assignments:{deleted_assignments} Submissions:{deleted_submissions} "
            f"Students:{deleted_students}"
        )

    except Exception as e:
        session.rollback()
        typer.echo(f"❌ Error during reset: {e}", err=True)
        logger.exception("Error resetting course")
        raise typer.Exit(1)

    finally:
        session.close()


@app.command()
def locs_import(
    course: str = typer.Option(..., help="Course identifier (e.g., CEGC/2025-2026-01)"),
    file: Path = typer.Option(..., help="Path to Excel file"),
) -> None:
    """Import learning outcomes from Excel file."""
    typer.echo(f"Importing learning outcomes from {file}")
    
    # Parse course identifier
    parts = course.split("/")
    if len(parts) != 2:
        typer.echo(f"❌ Invalid course format: {course} (expected: MODULE/TERM)", err=True)
        raise typer.Exit(1)
    
    module_code, term_code = parts
    
    # Get session
    session = SessionLocal()
    
    try:
        # Find course
        course_obj = session.query(Course).join(Module).join(Term).filter(
            Module.code == module_code,
            Term.name == term_code
        ).first()
        
        if not course_obj:
            typer.echo(f"❌ Course not found: {course}", err=True)
            raise typer.Exit(1)
        
        # Validate Excel file
        excel_file = ExcelFile(file)
        
        # Parse and import
        parsed_data = parse_excel_file(excel_file)
        import_learning_outcomes(course_obj, parsed_data, session)
        
        typer.echo(f"✅ Learning outcomes imported for course {course}")
        
    except Exception as e:
        typer.echo(f"❌ Error: {e}", err=True)
        logger.exception("Error importing learning outcomes")
        raise typer.Exit(1)
    
    finally:
        session.close()


if __name__ == "__main__":
    app()
```

#### D2. Create main.py Entry Point

**File:** `main.py` (in project root)

```python
"""CLI entry point for TGR."""

from tgr.cli import app

if __name__ == "__main__":
    app()
```

---

### Part E: Testing (1.5 hours)

#### E1. Create Pytest Fixtures

**File:** `tgr_tests/conftest.py`

```python
"""Pytest configuration and shared fixtures."""

import pytest
from pathlib import Path
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from tgr.database.models import Base, Module, Term, Course


@pytest.fixture(scope="session")
def test_db_engine():
    """Create in-memory SQLite database for testing."""
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(bind=engine)
    return engine


@pytest.fixture
def test_session(test_db_engine):
    """Get a test database session."""
    connection = test_db_engine.connect()
    transaction = connection.begin()
    
    TestSessionLocal = sessionmaker(bind=connection)
    session = TestSessionLocal()
    
    yield session
    
    session.close()
    transaction.rollback()
    connection.close()


@pytest.fixture
def sample_module(test_session):
    """Create a sample module for testing."""
    module = Module(code="TEST", name="Test Module", abbreviation="TEST")
    test_session.add(module)
    test_session.commit()
    return module


@pytest.fixture
def sample_term(test_session):
    """Create a sample term for testing."""
    term = Term(academic_year=2025, semester=1, name="2025-2026-01", human_name="Fall 2025")
    test_session.add(term)
    test_session.commit()
    return term


@pytest.fixture
def sample_course(test_session, sample_module, sample_term):
    """Create a sample course for testing."""
    course = Course(
        module_id=sample_module.id,
        term_id=sample_term.id,
        canvas_course_id=123456,
        canvas_course_name="TEST-001-2025F"
    )
    test_session.add(course)
    test_session.commit()
    return course


@pytest.fixture
def sample_excel_file() -> Path:
    """Path to sample Excel file for testing."""
    # Use the actual loc-sloc-dp.xlsx from project
    excel_path = Path(__file__).parent.parent / "loc-sloc-dp.xlsx"
    if not excel_path.exists():
        pytest.skip("Sample Excel file not found")
    return excel_path
```

#### E2. Write First Tests

**File:** `tgr_tests/test_excel_file.py`

```python
"""Tests for ExcelFile validation and data extraction."""

import pytest
from tgr.learning_outcomes.excel_file import ExcelFile


def test_excel_file_validation(sample_excel_file):
    """Test that ExcelFile validates successfully."""
    excel_file = ExcelFile(sample_excel_file)
    assert excel_file is not None
    assert len(excel_file.sheet_names) >= 3


def test_get_learning_outcomes(sample_excel_file):
    """Test LOC extraction."""
    excel_file = ExcelFile(sample_excel_file)
    locs = excel_file.get_learning_outcomes()
    
    assert isinstance(locs, list)
    assert len(locs) > 0
    assert all('identifier' in loc and 'description' in loc for loc in locs)


def test_get_sub_learning_outcomes(sample_excel_file):
    """Test SLOC extraction."""
    excel_file = ExcelFile(sample_excel_file)
    slocs = excel_file.get_sub_learning_outcomes()
    
    assert isinstance(slocs, list)
    assert len(slocs) > 0
    assert all('identifier' in sloc and 'loc' in sloc for sloc in slocs)


def test_get_datapoints(sample_excel_file):
    """Test DP extraction."""
    excel_file = ExcelFile(sample_excel_file)
    dps = excel_file.get_datapoints()
    
    assert isinstance(dps, list)
    assert len(dps) > 0
    assert all('name' in dp and 'weight' in dp for dp in dps)
    assert all(isinstance(dp['weight'], float) and dp['weight'] > 0 for dp in dps)


def test_get_sloc_datapoint_mappings(sample_excel_file):
    """Test SLOC-DP mapping extraction."""
    excel_file = ExcelFile(sample_excel_file)
    mappings = excel_file.get_sloc_datapoint_mappings()
    
    assert isinstance(mappings, list)
    assert len(mappings) > 0
    assert all(isinstance(m, tuple) and len(m) == 2 for m in mappings)
```

#### E3. Test Models

**File:** `tgr_tests/test_models.py`

```python
"""Tests for database models."""

import pytest
from tgr.database.models import Module, Term, Course


def test_module_creation(test_session):
    """Test creating a module."""
    module = Module(code="TEST", name="Test", abbreviation="T")
    test_session.add(module)
    test_session.commit()
    
    assert module.id is not None
    assert module.code == "TEST"


def test_course_relationships(test_session, sample_course, sample_module, sample_term):
    """Test course relationships."""
    assert sample_course.module_id == sample_module.id
    assert sample_course.term_id == sample_term.id
    assert sample_course.module.code == "TEST"
```

---

### Part F: Integration Test (30 min)

#### F1. End-to-End Test

**File:** `tgr_tests/test_e2e_import.py`

```python
"""End-to-end test of Excel import workflow."""

import pytest
from tgr.learning_outcomes.excel_file import ExcelFile
from tgr.learning_outcomes.parser import parse_excel_file
from tgr.learning_outcomes.importer import import_learning_outcomes


def test_e2e_import_workflow(test_session, sample_course, sample_excel_file):
    """Test full import pipeline: Excel → Parser → Importer."""
    # Validate Excel
    excel_file = ExcelFile(sample_excel_file)
    
    # Parse
    parsed_data = parse_excel_file(excel_file)
    
    # Import
    import_learning_outcomes(sample_course, parsed_data, test_session)
    
    # Verify
    from tgr.database.models import LearningOutcome, SubLearningOutcome, Datapoint
    
    locs = test_session.query(LearningOutcome).filter_by(course_id=sample_course.id).count()
    slocs = test_session.query(SubLearningOutcome).filter_by(course_id=sample_course.id).count()
    dps = test_session.query(Datapoint).filter_by(course_id=sample_course.id).count()
    
    assert locs > 0
    assert slocs > 0
    assert dps > 0
```

---

## Running Tests

```bash
# Run all tests
pytest tgr_tests/ -v

# Run with coverage
pytest tgr_tests/ --cov=tgr --cov-report=html

# Type check
mypy tgr/ --strict

# Run specific test
pytest tgr_tests/test_excel_file.py::test_get_learning_outcomes -v
```

---

## Summary of Files Created

By end of session:

```
tgr/
├── __init__.py
├── config.py                    # Config loader
├── cli.py                       # Typer CLI
├── database/
│   ├── __init__.py
│   ├── models.py               # SQLAlchemy models
│   └── connection.py           # Session mgmt
├── learning_outcomes/
│   ├── __init__.py
│   ├── excel_file.py           # Moved + enhanced
│   ├── parser.py               # Data extraction
│   └── importer.py             # DB import
└── utils/
    └── __init__.py

tgr_tests/
├── __init__.py
├── conftest.py                 # Fixtures
├── test_excel_file.py
├── test_models.py
└── test_e2e_import.py

config/
├── config.yaml                 # Logging config
└── modules.yaml                # Module registry

main.py                          # CLI entry
```

---

## Next Session Deliverables

✅ `tgr init` - Creates database  
✅ `tgr locs-import` - Imports Excel → Database  
✅ All tests passing (pytest)  
✅ mypy --strict clean  
✅ Full logging to file + console  

---

**Ready for tomorrow!** 🚀

Ask questions now if anything is unclear!
