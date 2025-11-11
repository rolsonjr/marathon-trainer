# marathon-trainer - Product Requirements Document

**Author:** Rodney
**Date:** 2025-11-11
**Version:** 1.0

---

## Executive Summary

Coach.ai (internal code name marathon-trainer) is a mobile-first conversational coaching companion that pairs a charismatic lead coach (Coach George) with specialist analytics and physical-therapy agents. The ensemble ingests Garmin workout files, schedule constraints, and injury signals to recompute marathon training plans in near real time so runners can stay confident and healthy on the path to race day—without hiring a human coach.

### What Makes This Special

Coach George consistently delivers workouts that feel like they were handcrafted for the runner’s exact race countdown, goal pace, experience level, and current aches—instantly adapting the plan the moment new data or conflicts appear.

---

## Project Classification

**Technical Type:** Mobile app + agentic coaching service  
**Domain:** Health & fitness (injury-aware marathon training)  
**Complexity:** High (medical/therapy considerations + adaptive analytics)

- Mobile-first experience with conversational UI backed by a LangGraph-style orchestration layer.
- Must fuse consumer-grade delight with quasi-clinical judgment from analytics and PT agents.
- Handles sensitive wellness and injury data, demanding thoughtful guardrails even though it is not a regulated medical device (yet).

### Domain Context

Runners will disclose injury history, pain signals, and biometric workout data so Coach George and the PT agent can prescribe safe adjustments. That creates quasi-medical obligations: evidence-based recovery advice, transparent limitations (not a doctor), HIPAA-adjacent privacy expectations, and careful messaging when altering load. The analytics agent also needs trustworthy interpretations of Garmin metrics (distance, pace, HR zones, variability) so training blocks stay physiologically sound.

---

## Success Criteria

- **Adaptive plan adherence:** 90% of workouts in the standard demo dataset (3 sample runners × 7 days) are updated within 5 minutes of new Garmin data or user input, and at least 80% of those updates include rationale citing analytics/PT signals (PT agent coverage required).
- **Runner confidence signal:** Average 8/10 “feels seen” rating from test users immediately after Coach George explains each plan adjustment tied to countdown, injury status, or schedule blocker.
- **Safety assurance:** Every injury or pain report triggers a PT-agent response plus modified load within the same conversation, with explicit “not a doctor” guardrails; log dependency on PT agent availability.
- **Personalization depth:** Weekly plan summaries highlight at least four personalization factors (race timeline, goal pace, experience tier, injury/recovery state) and show mastery deltas for key workout types using the standard demo dataset.
- **System reliability:** End-to-end orchestration (data ingestion → analytics → plan update → chat) succeeds across all three simulated runner profiles in the demo dataset without manual intervention.

### Business Metrics (assumptions)

- **Launch readiness:** Produce a compelling demo video and README that win 3+ recruiter conversations or pilot user interviews, or (fallback) secure internal stakeholder sign-off if external scheduling slips.
- **Engagement proxy:** 70% of testers request a second weekly plan within 14 days of their first plan, indicating perceived ongoing value.

---

## Product Scope

### MVP - Minimum Viable Product

- Mobile-first Coach.ai experience connected to a LangGraph orchestration backend capable of ingesting Garmin FIT files via a single manual upload path (automated sync deferred).
- Analytics agent that reviews each completed workout from the demo dataset and returns mastery deltas, pacing accuracy, and effort guidance back into the same chat session.
- Weekly training planner that recalculates the upcoming 7-day block every Sunday (or on-demand) using race countdown, target finish time, injury state, and schedule constraints.
- Conversational workflow with Coach George limited to three intents (schedule changes, pain/injury updates, goal adjustments) so plan modifications remain focused and auditable; all health suggestions include “not a doctor” guardrails.
- Lightweight privacy scaffolding (consent text, retention notes) to reassure runners that medical-style data stays protected even though the product is pre-regulation.

### Growth Features (Post-MVP)

- Automated Garmin sync plus persistent multi-agent deliberation (Coach George consults analytics and PT agents before committing significant plan changes), yielding explanations Garmin Connect cannot match.
- Structured injury recovery playbooks, load ramp caps, and automatic down-weeks triggered by HRV/fatigue signals rather than only user-declared pain.
- Competitive benchmarking panel that compares progress vs. similar runners, highlighting differentiated guidance from Coach George.
- Integrated schedule intelligence (calendar import, travel flags) so the system adjusts plans proactively when conflicts emerge.

### Vision (Future)

