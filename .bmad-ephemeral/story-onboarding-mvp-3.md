# Story 1.3: Weekly Plan Composer & Plan Carousel Delivery

**Status:** Draft

---

## User Story

As a newly onboarded runner,
I want Coach George to generate and display a rationale-rich 7-day plan based on my Strava history and goals,
So that I can start training immediately with confidence.

---

## Acceptance Criteria

**Given** a runner has completed onboarding and Strava linking (stories 1.1 & 1.2),
**When** the backend plan composer runs,
**Then** it ingests runner profile + last 4 weeks of workouts, classifies the training phase, and stores a 7-day plan with rationale tags in the `plans` table.

**And** when the client requests `/api/v1/plans/{plan_id}`,
**Then** it receives the latest plan payload, including workout type, duration/mileage, rationale chips (countdown, mastery delta, availability), and last sync timestamp.

**And** when the user views the plan carousel in-app,
**Then** they see day-by-day cards, can accept the plan (triggering background sync + offline cache), and offline mode displays the cached plan read-only with banner messaging.

---

## Implementation Details

### Tasks / Subtasks

- Implement `PlanComposer` LangGraph node + heuristic templates (base/build/peak) using data from Strava ingestion.
- Add FastAPI routes `/api/v1/plans/{plan_id}` and `/api/v1/plans/{plan_id}/accept` with validation + telemetry logging.
- Build SwiftUI `PlanCarouselView` + `PlanCardView` with rationale chips and quick actions (accept, refresh).
- Implement offline caching + banner messaging using persisted plan JSON.
- Add telemetry hooks for plan generation latency and acceptance events.

### Technical Summary

Rule-based plan generation, LangGraph orchestration, REST delivery endpoints, SwiftUI presentation + offline cache synchronization.

### Project Structure Notes

- **Files to modify:** `services/orchestrator/app/planner/weekly_plan.py`, `services/orchestrator/app/routes/plans.py`, `mobile/ios/Features/PlanOverview/*`, `mobile/ios/Services/PlanService.swift`.
- **Expected test locations:** `services/orchestrator/tests/test_plan_composer.py`, `services/orchestrator/tests/test_plans_api.py`, `mobile/ios/Tests/PlanCarouselSnapshotTests.swift`.
- **Estimated effort:** 6 story points (3 days).
- **Prerequisites:** Stories 1.1 and 1.2.

### Key Code References

`docs/tech-spec.md` – Implementation steps for plan composer, data models, telemetry requirements.

---

## Context References

**Tech-Spec:** [tech-spec.md](../docs/tech-spec.md) – Weekly plan heuristics, API contracts, offline cache behavior, acceptance criteria.

**Architecture:** Epic 6, Strava-Powered Onboarding MVP.

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
