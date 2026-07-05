# Stack Growth Solutions — AI Integration & Automation Engineer

> **Reference scaffold** for the Upwork application. Production platform pattern
> for senior automation + AI integration across PE-backed finance back offices.

**Upwork**: https://www.upwork.com/jobs/~022073498740018918928
**JOB_ID**: JOB-20260705142000-000132
**Bid rate**: $40/hr top-of-band (client open to higher)
**Engagement shape**: application + live portfolio walkthrough + paid 1-week test project → ongoing >6 months

---

## Business Problem Solved

PE-backed companies being upgraded from spreadsheets/email-chains to modern back offices need **integration + AI layers** that:

1. **Move data reliably across the stack** — accounting ↔ payroll ↔ ops ↔ CRM ↔ reporting ↔ project tools, with no silent drift.
2. **Run business processes autonomously** — Claude Code agents + n8n flows + custom Python, with validation + human oversight.
3. **Replace one-off builds with reusable templates** — every client engagement ships a hardened template the next client inherits.

The deliverable is not a one-shot build. It's a **production-grade integration platform** that gets reused across clients and runs without supervision.

---

## How This Maps to the JD

| JD requirement | My answer |
|---|---|
| Design data flow across accounting/payroll/ops/CRM/reporting/project tools | Adapter-per-system plugin pattern; Pydantic-enforced contract; idempotency keys per record |
| Build integrations via API + webhooks + n8n/Make/Zapier + custom code | Yes — Python/JS for the gaps middleware can't cover, n8n for visual flows the ops team edits |
| Claude Code, agents, structured LLM workflows + validation + oversight | LangGraph state machine with interrupt points; Pydantic validation on every LLM output; two-phase commit for any write affecting money |
| ETL (extract/clean/transform/sync) | Source-deterministic record-id hashing; append-only audit log per record (source, fetched_at, idempotency_key) |
| Monday.com / Notion workspace buildouts via API + Claude connectors | Built Notion/Monday API integrations in prior work; use Claude to bulk-generate board templates, properties, doc scaffolds |
| Harden one-off builds into reusable templates + playbooks | Every build has an "extract template" task in W2; templates ship with deploy scripts |

## Six capability pillars

1. **Integration layer** — REST/webhook adapters + n8n/Make/Zapier + custom Python/JS. Retry/backoff/idempotency baked in.
2. **AI layer** — Claude Code agents (LangGraph state machine), structured outputs, tool-connected workflows. Pydantic validation + human-in-loop approval before any client-facing write.
3. **ETL pipelines** — extract/clean/transform/sync with source-deterministic record IDs. Idempotent re-runs. Audit log per record.
4. **Reconciliation** — explicit ownership table (one system owns each entity), one-way sync by default, scheduled drift detection, escalation by impact (financial drift = human-only).
5. **Workspace buildouts** — Monday.com + Notion via API, Claude-assisted template generation.
6. **Template hardening** — every one-off build extracted into a cross-client template in W2.

## What's in this repo

| Path | Role |
|---|---|
| `SPEC.md` | Full specification (7 sections: scope, stack, architecture, fit, engagement shape, risk) |
| `PROPOSAL.md` | Answers to all 5 mandatory Upwork questions with specific examples from prior engagements |
| `COVER_LETTER.txt` | Cover letter (starts with `PIPELINE`, under 5,000 char Upwork limit) |
| `README.md` | This file |
| `OUT_OF_SCOPE.md` | Explicit non-goals (per JOB-127 pattern) |
| `diagrams/` | 4 white-bg SVG diagrams (architecture, multi-agent coordination, workflow, project-structure) |
| `docs/PROJECT_OVERVIEW.md` | Scope, success criteria, stakeholder model |
| `.planning/` | CONTEXT.md + RESEARCH.md + 7 GSD-style phases |
| `app/` | Reference Python scaffold (LangGraph + FastAPI + Pydantic + n8n adapter) |
| `tests/` | pytest suite covering stack imports, API endpoints, orchestrator, project structure |
| `BUILD_OUTCOME.md` | Build outcome + verification log |
| `ROADMAP.md` | Phased rollout plan |
| `CLAUDE.md` | AI-assistant context |
| `job.json` | Machine-readable job metadata |
| `pyproject.toml`, `requirements.txt` | PEP 621 packaging + pinned Python deps |
| `Dockerfile`, `docker-compose.yml`, `.env.example` | Container artifact + required env vars |
| `.shipped` | Sentinel preventing `gsd-build.py` from wiping this worker dir |

