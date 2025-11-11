
# marathon-trainer - Epic Breakdown

**Author:** Rodney
**Date:** 2025-11-11
**Project Level:** Level 2 (Method track)
**Target Scale:** Pilot cohort (~1k runners)

---

## Overview

This document provides the complete epic and story breakdown for marathon-trainer, decomposing the requirements from the [PRD](./PRD.md) into implementable stories.

**Epic Structure Proposal**

1. **Foundation & Orchestration Enablement** – Establish the SwiftUI shell, LangGraph backend scaffolding, environments, and CI/CD so every other epic has a stable lane. Includes initial HealthKit permissions, secret management, and demo runner datasets. _Sequence: must land first._
2. **Runner Profile & Data Intake** – Deliver conversational onboarding, consent/privacy flows, and manual FIT ingestion with background retry plus offline/read-only cache. Covers R1, R2, R7 (consent aspects), and groundwork for future sensors.
3. **Analytics & Mastery Intelligence** – Build the processing pipeline that turns FIT data into mastery deltas, effort guidance, anomaly detection, and interfaces to downstream agents. Enforces the 5-minute SLA and stores rationale tags. Depends on Epic 2 data plumbing.
4. **Adaptive Planning & Conversational Coach** – Generate weekly plans, handle schedule/injury/goal chat intents, surface plan cards, and update plans in-thread with rationale. Relies on Epics 2–3 for data + analytics.
5. **PT Safety, Compliance & Demo Assets** – Implement PT-agent flows, injury guardrails, audit logging, accessible plan exports, and demo/reporting artifacts required for launch-readiness metrics. Can run partly in parallel with Epics 3–4 but final verification happens last.

---

## Epic 6: Strava-Powered Onboarding MVP

**Goal:** Give first-time runners an end-to-end experience that captures their race goals, links Strava, ingests recent workouts, and returns a rationale-rich 7-day plan within five minutes.

**Scope:** SwiftUI onboarding wizard, Strava OAuth swap (no Garmin), activity ingestion worker, LangGraph weekly plan composer, and plan carousel display with offline cache. Out of scope: Android/web, PT automation, advanced mastery scoring.

**Success Criteria:**

- 100% of onboarding submissions produce a Strava-linked profile and plan ID.  
- Plans render with rationale chips and reflect runner availability preferences.  
- Strava sync + plan generation telemetry captured for three demo athletes.  
- Offline mode presents cached plan with readonly banner.

**Dependencies:** Strava developer credentials, Postgres/Redis infra, LangGraph runtime from Epic 1.

---
## Epic 1: Foundation & Orchestration Enablement

Establish the SwiftUI shell, LangGraph backend scaffolding, continuous integration, environment configuration, secrets management, and demo runner datasets so other epics can build on a reliable core. This epic delivers R1 prerequisites plus the infrastructure needed for ingestion, analytics, and chat.

### Story 1.1: Project Scaffold & CI/CD Baseline

As a implementation engineer,
I want a SwiftUI repo paired with a LangGraph backend workspace and automated CI/CD checks,
So that every future story lands on a consistent, testable foundation.

**Acceptance Criteria:**

- Given a fresh repo, When I bootstrap the SwiftUI app, LangGraph service, and infrastructure config, Then the codebase contains documented setup scripts plus CI pipelines (lint/test/build) running on the main branch.
- And when credentials are configured for staging, Then automated builds deploy to a sandbox environment with smoke tests.

**Prerequisites:** None

**Technical Notes:** Create mono-repo or coordinated repos with shared versioning; include Fastlane/GitHub Actions templates; document environment variables.

### Story 1.2: LangGraph Workflow Skeleton

As a platform engineer,
I want baseline LangGraph workflows (ingestion, analytics, PT hooks) stubbed with test agents,
So that downstream stories can plug real logic into proven paths.

**Acceptance Criteria:**

- Given the CI-ready repo, When I run the LangGraph workflow locally, Then it exposes nodes for ingestion, analytics, PT agent, and Coach George with mocked outputs.
- And when new nodes are added later, Then the workflow definition allows hot-reload without breaking existing contracts.

**Prerequisites:** Story 1.1

**Technical Notes:** Define graph schema, state passing, tracing hooks; include unit tests using mock FIT payloads.

### Story 1.3: Secrets & Environment Configuration

As a DevOps engineer,
I want secure storage for API keys, HealthKit entitlements, and PT agent credentials,
So that sensitive data never leaks while dev/stage environments remain easy to configure.

**Acceptance Criteria:**

