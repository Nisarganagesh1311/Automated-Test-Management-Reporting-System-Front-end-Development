# TestFlow — Automated Test Management & Reporting System

A full-stack platform to upload, analyze, and report test results from **JUnit**, **PyTest**, and **Playwright** with AI-powered failure analysis, Slack/Teams notifications, and real-time dashboards.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        TestFlow                             │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │  Next.js 14  │───▶│  FastAPI     │───▶│  PostgreSQL  │  │
│  │  (Frontend)  │    │  (Backend)   │    │  (Database)  │  │
│  │  Port 3000   │    │  Port 8000   │    │  Port 5432   │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│         │                   │                               │
│         │            ┌──────┴──────────────────┐           │
│         │            │  Services                │           │
│         │            │  • JUnit XML Parser      │           │
│         │            │  • PyTest JSON Parser    │           │
│         │            │  • Playwright Parser     │           │
│         │            │  • AI Failure Summary    │           │
│         │            │  • Slack/Teams Notifs    │           │
│         │            └─────────────────────────-┘           │
└─────────────────────────────────────────────────────────────┘
```

## Tech Stack

| Layer         | Technology                          |
|---------------|-------------------------------------|
| Frontend      | Next.js 14, TypeScript, Tailwind CSS |
| Charts        | Recharts (AreaChart, PieChart, BarChart) |
| Backend       | FastAPI (Python 3.11)               |
| Database      | PostgreSQL 16 + SQLAlchemy (async)  |
| AI            | OpenAI GPT-3.5 (with rule-based fallback) |
| Notifications | Slack Webhooks, MS Teams Webhooks   |
| CI/CD         | GitHub Actions                      |
| Container     | Docker + Docker Compose             |

---

## Features

- **Upload test results** from JUnit XML, PyTest JSON, Playwright JSON
- **Automated dashboards** — pass rate, failure trends, run history
- **Failure trend analysis** — spot regressions over time
- **Flaky test detection** — identify tests that fail intermittently
- **AI-generated summaries** — GPT-powered root cause hypotheses
- **Slack / Teams notifications** — instant alerts on test run completion
- **CI/CD integration** — one `curl` command in GitHub Actions
- **Interactive test case viewer** — expandable stack traces per test

---

## Quick Start (Docker)

```bash
# 1. Clone and enter the directory
cd "FULL STACK"

# 2. Copy and fill in secrets (optional)
cp .env.example .env

# 3. Start all services
docker compose up --build

# 4. Open the app
open http://localhost:3000
# API docs: http://localhost:8000/docs
```

---

## Local Development

### Backend

```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure environment
cp .env.example .env
# Edit .env with your DATABASE_URL etc.

# Start PostgreSQL (or use Docker)
docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=password \
  -e POSTGRES_DB=testmanager postgres:16-alpine

# Run the API
uvicorn app.main:app --reload
# → http://localhost:8000
# → Swagger UI: http://localhost:8000/docs
```

### Frontend

```bash
cd frontend

# Install dependencies
npm install

# Configure environment
echo "NEXT_PUBLIC_API_URL=http://localhost:8000" > .env.local

# Start dev server
npm run dev
# → http://localhost:3000
```

---

## Uploading Test Results

### Via UI
1. Click **Upload Results** in the sidebar
2. Select your framework (JUnit / PyTest / Playwright)
3. Drag & drop your report file
4. Fill in optional metadata (project, branch, commit)
5. Enable Slack/Teams notification if needed
6. Click **Upload & Analyze**

### Via cURL (CI/CD)

```bash
# JUnit XML
curl -X POST http://localhost:8000/api/upload \
  -F "file=@target/surefire-reports/TEST-all.xml" \
  -F "source_type=junit" \
  -F "name=Nightly Build #42" \
  -F "branch=main" \
  -F "project=my-service"

# PyTest (install pytest-json-report first)
pytest --json-report --json-report-file=results.json
curl -X POST http://localhost:8000/api/upload \
  -F "file=@results.json" \
  -F "source_type=pytest"

# Playwright (add JSON reporter to playwright.config.ts)
curl -X POST http://localhost:8000/api/upload \
  -F "file=@playwright-report.json" \
  -F "source_type=playwright"
```

---

## API Reference

| Method | Endpoint                        | Description                        |
|--------|---------------------------------|------------------------------------|
| POST   | `/api/upload`                   | Upload & parse test results        |
| GET    | `/api/test-runs`                | List test runs (paginated)         |
| GET    | `/api/test-runs/{id}`           | Get run details with test cases    |
| GET    | `/api/test-runs/{id}/cases`     | Get test cases for a run           |
| DELETE | `/api/test-runs/{id}`           | Delete a test run                  |
| GET    | `/api/analytics/summary`        | Overall statistics                 |
| GET    | `/api/analytics/trends`         | Daily pass/fail trend data         |
| GET    | `/api/analytics/flaky`          | Flaky test report                  |
| GET    | `/api/analytics/projects`       | List of all projects               |
| POST   | `/api/notifications/test`       | Send a test Slack/Teams message    |

Full interactive docs: `http://localhost:8000/docs`