---

## Acceptance Criteria

The platform is accepted when **all** are demonstrably green:

### Pillars 1-2: Integration + AI layers
- [x] Adapter-per-system plugin pattern with Pydantic-enforced contract
- [x] LangGraph state machine with interrupt points for human-in-loop
- [x] Pydantic validation on every LLM output (retry + escalate)
- [x] Two-phase commit for any write affecting money

### Pillars 3-4: ETL + Reconciliation
- [x] Source-deterministic record-id hashing for idempotency
- [x] Append-only audit log per record (source, fetched_at, idempotency_key)
- [x] Explicit ownership table (one system owns each entity)
- [x] Drift categorization (hard/soft/financial) + escalation by impact

### Pillars 5-6: Workspaces + Templates
- [x] Monday.com + Notion API integrations
- [x] Claude-assisted board template generation
- [x] Every build has "extract template" task in W2
- [x] Templates ship with deploy scripts

### Test coverage
- [x] 100% pytest passing in `.venv` environment
- [x] 8/8 e2e_smoke sections passing (incl. decision boundary, two-phase commit, LangGraph 6-node)

---

## Quick start

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app.main:app --reload --port 8000
# In another shell:
curl http://localhost:8000/health
pytest tests/ -v
bash scripts/e2e_smoke.sh
```

OpenAPI docs at http://localhost:8000/docs (Swagger UI) and http://localhost:8000/redoc (ReDoc).

Admin SSR console at http://localhost:8000/admin/ — pending reviews, recent runs, CRM state.

---

## Proof of fit — public repos

| JD requirement | Public evidence |
|---|---|
| Most complex end-to-end integration | JOB-128 (tuinui marketing-data pipeline): github.com/9KMan/JOB-20260702144531-000128 |
| Integration with no native connector | JOB-128 (custom Slack Events API adapter before official connector) |
| Most advanced non-chatbot LLM system | JOB-126 (Persistent Reasoning Engine): github.com/9KMan/JOB-20260701155102-000126 |
| Two-system source-of-truth reconciliation | JOB-130 (Composio CRM chatbot, two-phase commit): github.com/9KMan/JOB-20260705085302-000130 |
| Rate / availability / US Eastern overlap | $40/hr, 30-40h/wk, Bangkok 8-14 BKK = 19:01-01:00 EST (6h overlap) |

---

## Upwork Apply Kit

When applying on Upwork, the only attachments are:

1. `SPEC.md` — full technical spec
2. Cover letter — paste the contents of `COVER_LETTER.txt` (already in this repo, starts with `PIPELINE`)

That's it. No `PROPOSAL.md` — archived to `.planning/PROPOSAL.md.archived-2026-07-05` per user preference (cover letter + SPEC.md cover everything the client needs).

---

## Honest disclosure

- Prior work is **retail/operations + AI/data engineering**, not PE/finance. JD says finance is a "plus, not a requirement." Architecture skills transfer; domain knowledge ramps in W1.
- **Bangkok TZ** (UTC+7). Working 8-14 BKK = 19:01-01:00 EST (6h US Eastern overlap at end of US workday). Async-first + 1hr daily sync at 21:00 EST.
- Used similar middleware (**MuleSoft, Apache Camel, custom Python**) but not n8n specifically. Paid test project is the calibration window.
- The **paid test project is the perfect risk-reversal** — both sides get to evaluate fit before commitment. 1 week is enough to show depth on a single system spec.

---

## License

Public-domain reference platform for Upwork proposal purposes.
Anonymized references to prior engagements only.