- Given staging and local environments, When I provision secret management (e.g., 1Password, SSM), Then all services load credentials via config files excluded from source control.
- And when developers run setup scripts, Then they receive sample env files plus documentation on rotating keys.

**Prerequisites:** Story 1.1

**Technical Notes:** Tie into CI workflows, enforce lint checks blocking secret commits, include placeholder 'not a medical device' copy for localization.

### Story 1.4: Demo Runner Dataset & Telemetry Logging

As a product strategist,
I want three seeded runner profiles with sample FIT data and telemetry dashboards,
So that success criteria can be validated continuously.

**Acceptance Criteria:**

- Given the LangGraph skeleton, When I ingest the sample FIT files, Then the system stores them with metadata (runner type, date, metrics) and exposes replay scripts.
- And when stakeholders view telemetry, Then they can see ingestion latency, plan-update timestamps, and PT trigger counts.

**Prerequisites:** Stories 1.1 and 1.2

**Technical Notes:** Use lightweight SQLite/Postgres for demo data; produce dashboard or JSON logs feeding the README.

### Story 1.5: HealthKit Permissions & Background Task Hooks

As a iOS engineer,
I want the app to request HealthKit/file permissions and register background task handlers,
So that later ingestion stories can rely on OS-approved access.

**Acceptance Criteria:**

- Given the SwiftUI shell, When a tester opens the app, Then onboarding surfaces Apple-required consent text and captures permission decisions.
- And when the app enters background, Then registered tasks wake to process pending uploads (mocked) and log success/failure to telemetry.

**Prerequisites:** Stories 1.1 and 1.3

**Technical Notes:** Implement placeholder consent UI, integrate BGTaskScheduler IDs, confirm compliance text matches PRD requirements.

---

## Epic 2: Runner Profile & Data Intake

Deliver conversational onboarding, consent/privacy flows, manual FIT ingestion with background retry, and read-only offline cache so runners can onboard quickly and Coach George always has fresh data (covers PRD R1, R2, and consent portions of R7).

### Story 2.1: Conversational Goals & Profile Intake

As a runner,
I want onboarding prompts that capture race date, goal time, experience, injury history, schedule blockers, and sync preference,
So that Coach George personalizes plans immediately.

**Acceptance Criteria:**

- Given a first-time user, When they progress through onboarding, Then each required field is captured with validation and saved to their profile.
- And when the user returns, Then they can edit any field without restarting onboarding.

**Prerequisites:** Story 1.1

**Technical Notes:** Use progressive disclosure UI, leverage local + backend persistence, include 'not a doctor' reminder copy.

### Story 2.2: Consent, Privacy & Data Controls

As a privacy-conscious runner,
I want explicit consent copy plus export/delete controls,
So that they trust storing sensitive workout data.

**Acceptance Criteria:**

- Given onboarding completes, When I review privacy settings, Then I see consent text covering health data usage and disclaimers.
- And when I request export/delete, Then the backend fulfills the request within the session and logs the action.

**Prerequisites:** Stories 2.1 and 1.3

**Technical Notes:** Add audit logging hook for PRD R7; define export format (JSON/CSV).

### Story 2.3: Manual FIT Upload Flow

As a runner,
I want to import FIT files from HealthKit/Files with validation feedback,
So that Coach George ingests workouts without Garmin APIs.

**Acceptance Criteria:**

- Given permissions granted, When I choose a FIT file, Then the app validates timestamp/distance/pace and shows success or actionable errors.
- And when upload succeeds, Then the file is queued for analytics with status shown in chat.

**Prerequisites:** Stories 1.5 and 2.1

**Technical Notes:** Support fake datasets; store metadata for SLA tracking.

### Story 2.4: Background Retry & Status Surfacing

As a runner,
I want uploads to retry automatically when connectivity returns and to see status badges,
So that workouts aren’t lost due to spotty service.

**Acceptance Criteria:**

- Given a failed upload due to connectivity, When the app regains network or opens, Then it retries automatically and updates status chips (pending, syncing, complete).
- And when retries exceed limits, Then I get a chat notification with guidance to re-upload.

**Prerequisites:** Stories 1.5 and 2.3

**Technical Notes:** Use background task queue, log retries for telemetry.

### Story 2.5: Offline Read-Only Cache

As a runner,
I want to view the last synced plan/chat while offline,
So that they can still follow workouts on the go.

**Acceptance Criteria:**

- Given the device is offline, When I open the app, Then I see a banner explaining limited mode and can view cached schedule/chat.
- And when I attempt to edit, Then I’m blocked with messaging to reconnect.

**Prerequisites:** Stories 2.1 and 2.3

