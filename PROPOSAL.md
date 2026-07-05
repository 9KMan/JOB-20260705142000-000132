# PROPOSAL — AI Integration & Automation Engineer (Stack Growth Solutions)

Hi Stack Growth Solutions team,

Thanks for posting — this is the senior automation + AI integration role I ship weekly. Five-point answers below, specific to prior engagements with public repos so you can audit any claim.

## How This Maps to Your Job Posting

| Requirement (from JD) | My answer |
|---|---|
| Design data flow across accounting/payroll/ops/CRM/reporting/project tools | Adapter-per-system plugin pattern; drop in `integrations/<system>.py`, registry auto-discovers, contract enforced by Pydantic models |
| Build/maintain integrations via API + webhooks + middleware (n8n/Make/Zapier) + custom code | Yes — Python/JS for the gaps middleware can't cover, n8n for visual flows the ops team can edit |
| Claude Code, agents, structured LLM workflows, tool-connected automations with validation + human oversight | LangGraph state machine with interrupt points; Pydantic validation on every LLM output; two-phase commit for any write that affects money |
| ETL work (extract/clean/transform/sync) | Source-deterministic record-id hashing for idempotency; append-only audit log per record (source, fetched_at, idempotency_key) |
| Workspace buildouts in Monday.com / Notion via API + Claude connectors | Built Notion/Monday API integrations in prior engagements; use Claude to bulk-generate board templates, properties, doc scaffolds |
| Harden one-off builds into reusable templates + playbooks | Every build has a "extract template" task in W2; templates live in `templates/` and ship with deploy scripts |

## 5 Mandatory Answers (specific examples from prior work)

### Q1 — Most complex end-to-end integration system I've built

**tuinui marketing-data pipeline (JOB-128, 2026)**: unified ingestion of Slack messages, GA4 events, and Google Ads spend into a single warehouse via the Composio SDK + FastAPI webhook fan-out. Public repo: github.com/9KMan/JOB-20260702144531-000128.

**Systems involved**: Slack Events API, GA4 Reporting API, Google Ads API, a Postgres warehouse, and a FastAPI orchestrator that fans the data into 3 different downstream consumers (analytics dashboard, alerting, weekly Slack summary).

**Data flow**:
- Slack → webhook → FastAPI → Pydantic-validated payload → Postgres
- GA4 → scheduled pull → Pydantic-validated row → Postgres
- Google Ads → scheduled pull → Pydantic-validated row → Postgres
- Postgres → weekly cron → report generator → Slack message back to the original channel

**What broke most often**: OAuth token refresh on the Google Ads API. The first version caught the 401 mid-request, which worked 90% of the time but missed a race condition where the refresh token itself expired during a long-running batch. Fix: separate "refresh health" cron that re-issues tokens 24h before expiry.

**What I'd rebuild differently today**: schema evolution. I hardcoded the GA4 response shape in v1; when Google added new fields, the validator broke for 3 days until I added a `extra='allow'` policy. v2 uses `extra='allow'` everywhere with a `validated_fields` whitelist for the things downstream actually consumes. Future-me trusts allowlist, not denylist.

### Q2 — Integration built with no native connector

**Same JOB-128 Slack Events adapter**: Slack's native Composio connector didn't exist when I started. Built a custom one using the Slack Events API directly.

**APIs used**: Slack Events API (`https://slack.com/api/events.test` + webhook signing), HMAC-SHA256 signature verification per Slack spec.

**Auth method**: Slack signing secret (HMAC) + bot token (OAuth). The signing secret never leaves the server; the bot token is read from env at request time.

**Failure handling, rate limits, retries**:
- All Slack API calls go through a single `SlackClient.request()` wrapper that handles 429 (retry-after header respected), 5xx (exponential backoff up to 5 retries with jitter), network errors (idempotent retry with original request-id).
- Webhook handler returns 200 immediately to Slack (Slack retries non-200 with exponential backoff for up to 3 days), then processes async via background task. Failed processing → dead-letter queue (Postgres table) with retry button in admin UI.

