# marathon-trainer - Technical Specification

**Author:** Rodney  
**Date:** 2025-11-11  
**Project Level:** 1  
**Change Type:** New feature (onboarding + Strava-powered weekly plan)  
**Development Context:** Greenfield mobile + orchestration stack

---

## Context

### Available Documents

- `docs/PRD.md` – full product requirements, success criteria, domain constraints.  
- `docs/bmm-product-brief-coach.ai-2025-11-11.md` – problem statement and MVP scope.  
- `docs/epics.md` – cross-portfolio epics and functional requirements (runner intake, analytics, adaptive planning, PT safeguards).

### Project Stack

- **Mobile:** Swift 5.10, SwiftUI 5, Combine, BackgroundTasks, AuthenticationServices (for OAuth).  
- **Backend:** Python 3.11, FastAPI, LangGraph/LangChain 0.2, Postgres 15, Redis (queue/cache).  
- **Integrations:** Strava API v3 (OAuth 2.0, Activities endpoint, Webhooks), Clerk.dev (optional auth) for later phases.  
- **Infra:** AWS (Lambda or Fargate) or Render for LangGraph service, S3 for static assets, GitHub Actions for CI/CD.

### Existing Codebase Structure

Greenfield – no existing repositories. This spec establishes the directory layout:

```
/mobile/ios/
  MarathonTrainerApp.swift
  Features/Onboarding/
  Features/PlanOverview/
/services/orchestrator/
  app/main.py
  app/routes/
  app/strava/
  app/planner/
/infrastructure/
  terraform/
  github-actions/
```

---

## The Change

### Problem Statement

Coach.ai currently lacks an entry point for runners. We need an end-to-end onboarding experience that collects goal data, connects to Strava (instead of Garmin), ingests the runner’s recent workouts, and produces an actionable 7-day training plan tailored to their upcoming race so the MVP can demonstrate adaptive coaching value.

### Proposed Solution

1. **Conversational Onboarding (iOS):** Guided SwiftUI flow that captures race details, availability, injury history, and collects consent.  
2. **Strava OAuth & Data Sync:** Replace Garmin integrations with Strava OAuth 2.0. The iOS app initiates login, backend stores tokens, and scheduled jobs ingest activities.  
3. **Weekly Plan Service:** LangGraph node consumes onboarding inputs + recent Strava runs to generate a 7-day plan (baseline heuristics + mastery placeholders).  
4. **Plan Presentation:** iOS displays the generated plan in a chat-style carousel with rationale tags from the backend.  
5. **Telemetry & Logging:** Capture ingestion latency and plan generation timestamps for demo readiness metrics.

### Scope

**In Scope**

- iOS onboarding wizard (4 screens + confirmation).  
- Strava OAuth handshake, token storage, and activity ingestion job (polling 1×/hr).  
- Baseline weekly plan generator (easy/tempo/long/taper heuristics).  
- Plan delivery endpoint + SwiftUI plan carousel.  
- Consent + privacy copy for Strava data, data export stub.  
- CI/CD bootstrap and environment configs for these services.

**Out of Scope**

- Android or web clients.  
- Advanced mastery scoring & PT agent automation (future epics).  
- Push notifications, reminders, or schedule conflict intents.  
- Production-grade alerting/monitoring (basic logging only).  
- Multi-race scheduling or multi-athlete support.

---

## Implementation Details

### Source Tree Changes

- `mobile/ios/Features/Onboarding/*` – Views, view models, validation helpers.  
- `mobile/ios/Features/PlanOverview/*` – Plan carousel + rationale badges.  
- `services/orchestrator/app/routes/onboarding.py` – REST endpoints for profile creation and plan retrieval.  
- `services/orchestrator/app/strava/oauth.py` & `sync.py` – OAuth handlers, token store, activity ingestion worker.  
- `services/orchestrator/app/planner/weekly_plan.py` – Rule-based plan composer + rationale tags.  
- `infrastructure/github-actions/ios-ci.yml` & `infra/github-actions/backend-ci.yml` – build/test pipelines.  
- `infrastructure/terraform/strava-webhook.tf` (placeholder) – Strava subscription endpoint wiring.

### Technical Approach

1. **Onboarding Flow**  
   - SwiftUI `OnboardingFlowView` orchestrates steps: Welcome → Race Details → Availability & Injuries → Connect Strava → Review & Submit.  
   - Local `RunnerProfileDraft` stored via `@StateObject` and persisted in `Keychain` until backend confirmation.  
   - Submit payload to `/api/v1/onboarding` (JSON) and await plan ID.

2. **Strava OAuth Integration**  
   - Use `ASWebAuthenticationSession` with Strava authorize URL.  
   - Backend `/api/v1/strava/callback` exchanges code for tokens, persists `athlete_id`, `access_token`, `refresh_token`, `expires_at`.  
   - Provide `/api/v1/strava/link-token` so mobile obtains client_id/scope/state securely.  
   - Poll activities endpoint (`/athlete/activities?after=`) hourly via Celery/Redis worker; store workouts in `workouts` table.

