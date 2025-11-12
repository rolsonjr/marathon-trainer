# AGENTS GUIDE

## Project Overview

- AI marathon coach delivering adaptive 7-day training plans via SwiftUI mobile client plus orchestration backend. Runners supply race date, goal time, schedule constraints, and injury history so plans adjust with empathy and safety.
- Multi-agent narrative: Coach George handles dialogue, analytics agent scores workout mastery and load deltas, PT agent enforces injury guardrails. Ensemble must react within five minutes when new Strava data or runner inputs arrive.
- Core loop: onboarding → consent + Strava OAuth → Strava activity ingestion + telemetry → LangGraph planner composes rationale-rich plan → SwiftUI plan carousel & chat expose updates, even offline.
- Success signals to preserve: 90% of workouts re-evaluated within SLA, rationale chips highlight at least four personalization factors, PT responses always accompany injury mentions, and cached plans remain readable offline with banners.
- Primary scope is iOS + Python backend. Android/web, advanced mastery modeling, nutrition, and push notifications remain future work. Keep privacy/consent copy explicit (“not a doctor”) in every runner-facing flow.

## Architecture

- **Client (SwiftUI 5.10):** `mobile/ios/` hosts onboarding wizard (`Features/Onboarding`), plan carousel (`Features/PlanOverview`), `RunnerProfileDraft` state, and Strava linking UI (`ASWebAuthenticationSession`). Uses Combine + async/await networking, BackgroundTasks for sync retries, Keychain/FileManager for local drafts + cached plan JSON, VoiceOver-friendly components, and Snapshot/XCTest coverage.
- **Backend (FastAPI + LangGraph):** `services/orchestrator/` contains REST routes (`app/routes/{onboarding,plans}.py`), Strava OAuth handlers (`app/strava/{oauth.py,sync.py,client.py}`), Celery/Redis-powered ingestion worker, and weekly plan composer (`app/planner/weekly_plan.py` with heuristics templates). Planner nodes output workout payloads plus rationale tags consumed by the client. Error handling degrades gracefully when Strava data lags.
- **Data & Integrations:** Postgres 15 stores `RunnerProfile`, `StravaToken`, `Workout`, and `Plan` tables. Redis queues ingestion jobs and cache hints. Strava API v3 provides OAuth, webhook/polling ingestion (hourly), and activities fetch. Tokens encrypted via KMS/Keychain, PKCE enforced. Telemetry logs ingestion latency + plan generation timestamps for demo dashboards.
- **Infrastructure & CI/CD:** GitHub Actions workflows under `infrastructure/github-actions/` run lint/tests for iOS + backend and publish Docker images. Terraform (e.g., `infrastructure/terraform/strava-webhook.tf`) provisions webhook endpoints, secrets, and future Render/AWS targets. Local dev uses `make bootstrap-ios`, `make bootstrap-backend`, Docker Compose for Postgres/Redis, `.env` with Strava sandbox keys, and `docker compose up` to run services.
- **Testing & Quality:** iOS relies on XCTest + SnapshotTesting for onboarding and plan UI; backend uses Pytest + HTTPX test client; contract tests via Postman/Newman or schemathesis; manual E2E run with Strava sandbox athlete ensures plan delivery <5 minutes. Acceptance focuses on onboarding completion, plan rationale visibility, Strava status surfacing, and offline read-only mode.
- **Workstream Alignment:** Epics emphasize foundation scaffolding, runner intake, analytics mastery pipeline, adaptive planning/chat, and PT safety + demo assets. When prioritizing tasks, preserve ordering dependencies: scaffolding → onboarding/data intake → analytics → adaptive planner → PT safeguards.