---

## Sample Files

The `samples/` directory contains example test reports for each format:

| File                        | Format          |
|-----------------------------|-----------------|
| `samples/junit-sample.xml`  | JUnit XML       |
| `samples/pytest-sample.json`| PyTest JSON     |
| `samples/playwright-sample.json` | Playwright JSON |

---

## CI/CD Integration

Add to your `.github/workflows/*.yml`:

```yaml
- name: Upload Test Results to TestFlow
  if: always()
  run: |
    curl -X POST "${{ vars.TESTFLOW_API_URL }}/api/upload" \
      -F "file=@test-results.xml" \
      -F "source_type=junit" \
      -F "name=${{ github.workflow }} #${{ github.run_number }}" \
      -F "branch=${{ github.ref_name }}" \
      -F "commit_hash=${{ github.sha }}" \
      -F "project=${{ github.repository }}" \
      -F "notify_slack=true"
```

Set `TESTFLOW_API_URL` as a repository variable in GitHub Settings → Variables.

---

## Project Structure

```
FULL STACK/
├── backend/
│   ├── app/
│   │   ├── main.py               # FastAPI app entry point
│   │   ├── config.py             # Settings / env vars
│   │   ├── database.py           # Async SQLAlchemy setup
│   │   ├── models/               # SQLAlchemy ORM models
│   │   │   ├── test_run.py
│   │   │   └── test_case.py
│   │   ├── schemas/              # Pydantic schemas
│   │   ├── routes/               # API route handlers
│   │   │   ├── upload.py
│   │   │   ├── test_runs.py
│   │   │   ├── analytics.py
│   │   │   └── notifications.py
│   │   └── services/
│   │       ├── parsers/          # JUnit / PyTest / Playwright parsers
│   │       ├── ai_service.py     # OpenAI failure summarization
│   │       └── notification_service.py
│   ├── tests/                    # Backend unit tests
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── app/                  # Next.js App Router pages
│   │   │   ├── page.tsx          # Dashboard
│   │   │   ├── upload/
│   │   │   ├── test-runs/
│   │   │   ├── analytics/
│   │   │   └── settings/
│   │   ├── components/
│   │   │   ├── layout/           # Sidebar, Header
│   │   │   ├── dashboard/        # StatsCard, RecentRuns
│   │   │   ├── charts/           # Recharts wrappers
│   │   │   ├── upload/           # UploadDropzone
│   │   │   └── test-runs/        # TestRunCard, TestCaseTable
│   │   ├── lib/api.ts            # Axios API client
│   │   └── types/index.ts        # TypeScript interfaces
│   ├── package.json
│   └── Dockerfile
├── samples/                      # Example test reports
├── .github/workflows/ci.yml      # GitHub Actions CI pipeline
├── docker-compose.yml
└── README.md
```

---

## Environment Variables

| Variable             | Required | Description                        |
|----------------------|----------|------------------------------------|
| `DATABASE_URL`       | Yes      | PostgreSQL async connection URL     |
| `OPENAI_API_KEY`     | No       | For AI failure summaries (optional) |
| `SLACK_WEBHOOK_URL`  | No       | Incoming Webhook URL from Slack     |
| `TEAMS_WEBHOOK_URL`  | No       | Incoming Webhook from Teams         |
| `CORS_ORIGINS`       | No       | JSON array of allowed origins       |






#refrence images
Dash board
<img width="1901" height="924" alt="Screenshot 2026-04-02 121217" src="https://github.com/user-attachments/assets/946889a2-cd02-4721-968e-d64843a6b04b" />

<img width="1907" height="873" alt="Screenshot 2026-04-02 121937" src="https://github.com/user-attachments/assets/a6bf6d48-54fc-49aa-b793-4b9ef4941846" />
<img width="1901" height="905" alt="Screenshot 2026-04-02 121907" src="https://github.com/user-attachments/assets/d9ec634d-e6a1-47eb-8819-de313b1497f8" />
<img width="1883" height="862" alt="Screenshot 2026-04-02 121922" src="https://github.com/user-attachments/assets/f625e20a-08ed-4104-898b-5dafbab810c7" />
<img width="1016" height="842" alt="Screenshot 2026-04-02 121841" src="https://github.com/user-attachments/assets/6ec8b77c-2f0d-4fcf-b3d9-ad647bcdb1cc" />






