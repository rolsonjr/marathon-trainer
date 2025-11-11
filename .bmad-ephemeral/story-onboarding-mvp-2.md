# Story 1.2: Strava OAuth Link & Activity Sync Service

**Status:** Draft

---

## User Story

As a runner who tracks workouts on Strava,
I want to link my Strava account during onboarding and have Coach George ingest my recent runs,
So that the generated plan reflects my actual training load.

---

## Acceptance Criteria

**Given** the runner reaches the "Connect Strava" step,
**When** they tap "Link Strava",
**Then** an `ASWebAuthenticationSession` opens Strava OAuth, completes PKCE flow, and returns to the app with success/error messaging.

**And** when OAuth succeeds,
**Then** backend `/api/v1/strava/callback` stores encrypted access/refresh tokens, athlete_id, and responds with link status to the client.

**And** when the Strava sync worker runs (manual trigger or hourly schedule),
**Then** it fetches the last 28 days of activities for the athlete, persists workouts, and records ingestion latency in telemetry.

**And** when the user visits the profile summary,
**Then** they see Strava connection status + last sync timestamp (or actionable error state if sync fails).

---

## Implementation Details

### Tasks / Subtasks

- Implement Strava OAuth UI with PKCE and deep-link callback `coachai://oauth/strava`.
- Add backend endpoints `/api/v1/strava/link-token`, `/api/v1/strava/callback`, `/api/v1/strava/status`.
- Build `StravaClient` (HTTPX) with refresh token support, error handling, and rate limit logging.
- Create Celery worker to poll `/athlete/activities` hourly and store workouts in Postgres.
- Expose link status to mobile onboarding summary.

### Technical Summary

OAuth 2.0 integration, secure token storage, Celery/Redis worker for ingestion, telemetry logging per ingestion run.

### Project Structure Notes

- **Files to modify:** `mobile/ios/Features/Onboarding/ConnectStravaStep.swift`, `mobile/ios/Services/StravaLinkService.swift`, `services/orchestrator/app/strava/{oauth.py,client.py,sync.py}`, `services/orchestrator/app/routes/strava.py`, Celery config.
- **Expected test locations:** `mobile/ios/Tests/StravaLinkTests.swift`, `services/orchestrator/tests/test_strava_oauth.py`, `services/orchestrator/tests/test_activity_ingestion.py`.
- **Estimated effort:** 5 story points (2 days).
- **Prerequisites:** Story 1.1 completed (profile submission is required before linking Strava).

### Key Code References

N/A – follow tech-spec Strava integration details.

---

## Context References

**Tech-Spec:** [tech-spec.md](../docs/tech-spec.md) – Strava OAuth steps, token schema, ingestion worker, telemetry expectations.

**Architecture:** Epic 6 – Strava-Powered Onboarding MVP in `docs/epics.md`.

---

## Dev Agent Record

### Agent Model Used

<!-- Will be populated during dev-story execution -->

### Debug Log References

<!-- Will be populated during dev-story execution -->

### Completion Notes

<!-- Will be populated during dev-story execution -->

### Files Modified

<!-- Will be populated during dev-story execution -->

### Test Results

<!-- Will be populated during dev-story execution -->

---

## Review Notes

<!-- Will be populated during code review -->
