# Incident Postmortem Tracker

[![Python Version](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.104%2B-009688.svg)](https://fastapi.tiangolo.com/)
[![HTMX](https://img.shields.io/badge/HTMX-1.9.10-336699.svg)](https://htmx.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Code Style: Black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Tests](https://img.shields.io/badge/tests-passing-brightgreen.svg)]()

A centralized, full-stack operational postmortem platform for documenting service outages, analyzing root causes, and tracking preventive action items. 

The **Incident Postmortem Tracker** provides engineering teams with a single source of truth to reconstruct chronological event timelines, calculate detection and resolution latencies, conduct Five Whys root cause analyses, and enforce accountability on remediation tasks to prevent recurring outages.

---

## Problem Statement & Target Audience

### The Problem
During operational outages, incident documentation often becomes scattered across transient Slack channels, ephemeral shared documents, and disjointed issue trackers. This fragmentation leads to critical failure modes:
* **Loss of Institutional Context:** Key decisions, timestamps, and responder actions are lost after an incident resolves.
* **Untracked Remediation:** Action items agreed upon in post-incident reviews are never assigned or tracked to completion, leading to repeatable failures.
* **Lack of Standardized Metrics:** Calculating key reliability metrics—such as Mean Time to Detect (MTTD), Mean Time to Acknowledge (MTTA), and Mean Time to Resolve (MTTR)—is manually intensive and error-prone.

### Target Audience
* **Site Reliability Engineers (SREs) & DevOps Teams** seeking structured incident logging and automated ingestion from alert providers.
* **Engineering Managers & Tech Leads** requiring clear visibility into operational reliability, impact metrics, and action item accountability.
* **Software Operations & Platform Engineering** needing an accessible UI driven by lightweight server-rendered interactions.

---

## Key Features

* **Chronological Timeline Editor:** Log milestone timestamps (Detection, Escalation, Mitigation, Resolution), severity adjustments, and key responder actions in real-time or during post-incident retrospectives.
* **Structured Postmortem Builder:** Formulate comprehensive post-incident reports supporting Five Whys analysis, identification of contributing factors, financial/user impact metrics, and automated detection latency calculations.
* **Remediation Action Item Tracker:** Directly link preventive action items (preventative, corrective, detection) to incident records with strict status tracking (`Open`, `In Progress`, `Completed`), priority tiers, and assigned owners.
* **Automated Alert Ingestion API:** RESTful webhook endpoints built with Pydantic payload validation to automatically ingest alerts from monitoring tools (Prometheus, Datadog, Grafana) and create preliminary incident drafts.

---

## Architecture & System Design

The application follows a modular, server-driven architecture utilizing FastAPI for backend routing and logic, rendered via Jinja2 and HTMX for dynamic front-end UI updates without heavy JavaScript framework overhead.

```
                   +------------------------------------+
                   | Monitoring Alerts / Webhooks / UI  |
                   +-----------------+------------------+
                                     |
                                     v
                       +-------------+------------+
                       |   src/api.py (FastAPI)   |
                       |  - RESTful Endpoints     |
                       |  - Webhook Handlers      |
                       |  - Jinja2 / HTMX Views   |
                       +-------------+------------+
                                     |
                                     v
                       +-------------+------------+
                       |  src/core.py (Logic)     |
                       |  - DB Session Handler    |
                       |  - Latency Metrics Engine|
                       |  - CRUD Controller       |
                       +-------------+------------+
                                     |
                                     v
                       +-------------+------------+
                       |  src/models.py (ORM)     |
                       |  - Incident / Event      |
                       |  - Root Cause Analysis   |
                       |  - Action Item Schemas   |
                       +-------------+------------+
                                     |
                                     v
                       +-------------+------------+
                       |    SQLite Database       |
                       +--------------------------+
```

### Module Breakdown

| Module File | Purpose & Responsibilities |
| :--- | :--- |
| **`src/models.py`** | SQLAlchemy ORM models (`Incident`, `TimelineEvent`, `RootCauseAnalysis`, `ActionItem`) and Pydantic validation schemas for API requests, responses, and webhook payload structures. |
| **`src/core.py`** | Database engine configuration, session management context wrappers, CRUD operators, automated metric calculations (MTTD, MTTR), and postmortem logic execution. |
| **`src/api.py`** | FastAPI routes serving full JSON REST endpoints, alert/on-call webhook ingestion points, and server-rendered HTML views powered by Jinja2 templates and HTMX. |
| **`src/main.py`** | Application lifecycle entrypoint, FastAPI initialization, database table creation hooks on startup, and Uvicorn web server launcher. |

---

## Technology Stack

* **Backend Framework:** [FastAPI](https://fastapi.tiangolo.com/) (Python 3.10+)
* **Database & ORM:** [SQLAlchemy](https://www.sqlalchemy.org/) with [SQLite](https://sqlite.org/)
* **Data Validation:** [Pydantic v2](https://docs.pydantic.dev/)
* **Frontend Rendering:** [Jinja2](https://jinja.palletsprojects.com/) + [HTMX](https://htmx.org/)
* **Testing Suite:** [pytest](https://docs.pytest.org/)

---

## Installation & Virtual Environment Setup

### Prerequisites
* Python 3.10 or higher
* `pip` and `venv` installed

### Step-by-step Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-org/incident-postmortem-tracker.git
   cd incident-postmortem-tracker
   ```

2. **Create and activate a virtual environment:**
   * **Linux/macOS:**
     ```bash
     python3 -m venv venv
     source venv/bin/activate
     ```
   * **Windows:**
     ```cmd
     python -m venv venv
     venv\Scripts\activate
     ```

3. **Install dependencies:**
   ```bash
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

---

## Environment Variables Configuration

Copy the sample environment file to configure instance settings:

```bash
cp .env.example .env
```

### Configuration Options (`.env`)

```ini
# Application Core Settings
APP_NAME="Incident Postmortem Tracker"
DEBUG=True
LOG_LEVEL="INFO"

# Server Execution
HOST="127.0.0.1"
PORT=8000

# Database Configuration
DATABASE_URL="sqlite:///./incidents.db"
```

---

## Usage Guide

### Starting the Server

Run the application using the entrypoint script:

```bash
python -m src.main
```

Alternatively, start it via `uvicorn` directly:

```bash
uvicorn src.main:app --host 127.0.0.1 --port 8000 --reload
```

Once running, access the web dashboard at `http://127.0.0.1:8000/` and interactive API documentation at `http://127.0.0.1:8000/docs`.

---

### REST API Endpoints & Examples

#### 1. Ingest Automated Monitoring Webhook
Automatically create an open incident from an incoming monitoring system alert (e.g., Alertmanager/Datadog):

```bash
curl -X POST "http://127.0.0.1:8000/api/v1/webhooks/alert" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "High Latency on Payment Gateway",
    "severity": "SEV-1",
    "source": "Prometheus/Alertmanager",
    "description": "99th percentile response time exceeded 2000ms over 5m window."
  }'
```

#### 2. Log a Timeline Event
Append a critical action or milestone to an existing incident's chronological timeline:

```bash
curl -X POST "http://127.0.0.1:8000/api/v1/incidents/1/timeline" \
  -H "Content-Type: application/json" \
  -d '{
    "timestamp": "2023-10-24T14:32:00Z",
    "event_type": "MITIGATION",
    "description": "Traffic rerouted to secondary region US-East-2.",
    "actor": "sre-oncall@example.com"
  }'
```

#### 3. Create a Remediation Action Item
Assign a follow-up engineering task directly tied to the postmortem:

```bash
curl -X POST "http://127.0.0.1:8000/api/v1/incidents/1/action-items" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Increase connection pool size for DB replica",
    "assignee": "alex.dev@example.com",
    "priority": "HIGH",
    "status": "OPEN"
  }'
```

#### 4. Fetch Incidents List
Get a summary list of all tracked operational incidents:

```bash
curl -X GET "http://127.0.0.1:8000/api/v1/incidents"
```

---

## Automated Testing Instructions

The project uses `pytest` for unit and integration testing covering database access, core latency calculations, and API routes.

### Run All Tests

```bash
pytest
```

### Verbose Mode with Test Coverage Report

```bash
pytest -v --cov=src
```

### Running Specific Test Modules

```bash
# Run database and core business logic tests
pytest tests/test_core.py

# Run API endpoint and HTTP handler tests
pytest tests/test_api.py
```

---

## Project Structure

```
incident-postmortem-tracker/
│
├── .env.example          # Sample environment variables configuration
├── .gitignore            # Git exclusion rules
├── requirements.txt      # Python dependencies
│
├── src/
│   ├── __init__.py       # Package marker
│   ├── models.py         # SQLAlchemy ORM models & Pydantic validation schemas
│   ├── core.py           # Database engine management, sessions, logic & metrics
│   ├── api.py            # FastAPI REST routing, webhooks & HTMX views
│   └── main.py           # Application entrypoint & Uvicorn launcher
│
└── tests/
    ├── __init__.py       # Test package marker
    ├── test_core.py      # Tests for core calculations, models, and DB operations
    └── test_api.py       # Integration tests for FastAPI endpoints and webhooks
```

---

## Roadmap & Future Enhancements

- [ ] **Third-Party Integrations:** Native bi-directional sync with Slack, PagerDuty, Opsgenie, and Jira.
- [ ] **AI-Assisted Timeline Summarization:** Automatic postmortem draft generation based on raw chat logs and alert streams.
- [ ] **PDF & Markdown Exports:** Single-click export of complete postmortems for executive reporting or customer distribution.
- [ ] **Reliability Analytics Dashboard:** Visual trends for MTTD/MTTR grouped by service, team, and severity level.

---

## License

Distributed under the MIT License. See `LICENSE` for more information.