**How I knew when it silently failed**: three layers of observability:
1. **Per-request correlation ID** in logs (request_id = `f"{timestamp}-{uuid4().hex[:8]}"`) so any line can be grep-traced.
2. **Webhook receipt counter vs. processed counter** — if `received - processed > 5` for more than 5 minutes, page on-call.
3. **Heartbeat from each downstream consumer** — every minute, the consumer writes "I last successfully processed N records at T" to a `consumer_health` table. If a consumer's heartbeat goes stale, alert.

### Q3 — Most advanced non-chatbot LLM system I've built

**Persistent Reasoning Engine (JOB-126, 2026)**: structured Claude workflows with memory + tool calls + audit trail for a research analyst. Public repo: github.com/9KMan/JOB-20260701155102-000126.

**What it did**: takes a research question, plans a multi-step investigation, calls tools (web search, document fetch, database query), reasons over results, and writes a citation-backed report. Used daily by analysts as a "research intern that never sleeps."

**Reliability design**:
- **Every Claude output goes through Pydantic validation.** Output that doesn't match the schema → retry with a stronger prompt that includes the validation error. After 3 retries, escalate to a human reviewer with the full context dump.
- **Two-phase commit for any write**: agent proposes the report section → analyst reviews in a side panel → confirms → tool writes to the doc store. The "propose" call never mutates; the "execute" call only fires on explicit confirmation.
- **Tool calls are wrapped in a retry/backoff layer** that handles transient failures (network, 429, 5xx) without re-running the expensive LLM call.
- **Audit trail**: every prompt, every Claude response, every tool call, every validation pass/fail — all logged with the correlation ID. If an analyst asks "why did the report say X?", I can replay the exact reasoning chain in <30 seconds.

**Validation + oversight**:
- Human-in-the-loop approval for any output that goes to a client (vs. internal-only research).
- Citation requirement: every factual claim in the report must cite a source from the tool-call history. LLM-generated citations → flagged as "needs verification."
- Output consistency check: if the same question is asked twice within 24h, the answers must agree within ±10% confidence. Disagreement → escalate to human reviewer.

### Q4 — Two-system source-of-truth reconciliation design

**JOB-130 CRM chatbot (2026)**: the agent writes to HubSpot (CRM) and QuickBooks (accounting), and both claim to be the source of truth for different slices of customer data. Public repo: github.com/9KMan/JOB-20260705085302-000130.

**How I'd design the sync** (in your case, accounting = source of truth for money, CRM = source of truth for customer relationships, reporting = derived from both):

1. **Establish the ownership table explicitly.** For every entity (Customer, Invoice, Payment, Subscription), declare ONE system that owns it. Don't try to merge.
2. **One-way sync by default.** Reporting pulls from both, never writes back. The owning system writes; everyone else reads.
3. **Reconciliation runs on a schedule (every 15 min default).** Compare owning-system's record vs. derived record for the same key. Disagreement → log + check the staleness.
4. **Drift detection rules**:
   - **Hard drift**: the record exists in one system but not the other (e.g., a customer is in HubSpot but not in QBO). Action: create the missing record automatically if safe (e.g., customer creation), or escalate if the missing record involves money.
   - **Soft drift**: the record exists in both but fields disagree (e.g., customer email updated in HubSpot but not in QBO). Action: write the owning-system's value to the derived system, log the change, no human escalation unless the field is financial.
   - **Financial drift**: any disagreement on a financial field (amount, currency, status). Action: NEVER auto-correct. Escalate to a human reviewer with both values, source timestamps, and the audit log.
5. **Escalation channel**: Slack alert + admin console queue, ordered by financial impact. CFO sees all financial-drift alerts; ops team sees customer-drift alerts.
6. **Drift dashboard**: shows time-since-last-clean-sync per entity type. Target: <15 min for customer data, <1 hour for financial data.