**Technical Notes:** Store last plan payload + timestamp; clear cache on logout.

---

## Epic 3: Analytics & Mastery Intelligence

Transform uploaded FIT data into mastery deltas, effort guidance, anomaly detection, and rationale tags that downstream agents and Coach George can trust (fulfills PRD R3 and feeds R5–R8).

### Story 3.1: Ingestion Pipeline & Data Schemas

As a data engineer,
I want a pipeline that parses FIT files into normalized workout records,
So that analytics logic has reliable inputs.

**Acceptance Criteria:**

- Given new FIT uploads, When the pipeline processes them, Then records capture timestamp, distance, pace, HR stats, and workout type.
- And when parsing fails, Then descriptive errors flow back to the Epic 2 upload queue for display.

**Prerequisites:** Stories 1.2 and 2.3

**Technical Notes:** Use Postgres with migrations; log ingestion duration for SLA metrics.

### Story 3.2: Mastery Scoring Engine

As a analytics engineer,
I want mastery deltas per workout category versus plan targets,
So that progress stays measurable.

**Acceptance Criteria:**

- Given a new workout record, When the engine runs, Then mastery scores update within 5 minutes and store rationale references.
- And when demo runner datasets process, Then sample mastery outputs are visible for reviewers.

**Prerequisites:** Story 3.1

**Technical Notes:** Implement formulas + unit tests; persist to analytics table accessible by LangGraph.

### Story 3.3: Effort Guidance & Recovery Signals

As a Coach George,
I want qualitative effort summaries and recovery recommendations,
So that runners know how to respond after uploads.

**Acceptance Criteria:**

- Given mastery updates, When analytics runs, Then each workout receives guidance (on target/overcooked) plus numeric deltas.
- And when potential injury signals surface, Then PT trigger thresholds are logged (prototype level).

**Prerequisites:** Story 3.2

**Technical Notes:** Map HR variance and pace deviation; store outputs for chat consumption.

### Story 3.4: Rationale Tagging API

As a orchestration layer,
I want structured tags bundling mastery, injury flags, schedule blockers, and countdown,
So that chat responses cite exact signals.

**Acceptance Criteria:**

- Given plan generation, When the API is called, Then it returns tags (e.g., countdown=12, injury='IT band', mastery='speed +5') within 2 seconds.
- And when plans update multiple times, Then cached tags respect TTL and invalidate correctly.

**Prerequisites:** Stories 3.2 and 2.1

**Technical Notes:** Expose REST/GraphQL endpoint; add caching layer.

### Story 3.5: SLA Monitoring & Alerting

As a operator,
I want dashboards and alerts for ingestion/analytics latency,
So that 5-minute SLAs never silently fail.

**Acceptance Criteria:**

- Given telemetry data, When latencies exceed thresholds, Then alerts fire (email/Slack) and logs capture timestamps.
- And when viewing the dashboard, Then I can see percentile latency for each runner archetype.

**Prerequisites:** Stories 1.4 and 3.2

**Technical Notes:** Reuse telemetry stack; integrate lightweight alerting.

---

## Epic 4: Adaptive Planning & Conversational Coach

Generate weekly plans, honor schedule/injury/goal chat intents, and surface plan cards with rationale-backed adjustments (covers PRD R4, R5, R8, R9).

### Story 4.1: Weekly Plan Generator Service

As a Coach George,
I want a service that recomputes a 7-day block using countdown, mastery, injury, and schedule data,
So that runners always see an up-to-date plan.

**Acceptance Criteria:**

- Given Sunday 6am local or an on-demand trigger, When the service runs, Then a 7-day plan is generated with rationale tags per workout.
- And when demo runners execute, Then three distinct plan versions are persisted with history.

**Prerequisites:** Stories 3.2 and 2.1

**Technical Notes:** Parameterize workouts per archetype; persist revisions with version IDs.

### Story 4.2: Schedule Conflict Intent Handling

As a runner,
I want to inform Coach George about schedule blockers and receive immediate plan tweaks,
So that they stay compliant without manual replanning.

**Acceptance Criteria:**

- Given a schedule-change message, When the intent classifier detects it, Then the workflow adjusts affected days, refreshes plan cards, and logs change reason.
- And when the runner disagrees, Then they can undo or choose quick replies (swap/lighten).

**Prerequisites:** Stories 4.1 and 2.4

**Technical Notes:** Provide quick replies; ensure adjustments respect offline sync queue.

### Story 4.3: Injury & Goal Intent Adjustments