- Customizable coach personas (tone, expertise emphasis) and training templates for half marathons, ultras, triathlons, and multi-race seasons—dependent on mature analytics telemetry.
- Nutrition strategist agent that pairs with PT guidance to adjust fueling recommendations alongside training intensity.
- Motivation/psychology agent providing contextual encouragement, accountability nudges, and taper-week mental prep (prerequisite for personalized coach personas).
- Multimodal inputs (voice check-ins, video gait uploads) feeding evaluation models so Coach George can comment on form, mood, and recovery quality.

---

## Domain-Specific Requirements

- Coach George is not a licensed medical professional; every health recommendation must include disclaimers and options to consult a human specialist.
- Injury and pain data qualify as sensitive wellness information: encrypt in transit and at rest, enforce least-privilege agent access, and provide transparent data-retention policies.
- PT agent guidance must reference evidence-informed rehab principles (load management, return-to-run protocols) and log decision rationale for auditability.
- Any automated plan change driven by biometric signals must document the inputs (e.g., HRV drop, pace variance) so runners understand why the recommendation is safe.
- If the product ever ingests protected health information or connects to regulated care, the team must evaluate HIPAA and FDA Software as a Medical Device implications before release.

This section shapes all functional and non-functional requirements below.

---

## Innovation & Novel Patterns

- **Agentic coaching ensemble:** Coach George acts as the face while analytics and PT agents operate as specialist advisors, giving runners a “staff” that no competing app simulates today.
- **Mastery-driven plan rewrites:** Instead of simple pace feedback, the analytics agent recalculates mastery scores per workout type and feeds them back into LangGraph to reshape plans mid-week.
- **Context-rich explanations:** Every adjustment cites the exact signals (race countdown, injury state, mastery deltas, schedule blockers) so the runner feels the rationale, not just the output.

### Validation Approach

- Build scripted simulations for three runner archetypes (novice, returning-from-injury, Boston-qualifier) and verify that each plan rewrite references at least three distinct signals.
- Run shadow comparisons against Garmin Connect sample plans to confirm Coach George surfaces more granular rationale and injury accommodations.
- Conduct moderated usability tests where runners challenge Coach George with conflicting inputs (injury + goal increase) to ensure the agent ensemble produces coherent guidance and safe guardrails.

---

## Mobile App Specific Requirements

- SwiftUI-first experience backed by LangGraph orchestration (Python backend) accessed through secure APIs; Android arrives later once orchestration patterns stabilize.
- Background sync is mandatory so Garmin FIT uploads queue and send when connectivity resumes; MVP can rely on manual FIT import but must automatically push once online.
- Offline mode is read-only (view cached plan + chat history). Any adjustments or data uploads require connectivity to keep the agent graph consistent.
- Apple App Store compliance demands explicit disclosures: health data collection, “not a medical device” copy, and the privacy nutrition label covering workout/injury fields.

### Platform Support

- Target iOS 17+ for BackgroundTasks, App Intents, and the latest notification permission flows.  
- Provide a pared-down webview/HTML export of weekly plans for Apple Watch glanceability and future cross-platform parity.

### Device Capabilities

- Integrate HealthKit document picker plus manual FIT file selection to cover runners before Garmin API approval arrives.  
- Use background transfer services to upload FIT files and show chat badges indicating whether analytics has processed the latest workout.  
- Support local notifications reminding runners to confirm injury status when anomalies (e.g., HR spikes) are detected once analytics completes.

---

## User Experience Principles

- Tone: Coach George should feel like a trusted mentor—professional but encouraging, never cheesy. Visual language stays clean with subtle athletic accents (dark mode friendly).
- Critical signals (injury alerts, mastery changes) must appear in conversational context, not hidden in charts, so runners always see *why* plans changed.
- Chat-first navigation: the primary canvas is the conversation thread with embedded cards for workouts, analytics explanations, and suggested adjustments.
- Accessibility: large tap targets, optional haptic confirmations, and clear color contrast so fatigued runners can skim quickly post-workout.

### Key Interactions

- **Onboarding goals intake:** conversational flow capturing race date, goal time, experience tier, injury history, and schedule blockers with progressive disclosure to avoid fatigue.
- **Workout review card:** after each Garmin upload, Coach George summarizes mastery deltas, effort guidance, and asks for subjective feedback (“felt easy / on target / overcooked”).
- **Weekly plan carousel:** horizontally scrollable cards embedded in chat showing day-by-day workouts; tapping a card reveals rationale and allows quick adjustments (“swap days”, “lighten load”).
- **Injury alert dialogue:** when pain is reported, PT agent card appears with rehab guidance, while Coach George confirms plan adjustments and reiterates health disclaimers.

