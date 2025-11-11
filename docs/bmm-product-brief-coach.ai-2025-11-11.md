# Product Brief: coach.ai

**Date:** 2025-11-11
**Author:** Rodney
**Context:** Solo runner-driven greenfield project

---

## Executive Summary

Runner-focused AI coaching platform born from firsthand marathon prep struggles; aims to deliver adaptive training plans that reroute around injuries and scheduling conflicts using agentic orchestration.

---
## Core Vision

### Problem Statement

Serious amateur runners lack a coach that can keep their race goals intact when life or injury forces plan deviations; current training apps simply flag missed workouts instead of recalibrating the path to race day.

### Problem Impact

- Training setbacks compound quickly: even a week off for injury can erase months of gains if plans resume blindly.
- Runners lose confidence without authoritative guidance on how to adjust load while healing.
- Generic plans ignore schedule disruptions, making it hard to stay compliant with work/family demands.

### Proposed Solution

Coach.ai orchestrates specialist agents (Coach George, analytics, physical therapist) to deliver a living marathon plan that adapts weekly to execution data, recovery signals, and real-world conflicts so runners can stay on track safely.

### Key Differentiators

- Agent ensemble mirrors a coaching staff—motivational lead coach plus data and rehab experts.
- Mastery tracking per workout type drives smarter progressions than static plan templates.
- Injury-aware adjustments blend recovery prescriptions with training edits instead of forcing full stops.

---
## Target Users

### Primary Users

Serious amateur runners targeting marathons (first-time or repeat) who juggle demanding schedules and want a coach-like experience without hiring a human.

### Secondary Users

Future expansion: other endurance athletes (half-marathoners, triathletes) facing similar injury and scheduling hurdles.

---
### User Journey

1. **Onboarding:** Runner states race date, goal time, current mileage, injury history, and schedule constraints. Coach George sets tone and collects opt-ins for data sync.
2. **Weekly Planning:** Every Sunday, Coach George publishes a personalized plan grounded in mastery scores, recovery status, and upcoming schedule blockers.
3. **Workout Sync:** After each run, smartwatch data auto-sends to coach.ai. Analytics agent measures execution vs plan; PT agent flags any stress signals.
4. **Dynamic Adjustment:** If the runner under- or over-performs, Coach George updates the remaining week's workouts, citing rationale from analytics/PT recommendations.
5. **Dialogue Loop:** Runner chats with Coach George anytime to log conflicts, discuss pain, or adjust goals. The coach responds in-session with coordinated guidance.
6. **Injury Detour:** When pain is logged, PT agent prescribes recovery protocol while Coach George recalibrates load and timelines.
7. **Race Week:** System transitions to taper + psychological prep, reflecting the runner’s mastery and highlighting readiness stats.

---
## MVP Scope

### Core Features

- **Garmin Sync + Analysis:** Pull FIT data after every run, compute mastery metrics, and flag deviations.
- **Agentic Weekly Planner:** Coach George consults analytics + PT agents to publish and adjust weekly plans tied to race goals.
- **Conversational Interface:** Chat UI for runners to discuss schedule changes, pain signals, and goal updates directly with Coach George.

### Out of Scope for MVP

- Automated integrations with non-Garmin wearables.
- Real-time push notifications; MVP relies on in-app updates.
- Complex nutrition or strength training planning.

### Future Vision

- Multimodal insights (video form analysis, sentiment detection).
- Proactive injury prediction using longitudinal models.
- Community features where runners compare mastery progress.

---
## Success Metrics

- Demonstrate end-to-end agentic orchestration (data ingestion → analytics → adaptive plan → chat experience) using modern frameworks (e.g., LangGraph + LLMs).
- Showcase dynamic adjustments validated by sample Garmin data runs.
- Present a polished UX demo highlighting Coach George dialogue and plan updates.

### MVP Success Criteria

- Given sample Garmin workouts, system adjusts the remaining week automatically within minutes.
- Chat experience can handle at least three conversation threads (schedule, injury, goal change) without context loss.
- README/demo video communicates the architecture and personalization logic clearly for recruiters.

---
## Technical Preferences

- Python backend to keep prototyping fast.
- LangChain + LangGraph stack for LLM orchestration and agent workflows.
- Flexible on database/storage (any document or vector store works for MVP).

---
