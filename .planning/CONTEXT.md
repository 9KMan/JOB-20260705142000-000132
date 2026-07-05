# Integration + AI Platform — Project Context

## Engagement

**Client**: Stack Growth Solutions (PE-backed finance back-office integrator)
**Job**: JOB-20260705142000-000132
**Upwork**: https://www.upwork.com/jobs/~022073498740018918928
**Bid**: $40/hr top-of-band (client open to higher)
**Shape**: Application + live portfolio walkthrough + paid 1-week test project → ongoing >6 months
**Tier**: EXPERT
**Qualifier score**: 62/100 → REVIEW → BID (manual override)

## What Stack Growth Solutions does

Builds modern finance back offices for PE-backed companies. Their client portfolio
includes companies being upgraded from spreadsheets/email-chains to integrated
back-office systems. They hire engineers to design + build the integration + AI layer.

## What the engagement covers

The senior AI Integration & Automation Engineer role covers six capability pillars:

1. **Integration layer** — REST/webhook adapters + n8n/Make/Zapier + custom Python/JS
2. **AI layer** — Claude Code agents (LangGraph), structured outputs, tool-connected workflows
3. **ETL pipelines** — extract/clean/transform/sync with source-deterministic idempotency
4. **Reconciliation** — explicit ownership table, one-way sync, scheduled drift detection
5. **Workspace buildouts** — Monday.com + Notion via API + Claude-assisted templates
6. **Template hardening** — every one-off build extracted into a cross-client template

## Why this matters now

PE-backed companies being upgraded have ZERO tolerance for:
- Silent data drift (wrong number on a board report)
- Hallucinated outputs from agents (an AI telling a CFO the wrong cash position)
- Writes that bypass audit (no record of who changed what)
- Systems that require babysitting (the founder/owner is too busy to debug every Monday)

The platform has to be **production-grade from day one**, not a demo that gets
hardened later. The paid test project is the calibration window — if the integration
scaffold can deliver ~80% of a real spec in 1 week, it's good enough for ongoing work.

## Why we're a strong fit

Prior portfolio covers every pillar:
- **JOB-128** (tuinui marketing-data pipeline): Slack + GA4 + Google Ads unified via
  Composio SDK + FastAPI. Demonstrates integration + ETL + idempotency + audit log.
- **JOB-130** (Composio CRM chatbot): two-phase commit + drift detection. Demonstrates
  reconciliation + write safety + human-in-loop.
- **JOB-126** (Persistent Reasoning Engine): structured Claude workflows with memory +
  tool calls + audit trail. Demonstrates AI layer + reliability design.

Plus 20 years of real-time POS data-pipeline work (3,000+ terminals, 24/7) covering
ETL/sync/reconciliation instincts at a level most competitors can't claim.

## Five-question answer coverage

All 5 mandatory application questions answered with specific examples in `PROPOSAL.md`:

- **Q1 (most complex end-to-end)**: tuinui marketing-data pipeline (JOB-128)
- **Q2 (no native connector)**: custom Slack Events adapter (JOB-128)
- **Q3 (advanced non-chatbot LLM)**: Persistent Reasoning Engine (JOB-126)
- **Q4 (two-system reconciliation)**: explicit ownership + drift categorization (JOB-130 pattern)
- **Q5 (rate/availability/TZ)**: $40/hr, 30-40 hrs/wk, Bangkok 8-14 BKK = 19:01-01:00 EST

## Production-shape artifacts

Per the JOB-127 production-platform pattern:
- Helm chart + Azure bicep + Dockerfile showing deployability, not just algorithm correctness.
- Reference Python scaffold under `app/` mirroring the JOB-127 directory structure
  (api/orchestrator/crm/models/observability/ui).
- 4 white-bg SVG diagrams (architecture, multi-agent, workflow, project-structure).
- 7 GSD-style planning phases under `.planning/phases/1..7/PLAN-01.md`.
- This is a **reference platform** (per JD's "this is not an easy role and the pay rate
  reflects that" — the client expects senior-level engineering maturity from the artifacts,
  not just a hand-written proposal).

## Honest disclosure (in cover letter + interview)

- **Prior work is retail/operations + AI/data engineering, not PE/finance.** JD explicitly
  says finance is a "plus, not a requirement." Architecture skills transfer; domain
  knowledge ramps in W1.
- **Bangkok TZ.** Working 8-14 BKK = 19:01-01:00 EST (6h US Eastern overlap at end of US
  workday). Async-first + 1hr daily sync at 21:00 EST = 09:00 BKK.
- **No n8n experience specifically.** Used similar middleware (MuleSoft, Apache Camel,
  custom Python). Paid test project is the calibration window.

## Bid timeline

- **Apply**: 2026-07-05 (today)
- **Live portfolio walkthrough**: TBD (within 1-2 weeks of apply)
- **Paid test project**: TBD (1 week after walkthrough if selected)
- **Ongoing engagement**: TBD (>6 months if test passes)

## Status

INTAKE → awaiting CEO approval → build pipeline.