---

## Functional Requirements

- **R1 · Goal & Profile Intake:** Conversational onboarding must capture race date, goal time, experience tier, injury history, schedule blockers, and device sync preference within 5 minutes; all fields saved to runner profile for reuse in plan generation.
- **R2 · Garmin Data Import:** Support manual FIT file upload from iOS Files/HealthKit at launch; files are validated (timestamp, distance, pace) and queued for processing with status surfaced in chat. Background transfer retries until success or explicit failure message.
- **R3 · Analytics Mastery Engine:** After each workout ingestion, compute mastery deltas per workout type (speed, endurance, recovery) plus effort guidance; results stored alongside raw metrics and sent to Coach George within 5 minutes 90% of the time.
- **R4 · Weekly Plan Generator:** Every Sunday at 6am local (or on user request) regenerate a 7-day schedule that references race countdown, mastery deltas, injury flags, and schedule blockers. Output includes rationale tags so chat summaries can cite “what changed”.
- **R5 · Conversational Adjustments:** Coach George must handle at least three intents—schedule conflicts, injury/pain updates, and goal changes—returning an updated plan snippet plus explanation in-thread. All health-related responses include PT guardrails and disclaimers.
- **R6 · PT Agent Guidance:** When pain or injury context appears, trigger PT agent to deliver load modification + rehab suggestion. Chat transcript logs the trigger reason and the PT response, enabling future audit and analytics.
- **R7 · Safety & Compliance Layer:** Present consent text before first data upload, allow runners to review/delete stored workouts, and append “not medical advice” language whenever PT or injury-related guidance appears. All such actions recorded in audit log.
- **R8 · Plan & Insight Surfacing:** Embed workout cards, weekly carousel, and notification badges directly in chat with quick actions (“swap days”, “lighten load”, “accept change”). Acceptance must update the schedule immediately and show confirmation.
- **R9 · Background Sync & Offline Cache:** When the app is reopened after a workout, queued FIT uploads send automatically; if offline, the last synced plan and chat remain viewable but editing is disabled with a banner explaining why.
- **R10 · Demo & Reporting Assets:** Generate shareable plan exports (Markdown/PDF) and a system log that demonstrates end-to-end orchestration for the three demo runner archetypes, enabling recruiter demos and future architecture handoffs.

---

## Non-Functional Requirements

### Performance

- Garmin ingestion + analytics loop must complete within 5 minutes for 90% of workouts in the demo dataset; weekly plan regeneration must finish within 30 seconds.
- Chat responses for plan adjustments should render in under 2 seconds after analytics output lands to preserve conversational flow.

### Security

- Encrypt all workout and injury data in transit (TLS 1.3) and at rest (AES-256). Limit PT agent access tokens to least privilege needed for rehab suggestions.
- Log every injury-related adjustment with timestamp, triggering data points, and user acknowledgement to create an auditable trail.
- Provide user-facing controls to delete synced workouts and export their data, complying with privacy expectations for sensitive wellness info.

### Scalability

- Architecture must support at least 1,000 concurrent runners in POC environments, ensuring orchestration can parallelize analytics jobs without starving chat latency.
- LangGraph workflow definitions should be modular so new race types or agents can plug in without reauthoring the entire graph.

### Accessibility

- Follow WCAG 2.1 AA contrast ratios and provide scalable typography so post-run users can read summaries easily.
- Offer haptic confirmations and voice-over labels for training cards and quick actions.

### Integration

- Provide a documented API/webhook pattern for future integrations (Garmin official API, calendar sync). Initial MVP uses manual FIT imports but architecture must anticipate OAuth-based pipelines.

---

## Implementation Planning

### Epic Breakdown Required

Requirements must be decomposed into epics and bite-sized stories (200k context limit).

**Next Step:** Run `workflow epics-stories` to create the implementation breakdown.

---

## References

- Product Brief: docs/bmm-product-brief-coach.ai-2025-11-11.md
- Domain Brief: _Not yet created — capture rehab protocols, risk language, and compliance boundaries._
- Research: _None documented yet; commission domain or market research if needed._

---

## Next Steps

1. **Epic & Story Breakdown** - Run: `workflow epics-stories`
2. **UX Design** (if UI) - Run: `workflow ux-design`
3. **Architecture** - Run: `workflow create-architecture`

---

_This PRD captures the essence of {{project_name}} - {{product_magic_summary}}_

_Created through collaborative discovery between {{user_name}} and AI facilitator._