3. **Weekly Plan Generation**  
   - LangGraph node `PlanComposer` takes onboarding JSON + last 4 weeks of Strava metrics.  
   - Heuristic: compute average weekly mileage, categorize runner as Base / Build / Peak, and map to template (Mon rest, Tue tempo, Wed easy, Thu workout, Fri rest, Sat long run, Sun recovery).  
   - Tag each workout with rationale (e.g., `countdown=12`, `mastery-speed=-2`).  
   - Persist plan document + revision to `plans` table and send to client.

4. **Plan Presentation**  
   - SwiftUI `PlanCarouselView` renders cards per day with quick actions (accept, swap).  
   - Pulls `GET /api/v1/plans/{plan_id}`; caches plan for offline read-only access.  
   - Displays Strava connection status and last sync timestamp.

5. **Telemetry**  
   - Log ingestion latency, plan generation time, and Strava sync failures to CloudWatch or OpenTelemetry file sink for MVP demos.

### Existing Patterns to Follow

- Follow LangGraph node conventions from prior prototypes (context store + tool definitions).  
- Use `PlanOutput` schema from PRD’s functional requirements for plan cards.  
- iOS uses MVVM (View + ViewModel + Service) with Combine for network state.

### Integration Points

- **Strava API:** OAuth endpoints, activities endpoint, optional webhook subscription.  
- **LangGraph Planner:** New node integrates with existing orchestrator runtime.  
- **Postgres:** Tables `runner_profiles`, `strava_tokens`, `workouts`, `plans`.  
- **Redis/Celery:** Jobs for activity polling and plan generation queue.

---

## Development Context

### Relevant Existing Code

None – greenfield. Patterns are defined in this spec and reused from PRD epics.

### Dependencies

**Framework/Libraries**

- SwiftUI, Combine, Alamofire (network), AuthenticationServices.  
- FastAPI 0.111, LangChain 0.2, HTTPX, SQLAlchemy, Celery 5.3, Redis 7, psycopg2.  
- Strava API SDK (community) or custom client.

**Internal Modules**

- `app/core/config.py` for settings.  
- `app/common/clients/strava_client.py`.  
- `app/planner/templates/*.yaml` (plan archetypes).

### Configuration Changes

- `.env` additions: `STRAVA_CLIENT_ID`, `STRAVA_CLIENT_SECRET`, `STRAVA_WEBHOOK_VERIFY_TOKEN`, `PLAN_QUEUE_URL`.  
- iOS `Info.plist`: add `ASWebAuthenticationSession` callback URL (`coachai://oauth/strava`).  
- Terraform/CloudFormation resources for webhook endpoint + Secrets Manager values.

### Existing Conventions (Brownfield)

N/A – define conventions: snake_case APIs, JSON:API style responses, UTC timestamps, ISO date strings, BDD-style tests.

### Test Framework & Standards

- iOS: XCTest + SnapshotTesting for SwiftUI flows.  
- Backend: Pytest with HTTPX test client, integration tests hitting Strava sandbox mocks.  
- Contract tests: Postman/Newman or schemathesis for REST endpoints.

---

## Implementation Stack

- **Client:** SwiftUI, Combine, async/await networking.  
- **Server:** FastAPI + LangGraph orchestrator, Postgres persistence, Redis queues.  
- **Infra:** Dockerized services, GitHub Actions for CI, Render/Heroku for staging, AWS/GCP for prod later.

---

## Technical Details

- **Data Models:**  
  - `RunnerProfile(id, name, race_date, race_type, goal_time, availability, injury_flags, strava_athlete_id)`  
  - `StravaToken(profile_id, access_token, refresh_token, expires_at, scope)`  
  - `Workout(profile_id, activity_id, start_time, distance_m, duration_s, avg_hr, type)`  
  - `Plan(plan_id, profile_id, week_start_date, status, payload_json, rationale_json)`

- **Security:** PKCE for Strava OAuth, encrypt tokens with KMS/Keychain before storage.  
- **Error Handling:** Gracefully degrade when Strava sync fails (display pending badge).  
- **Offline Cache:** Store latest plan JSON in `FileManager` + CoreData; mark as read-only offline.

---

## Development Setup

1. Clone mono-repo (`mobile`, `services`, `infrastructure`).  
2. Install toolchains: Xcode 16 beta, Python 3.11 (pyenv), Poetry.  
3. `make bootstrap-ios` – installs SwiftLint, sets bundle IDs.  
4. `make bootstrap-backend` – creates virtualenv, installs dependencies, seeds Postgres/Redis via Docker Compose.  
5. Configure `.env` with Strava sandbox credentials.  
6. Run `docker compose up` to start backend stack, `make run-ios` to launch simulator.