As a runner,
I want to report pain or change goal pace and see plan updates in-thread,
So that they trust Coach George’s responsiveness.

**Acceptance Criteria:**

- Given an injury intent, When processed, Then PT stub response + plan reduction occur with disclaimers shown.
- And given a goal-change intent, When processed, Then pacing blocks recalc and rationale tags are cited.

**Prerequisites:** Stories 3.4 and 2.2

**Technical Notes:** Hook into Epic 5 PT templates; maintain audit logs referencing tag IDs.

### Story 4.4: Plan Cards & Quick Actions in Chat

As a runner,
I want embedded cards showing upcoming workouts with quick actions,
So that they can modify or accept plans without leaving chat.

**Acceptance Criteria:**

- Given plan data, When rendered, Then a horizontal carousel appears with day-by-day cards and quick actions.
- And when offline mode is detected, Then cards remain viewable but actions are disabled with messaging.

**Prerequisites:** Story 4.1

**Technical Notes:** Ensure accessibility (contrast, haptics); integrate with acceptance queue.

### Story 4.5: Acceptance & Background Sync Glue

As a engineer,
I want plan acceptance to sync with offline caches and telemetry,
So that the system stays consistent.

**Acceptance Criteria:**

- Given a plan acceptance, When triggered, Then updates queue for offline cache and writes telemetry event.
- And when network returns, Then acceptance status syncs to backend and logs the event.

**Prerequisites:** Stories 2.5 and 4.4

**Technical Notes:** Reuse upload queue; track states (pending/accepted).

---

## Epic 5: PT Safety, Compliance & Demo Assets

Implement PT-agent flows, injury guardrails, audit logging, accessible exports, and demo/reporting artifacts required for launch-readiness metrics (covers PRD R6, remainder of R7, and R10).

### Story 5.1: PT Agent Response Templates

As a PT agent,
I want structured templates for common injuries with load-modification guidance and disclaimers,
So that runners receive consistent, safe advice.

**Acceptance Criteria:**

- Given a pain report, When processed, Then a template is selected by body part/severity and response includes rehab suggestion + 'not medical advice' copy.
- And when stored, Then rationale references link to analytics/plan context.

**Prerequisites:** Stories 3.3 and 4.3

**Technical Notes:** Keep recommendations informational; flag future need for licensed review if scope expands.

### Story 5.2: Injury Audit Logging & Guardrails

As a compliance owner,
I want every injury-related adjustment logged with timestamp, triggering data, user acknowledgement, and PT response,
So that the team can prove adherence to safety expectations.

**Acceptance Criteria:**

- Given an injury adjustment, When finalized, Then an append-only log entry is created with required fields.
- And when the runner reviews the change, Then they must acknowledge before it takes effect.

**Prerequisites:** Stories 5.1 and 2.2

**Technical Notes:** Store logs in secure location with retention policy; expose export tool.

### Story 5.3: Accessible Plan & Data Exports

As a runner,
I want weekly plan and training history exportable (Markdown/PDF) with accessibility considerations,
So that they can share progress with coaches or recruiters.

**Acceptance Criteria:**

- Given the export action, When invoked, Then Markdown + optional PDF are generated including rationale highlights and disclaimer footers.
- And when audited for accessibility, Then documents pass basic contrast/structure checks.

**Prerequisites:** Stories 4.1 and 2.2

**Technical Notes:** Integrate with share sheet; reuse PRD template motifs.

### Story 5.4: Demo Telemetry & README Enhancements

As a product marketer,
I want scripted demo flows, telemetry screenshots, and README updates,
So that stakeholders can see end-to-end orchestration.

**Acceptance Criteria:**

- Given the demo dataset, When scripts run, Then README documents architecture, success metrics, and replay instructions with telemetry screenshots.
- And when recruiters run the demo, Then they can follow the instructions without extra setup.

**Prerequisites:** Stories 1.4 and 4.2

**Technical Notes:** Include video script outline; anonymize data before sharing.

### Story 5.5: Safety & Compliance Review Dry Run

As a product manager,
I want a checklist-driven dry run covering injury, privacy, and plan-adjustment scenarios,
So that guardrails are validated before handoff.

**Acceptance Criteria:**

- Given prepared scenarios, When executed, Then outputs/disclaimers/logs are documented for injury, schedule conflict, and goal change paths.
- And when gaps are discovered, Then remediation actions and owners are recorded.

**Prerequisites:** Stories 5.1–5.4

**Technical Notes:** Store checklist alongside audit logs; use GxP-style review format.

---

_For implementation: Use the `create-story` workflow to generate individual story implementation plans from this epic breakdown._
