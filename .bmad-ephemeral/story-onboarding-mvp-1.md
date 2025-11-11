# Story 1.1: Guided Onboarding Flow & Profile Submission

**Status:** Draft

---

## User Story

As a runner preparing for a race,
I want a conversational onboarding flow that captures my goals, schedule, and consent,
So that Coach George can personalize my plan from the first interaction.

---

## Acceptance Criteria

**Given** a first-time user launches the iOS app,
**When** they progress through the onboarding wizard (Welcome → Race Details → Availability/Injuries → Connect Strava → Review),
**Then** all required fields are validated inline and stored in the local draft profile.

**And** when the user taps "Generate My Plan",
**Then** the app submits the profile payload to `/api/v1/onboarding`, receives a plan ID within 10 seconds, and shows a confirmation state (or actionable error if submission fails).

**And** when the user relaunches the app before finishing onboarding,
**Then** the wizard resumes at the last completed step with previously entered values.

---

## Implementation Details

### Tasks / Subtasks

- Build SwiftUI stepper (`OnboardingFlowView`) with progress indicator and per-step validation.
- Implement `RunnerProfileDraft` model + persistence (Keychain/FileManager) to resume onboarding.
- Create `RunnerProfileService` to call `/api/v1/onboarding` and surface loading/error states.
- Add consent & privacy copy referencing Strava data usage with checkbox gating submission.
- Capture Strava link intent and store temporary state for Story 1.2.

### Technical Summary

SwiftUI MVVM views backed by Combine, local persistence of draft data, network layer using async/await to call FastAPI endpoint, shared DTO definitions for onboarding schema.

### Project Structure Notes

- **Files to modify:** `mobile/ios/Features/Onboarding/*`, `mobile/ios/Services/RunnerProfileService.swift`, `services/orchestrator/app/routes/onboarding.py`.
- **Expected test locations:** `mobile/ios/Tests/OnboardingTests.swift`, `services/orchestrator/tests/test_onboarding_api.py`.
- **Estimated effort:** 5 story points (2 days).
- **Prerequisites:** None.

### Key Code References

N/A (greenfield) – follow tech-spec data models.

---

## Context References

**Tech-Spec:** [tech-spec.md](../docs/tech-spec.md) - Primary context document containing:

- Data model definitions (RunnerProfile, Plan) and payload schema
- API contract for `/api/v1/onboarding`
- UX copy requirements and consent text
- Implementation steps for onboarding wizard

**Architecture:** Strava-Powered Onboarding MVP – see `docs/epics.md` Epic 6 section.

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