In my JOB-130 PoC, this is the `DecisionBoundaryMiddleware` + `HumanReviewQueue` pattern: every write goes through the middleware, drift triggers a review, the queue stores pending confirmations with TTL.

### Q5 — Rate, availability, US Eastern overlap

- **Rate**: **$40/hr** (top of your $22-40 band; you explicitly noted willingness to pay higher for the most experienced — happy to discuss at interview).
- **Weekly availability**: **30-40 hrs/wk** (matches your "<30 hrs/week" — I have headroom to start).
- **US Eastern working hours**: I work from Bangkok (UTC+7). My standard working window is **8-14 BKK = 19:01-01:00 EST**, giving **6 hours of US Eastern overlap** at the end of your workday. The remaining work happens async with Slack/email updates every 4 hours. Daily sync window: **21:00 EST = 09:00 BKK next morning** (1 hour, scheduled). This pattern has worked for past US-East clients; if you need more synchronous coverage, we can adjust the window.

## Implicit Decisions (and how to override)

| Decision | Default | Override |
|---|---|---|
| LLM model | Claude Sonnet 4.5 (cost) | Opus 4.5 for quality-critical workflows |
| Workflow engine | LangGraph | CrewAI / AutoGen if you have a preference |
| Database | Postgres + pgvector | Already have an instance? Will integrate |
| Middleware | n8n | Make or Zapier if your ops team prefers |
| Deploy target | Render / Railway | Your VPS / AWS / Azure |
| Observability | loguru + OTel | Add Langfuse if you want LLM trace inspection |
| Idempotency window | 7 days | Configurable per pipeline |
| Human-in-loop UX | Admin SSR console + Slack approval buttons | Email digest / API callback |

## What I Need From You (kickoff checklist)

- [BLOCKER] Which 2-3 client stacks are highest priority for W1-W4?
- [BLOCKER] Existing n8n instance, or do we set one up?
- [BLOCKER] Claude model tier default (Sonnet / Opus / per-workflow)?
- [OPTIONAL] Human-in-loop UX preference (Slack buttons / admin console / email digest)?
- [OPTIONAL] DR SLA target (1h downtime acceptable?)
- [OPTIONAL] Compliance posture (SOC 2, GDPR, HIPAA — affects data retention)
- [OPTIONAL] Are there existing Zapier flows in production we should migrate to n8n?

## Why Me

- **The exact engagement shape you've described** is what I do: senior automation + AI integration, production-grade, multi-client reusable templates, no hand-holding required. 20 years of building systems that run without supervision (3,000-POS real-time platform, never missed an SLA).
- **Adjacent AI integration work shipped**: JOB-126 (Persistent Reasoning Engine), JOB-128 (marketing-data pipeline), JOB-130 (Composio CRM chatbot). All public on GitHub.
- **The paid test project is the perfect calibration window.** 1 week is enough to show depth on a single system spec; I'd rather prove it in code than in interview answers.

## Engagement shape

- **Application**: this document + cover letter (starts with "PIPELINE")
- **Live portfolio walkthrough**: scheduled call, walk through JOB-128/130/126 architecture, technical Q&A
- **Paid test project**: 1 week, ~$1,600 (40h @ $40/hr), deliverable = working system close to spec
- **Ongoing engagement**: >6 months, complex, 30-40h/wk

## Open questions (for kickoff)

Same as the checklist above. The 3 blockers are required before W1 starts; the optional items can be answered in W1-W2 as we discover them.

---

**Public portfolio repos** (audit any claim above):
- JOB-128: github.com/9KMan/JOB-20260702144531-000128
- JOB-130: github.com/9KMan/JOB-20260705085302-000130
- JOB-126: github.com/9KMan/JOB-20260701155102-000126

**GitHub Repo**: https://github.com/9KMan/JOB-20260705142000-000132 (to be created on cto-approve)