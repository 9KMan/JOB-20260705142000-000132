# SPEC — AI Integration & Automation Engineer for Stack Growth Solutions

**Upwork**: https://www.upwork.com/jobs/~022073498740018918928
**JOB_ID**: JOB-20260705142000-000132
**Bid rate**: $40/hr top-of-band (client open to higher for fit)
**Engagement shape**: application + live portfolio walkthrough + paid 1-week test project → ongoing >6 months

---

## 1. Executive summary

Stack Growth Solutions builds modern finance back offices for PE-backed companies. This engagement covers the **integration + AI layer** across their client portfolio: data pipelines between accounting/payroll/ops/reporting + Claude-Code agents/automations embedded in real business processes. The deliverable is not a one-shot build — it's a **production-grade integration platform** that gets reused across clients (templates + playbooks) and runs without supervision.

The five hard requirements (3+ yrs production automation, real LLM work, Python/JS scripting, sync/reconciliation instincts, 6+ hrs US Eastern overlap) line up cleanly with the public portfolio: JOB-128 (tuinui marketing-data pipeline, Composio + FastAPI), JOB-130 (CRM chatbot with two-phase commit write safety), JOB-126 (Persistent Reasoning Engine, structured Claude workflows), and 20 years of real-time POS data-pipeline work.

## 2. Scope

### In scope

1. **Cross-stack data flow design** — accounting (QBO/Sage/NetSuite/Xero), payroll (Gusto/Rippling/Justworks), ops (field service platforms), CRM (HubSpot/Salesforce), reporting (Tableau/Looker/Power BI), project tools (Monday/Notion/Asana). Adapter-per-system plugin pattern.
2. **Integrations** — REST + webhooks + middleware (n8n/Make/Zapier) + custom Python/JS where connectors don't exist. OAuth + key auth + JWT. Rate limits, retries, exponential backoff, idempotency keys, dead-letter queues.
3. **AI layer** — Claude Code agents, structured-output workflows, tool-connected automations. Validation + human oversight = dependable, not demo-grade. Single-agent + multi-agent coordination where appropriate.
4. **ETL pipelines** — extract/clean/transform/sync. Source-deterministic record-id hashing for idempotency. Append-only audit log per record (source, fetched_at, idempotency_key).
5. **Workspace buildouts** — Monday.com + Notion via API + Claude connectors for the bulk operations (board templates, property setup, doc scaffolding).
6. **Reusable templates + playbooks** — every one-off build gets hardened into a template the next client inherits.

### Out of scope (per JD)

- Real-time streaming infra, custom data warehouse builds
- Custom CRM/ERP implementations
- Voice/phone integrations
- On-prem deployment (cloud-native by default)
- ML model fine-tuning (Claude API + prompt engineering only)

### Engagement phases (post-bid)

| Phase | Duration | Deliverable |
|---|---|---|
| W0 — Paid test project | 1 week | Hand a system spec, deliver ~80% of the spec working, then debrief |
| W1-W2 — Discovery + first integration | 40h | First cross-stack pipeline live for one client (accounting → reporting) |
| W3-W4 — AI layer | 40h | One Claude Code agent in production for one client workflow |
| W5-W6 — Template hardening | 40h | First template ready for cross-client reuse |
| W7+ — Ongoing | 30-40h/wk | New clients, new integrations, new agents |

## 3. Tech stack

| Concern | Choice | Why |
|---|---|---|
| Primary language | Python 3.11+ | Best LLM SDK support, async-friendly, mature integration libs |
| Secondary language | JavaScript / TypeScript | n8n custom nodes, browser-side scripts, Notion/Monday glue |
| LLM | Claude (Sonnet/Opus) via Anthropic SDK | Per JD requirement (Claude Code, agents, structured outputs) |
| Orchestration | LangGraph | State machine with interrupt points; matches JOB-130 pattern |
| Middleware | n8n (primary), Make/Zapier (when client mandates) | Per JD |
| Database | PostgreSQL | Idempotency keys, audit log, reconciliation state |
| Vector store | pgvector (optional) | RAG over client docs when needed |
| Validation | Pydantic v2 | Strict schema enforcement on every LLM output |
| Observability | loguru + OpenTelemetry + Langfuse (optional) | Per JOB-130 pattern |
| API client | httpx (async + sync) | Mature retry/backoff, OAuth flows |
| Testing | pytest | Real site fixtures + mock backends |
| Container | Docker + docker-compose | One-command boot for client demos |
| Deploy | Render / Railway / Fly.io (default) + client-VPS if required | Low-friction first deploy |

## 4. Architecture