---

## Implementation Guide

### Setup Steps

1. Provision Strava API app, capture client credentials.  
2. Configure GitHub Actions secrets and Terraform variables.  
3. Create database schema migrations for new tables.  
4. Wire LangGraph runtime with new nodes (PlanComposer, StravaSync).

### Implementation Steps

1. **Onboarding UI & API Contract (2 days)**  
   - Build SwiftUI wizard, local validation, `RunnerProfileService`.  
   - Define `/api/v1/onboarding` schema + FastAPI route; persist profile + initial plan job.
2. **Strava OAuth + Activity Sync (2 days)**  
   - Implement mobile OAuth session + backend callback.  
   - Store tokens, implement refresh + Celery worker to pull last 28 days of workouts.  
   - Provide `/api/v1/strava/status` for client.
3. **Weekly Plan Composer & Delivery (2 days)**  
   - Implement plan templates, heuristics, LangGraph node.  
   - Build `/api/v1/plans/{id}` endpoint, plan carousel UI, offline cache.  
   - Log telemetry + acceptance tests.

### Testing Strategy

- Unit tests for validation, plan heuristics, Strava client.  
- Integration tests simulating OAuth callback + activity ingestion.  
- UI snapshot tests for each onboarding screen and plan cards.  
- Manual end-to-end run using Strava sandbox athlete to ensure plan generated within 5 minutes.

### Acceptance Criteria

1. Runner completes onboarding, links Strava, and receives a 7-day plan within 5 minutes.  
2. All plan cards display rationale badges (countdown, intensity, availability).  
3. Strava sync status + last sync timestamp visible in-app.  
4. Offline mode shows cached plan read-only with banner messaging.  
5. Telemetry captures ingestion + plan generation timestamps for demo dashboards.

---

## Developer Resources

### File Paths Reference

- `mobile/ios/Features/Onboarding/*.swift`  
- `mobile/ios/Features/PlanOverview/*.swift`  
- `services/orchestrator/app/routes/onboarding.py`  
- `services/orchestrator/app/routes/plans.py`  
- `services/orchestrator/app/strava/{oauth.py,sync.py,client.py}`  
- `services/orchestrator/app/planner/weekly_plan.py`

### Key Code Locations

- SwiftUI view models for state handling: `OnboardingViewModel.swift`.  
- `RunnerProfileRepository.swift` for persistence + API calls.  
- `PlanCarouselView.swift` for UI.  
- `app/planner/templates/*.yaml` for workout patterns.  
- `app/strava/client.py` for API wrapper.

### Testing Locations

- `mobile/ios/Tests/OnboardingTests.swift`  
- `mobile/ios/Tests/PlanCarouselSnapshotTests.swift`  
- `services/orchestrator/tests/test_onboarding_api.py`  
- `services/orchestrator/tests/test_plan_composer.py`

### Documentation to Update

- `README.md` – add onboarding instructions + Strava setup notes.  
- `docs/PRD.md` – mark Strava pivot + onboarding progress (optional).  
- `docs/epics.md` – link Level 1 epic/stories generated below.

---

## UX/UI Considerations

- Progress indicator (step dots) + friendly microcopy in onboarding.  
- Show Strava branding guidelines during OAuth step.  
- Plan cards highlight workout type, duration, rationale chip, and quick actions.  
- Offline banner uses high-contrast color and is dismissible after user acknowledges.

---

## Testing Approach

- Automated: Unit + integration tests described above.  
- Manual: Run E2E on simulator and physical device with Strava sandbox account.  
- Regression: Ensure plan retrieval gracefully handles empty Strava history (fallback template).  
- Accessibility: VoiceOver labels for onboarding steps, dynamic type support, haptic feedback on plan acceptance.

---

## Deployment Strategy

### Deployment Steps

1. Merge PRs to `main`; GitHub Actions runs unit/integration suites.  
2. Backend: Docker image pushed, deployed to staging (Render/AWS).  
3. Run smoke tests (create profile, ingest fake Strava data, fetch plan).  
4. iOS: Archive build, distribute via TestFlight to pilot testers.

### Rollback Plan

- Backend: Deploy previous container image (GitHub Actions retains last 5).  
- iOS: Keep prior TestFlight build as fallback; instruct testers to revert if critical issues.  
- Data: Keep DB migration down scripts to drop new tables if needed (not recommended once real runners onboard).

### Monitoring

- CloudWatch/OpenTelemetry logs for Strava sync + plan generation timing.  
- Temporary dashboard (Grafana or Metabase) showing ingestion SLA.  
- Sentry/Crashlytics for client error tracking.

---

## Developer Resources Summary

- Tech-spec (this document) is primary context for onboarding MVP.  
- Use LangGraph nodes and Strava client defined herein as canonical implementations.  
- Stories generated below decompose the work into implementable slices.

