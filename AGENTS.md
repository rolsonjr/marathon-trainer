# Repository Guidelines

## Project Structure & Module Organization
SwiftUI lives in `mobile/ios/`, organized by feature folders (`Features/Onboarding`, `Features/PlanOverview`, `Features/StravaLinking`) with shared state kept in `RunnerProfileDraft`. The FastAPI + LangGraph backend sits in `services/orchestrator/`: `app/routes/` exposes onboarding + plan APIs, `app/strava/` manages OAuth and sync, and `app/planner/weekly_plan.py` emits rationale-rich workouts while Celery workers handle ingestion. CI workflows and Terraform modules stay under `infrastructure/`, and repo-root resources (`docker-compose.yml`, `.env.example`, docs) wire local Postgres/Redis.

## Build, Test, and Development Commands
- `make bootstrap-ios` – install Swift toolchains, lint/formatters, and resolve SPM deps.
- `make bootstrap-backend` – create the Python venv, install FastAPI/Celery, seed `.env`.
- `docker compose up` – run Postgres, Redis, orchestrator API, and Strava webhook tunnel.
- `xcodebuild -scheme MarathonTrainer test` – run unit + snapshot suites headlessly.
- `pytest services/orchestrator -q` – cover REST routes, planner logic, ingestion workers.
- `make lint` – aggregate SwiftLint, SwiftFormat, Ruff, and Black before commits.

## Coding Style & Naming Conventions
Swift uses 2-space indentation, UpperCamelCase types, lowerCamelCase members, and protocol-based ViewModels per feature. Extract subviews as files approach 200 lines and prefer async/await for networking while honoring existing Combine pipelines. Python follows PEP 8 with Black (88 cols) and Ruff; keep snake_case functions, CapWords classes, and type hints on planner nodes. Name directories by capability (`planner`, `strava`, `features`) and choose action-oriented filenames (`WeeklyPlanComposer.swift`, `weekly_plan.py`).

## Testing Guidelines
`mobile/ios/Tests/` hosts XCTest and SnapshotTesting with baselines stored in `__Snapshots__`; mirror screen names (e.g., `PlanOverviewViewTests`). Backend tests live in `services/orchestrator/tests/`, using Pytest + HTTPX with live Strava hits tagged `@pytest.mark.integration` and fixtures for Redis/Postgres. Maintain ≥80% coverage for planner modules, add a regression whenever ingestion logic changes, and document manual Strava sandbox runs inside the PR.

## Commit & Pull Request Guidelines
Commits stay short, imperative, and scoped (`Add PT guardrail banner`), with WIP noise squashed before pushing. PRs explain intent, link tickets, and call out migrations or Terraform edits. UI changes need screenshots or VoiceOver notes; backend work should attach sample JSON payloads and list the commands run (`make lint`, `pytest`, `xcodebuild … test`, `docker compose up`). Mention any telemetry or sandbox data touched.

## Security & Configuration Tips
Never commit `.env`, Strava tokens, or Keychain exports—use `.env.example`, Vault/KMS, and Keychain. Enforce PKCE, redact PII in logs, and keep the “not a doctor” disclaimer in runner-facing screens. Cached plans in `Application Support/Plans` stay readable offline with stale-data banners when telemetry lags.