```
                            [ Stack Growth Solutions owner ]
                                       |
                                       v
                          [ AI Integration & Automation Engineer ]
                          +-----------------+-----------------+
                          |                 |                 |
                          v                 v                 v
                  [ Claude Code ]   [ n8n / Make ]   [ Custom Python/JS ]
                  (agents, RAG,    (visual flows     (custom integrations,
                   tool calls)      for ops team)     complex auth, batch jobs)
                          |                 |                 |
                          +--------+--------+--------+-------+
                                   |                 |
                          [ API gateway ]    [ Validation + audit ]
                          (REST + webhooks)    (Pydantic + log)
                                   |
                +------------------+------------------+
                |                  |                  |
                v                  v                  v
        [ Accounting ]       [ CRM / Ops ]      [ Reporting ]
        (QBO/Sage/Xero/      (HubSpot/SF/       (Tableau/Looker/
         NetSuite)            Monday.com)         Power BI)
                |                  |                  |
                +--------+---------+--------+--------+
                         |                  |
                  [ Reconciliation ]  [ Idempotency keys ]
                  (drift detection,    (deterministic record IDs,
                   auto-retry,          re-run = no-op)
                   escalation)
```

Key principles:
- **CRM is the source of truth for customers; accounting is the source of truth for money.** Reconciliation detects drift, doesn't try to merge.
- **Every LLM output goes through Pydantic validation.** Bad LLM output → retry with stronger prompt, not silent acceptance.
- **Every action has an audit row.** source, timestamp, idempotency_key, before/after.
- **Two-phase commit for any write that affects money.** agent proposes → human confirms → tool executes. No exceptions.
- **Templates are first-class artifacts.** Each one-off build is born with a "extract template" task in W2.

## 5. Proof of fit — prior engagements

The 5 mandatory questions in the JD require specific examples. Here's what each maps to in the public portfolio:

| JD requirement | Public evidence |
|---|---|
| Most complex end-to-end integration | JOB-128 (tuinui marketing-data pipeline): Slack + GA4 + Google Ads unified ingestion via Composio SDK + FastAPI webhook fan-out. github.com/9KMan/JOB-20260702144531-000128 |
| Integration with no native connector | JOB-128 again: built a custom Slack Events API webhook adapter before the official connector existed. Also: 20-yr real-time POS platform connecting SAP/Oracle Retail/IBM AS/400 to 3,000+ terminals via custom middleware |
| Most advanced non-chatbot LLM system | JOB-126 (Persistent Reasoning Engine): structured Claude workflows with memory + tool calls + audit trail. github.com/9KMan/JOB-20260701155102-000126 |
| Two-system source-of-truth reconciliation | JOB-130 (Composio CRM chatbot): two-phase commit write safety + drift detection. github.com/9KMan/JOB-20260705085302-000130 |
| Hourly rate / availability / US Eastern overlap | $40/hr top-of-band, 30-40h/wk, Bangkok TZ = US Eastern evening overlap (8-14 BKK = 19-01 EST, 6h) |

## 6. Engagement shape

- **Application** — this document + cover letter (starts with "PIPELINE")
- **Live portfolio walkthrough** — scheduled call, walk through JOB-128/130/126 architecture, answer technical Q's
- **Paid test project** — 1 week, given a system spec, deliver ~80% of spec working, then debrief
- **Ongoing engagement** — >6 months, complex/ongoing, 30-40h/wk

## 7. Risk + open questions for kickoff

### Risks (with mitigations)

1. **TZ overlap** — Bangkok = US Eastern + 12h. Working 8-14 BKK = 19-01 EST = 6h US Eastern coverage but at non-standard hours. Honest disclosure in cover letter. Mitigation: async-first (default for the JD), with daily 1-hour sync window at 21:00 EST = 09:00 BKK next morning.
2. **n8n tooling depth** — have used similar middleware patterns (MuleSoft, Apache Camel, custom Python) but not n8n specifically. 1-week paid test project is the calibration window.
3. **Finance domain context** — JD explicitly says "finance or accounting context is a plus, not a requirement." Disclosed: prior work is retail/operations, not PE/finance. Architecture skills transfer; domain knowledge will ramp in W1.

### Open questions for kickoff (5 blocker items, same as the JD's mandatory questions, restated)

1. Which 2-3 client stacks are highest priority for the W1-W4 deliverables?
2. What's the team's existing tooling — is there an n8n instance already, or do we set one up?
3. Which Claude model tier is the default — Sonnet for cost, Opus for quality?
4. What's the human-in-the-loop approval UX preference — Slack approval buttons, email digest, admin console?
5. What's the disaster-recovery SLA for the integration platform (e.g., 1h downtime acceptable)?

---

**Status**: INTAKE → awaiting CEO approval → build pipeline.
**JOB_ID**: JOB-20260705142000-000132
**Tier**: EXPERT (qualifier score 62/100 → REVIEW → BID manual override)
**Rate**: $40/hr top-of-band (client open to higher)
**GitHub repo**: https://github.com/9KMan/JOB-20260705142000-000132 (created on cto-approve)