# Stack Research — Integration + AI Platform

## Stack selection rationale

### Primary language: Python 3.11+

**Why**: Best LLM SDK support (Anthropic SDK first-class), mature async I/O (asyncio,
httpx), mature integration libraries (httpx for OAuth/webhooks/retry, pydantic for
schema validation, sqlalchemy for DB).

**Tradeoff**: Python is slower than Go/Rust for high-throughput pipelines. Mitigated
by async + Postgres checkpointing for state, and by horizontal scaling (each
integration worker is a separate process).

### Secondary: JavaScript / TypeScript

**Why**: n8n custom nodes are written in JS; Notion/Monday SDKs are JS-first;
browser-side scripts for workspace buildouts.

**Tradeoff**: Two-language codebase requires more careful contract definition
between Python (orchestration) and JS (custom nodes). Mitigated by Pydantic
schemas as the contract source of truth.

### LLM: Claude (Sonnet 4.5 + Opus 4.5) via Anthropic SDK

**Why**: Per JD requirement (Claude Code, agents, structured outputs). Claude has
the best track record on long-context reasoning + tool calls + structured outputs
(per Anthropic's evals).

**Model tier**:
- **Sonnet 4.5**: default for high-volume integration work (cost-effective)
- **Opus 4.5**: quality-critical workflows (CFO-facing reports, financial reconciliation)

### Orchestration: LangGraph

**Why**: Native interrupt points for human-in-loop (matches "validation and human
oversight so they are dependable" from the JD). First-class Postgres-backed
checkpointing for workflow memory. OpenTelemetry tracing on every node.

**Tradeoff**: Newer framework (vs. CrewAI / AutoGen). Mitigated by prior use in
JOB-126 + JOB-130 (proven pattern).

**Alternatives considered**:
- **CrewAI**: no native interrupt primitive — can't do clean human-in-loop
- **AutoGen**: similar limitations + heavier abstraction
- **Custom loop**: reinvents state machine + persistence — no upside for this scope

### Middleware: n8n (primary), Make/Zapier (when client mandates)

**Why**: Per JD requirement. n8n has the strongest custom-node extensibility (JS)
and the best self-host story for client-data-residency.

**Tradeoff**: No prior n8n experience. Mitigated by deep familiarity with MuleSoft
+ Apache Camel (similar middleware patterns) + the paid test project as calibration.

**Alternatives**:
- **Make.com**: visual-first, less code-extensible
- **Zapier**: limited self-host, higher cost at scale

### Database: PostgreSQL + pgvector (optional)

**Why**: Idempotency keys, audit log, reconciliation state, conversation memory,
optional RAG over client docs.

**Tradeoff**: Postgres is overkill for simple pipelines. Mitigated by SQLite option
for single-client engagements, with Postgres when scaling beyond ~100 records/min.

### Validation: Pydantic v2

**Why**: Strict schema enforcement on every LLM output. Pydantic v2 is 5-50x faster
than v1 and is the de-facto standard for Python API contracts.

**Tradeoff**: Adds a layer of indirection (need to define models). Mitigated by
generating models from OpenAPI specs when integrating with third-party APIs.

### Observability: loguru + OpenTelemetry + Langfuse (optional)

**Why**: loguru for structured logging with `SecretsFilter` (automatic redaction of
api_key/token/secret/password/webhook). OpenTelemetry for distributed tracing.
Langfuse for LLM-specific trace inspection (prompt/response/tool-call replay).

**Tradeoff**: Three tools to wire up. Mitigated by the JOB-130 pattern (already
production-tested).

### API client: httpx (async + sync)

**Why**: Mature retry/backoff, OAuth flows, async + sync in one library, good
timeout handling.

**Tradeoff**: No built-in rate-limit awareness (need to wrap). Mitigated by the
`request_with_rate_limit` wrapper from JOB-128.

### Testing: pytest

**Why**: De-facto standard for Python. Real site fixtures + mock backends.

**Tradeoff**: No built-in async test support (need pytest-asyncio). Mitigated by
config in `pyproject.toml`.

### Container: Docker + docker-compose

**Why**: One-command boot for client demos. Reproducible across environments.

**Tradeoff**: None significant for this scope.

### Deploy: Render / Railway / Fly.io (default) + client-VPS if required

**Why**: Low-friction first deploy. Auto-SSL + auto-deploy from GitHub. Cost:
~$7-25/mo for a small integration worker.

**Tradeoff**: Vendor lock-in. Mitigated by Docker portability — can move to
client-VPS / AWS / Azure without code changes.

## Stack diagram

```
[ Stack Growth Solutions owner ]
              |
              v
[ AI Integration & Automation Engineer ]
              |
       +------+------+------+
       |      |      |      |
       v      v      v      v
   [Python] [n8n]  [JS]   [Claude SDK]
   (custom  (visual (custom (Sonnet/Opus
    glue)   flows)  nodes)  via Anthropic)
       |      |      |      |
       +------+------+------+
              |
       [LangGraph orchestrator]
              |
       [Pydantic validation + audit log]
              |
       [Postgres + pgvector]
              |
  +-----+-----+-----+-----+
  |     |     |     |     |
  v     v     v     v     v
[Acc] [Pay] [CRM] [Ops] [Reports]
```

## What we don't use (and why)

- **Terraform / Pulumi**: out of scope for the engagement (client infra is theirs)
- **Kubernetes**: Render/Railway handles scale; k8s only if client mandates
- **Ray / Dask**: not needed for the integration workload (Postgres + asyncio handles it)
- **Hugging Face transformers**: Claude API is sufficient (per JD's Claude-Code focus)
- **Vector DB (Qdrant, Weaviate)**: pgvector is sufficient for ≤1M docs per client

## Key references

- **JOB-128**: github.com/9KMan/JOB-20260702144531-000128 (marketing-data pipeline, Composio + FastAPI + Slack + GA4 + Google Ads)
- **JOB-130**: github.com/9KMan/JOB-20260705085302-000130 (CRM chatbot, two-phase commit)
- **JOB-126**: github.com/9KMan/JOB-20260701155102-000126 (Persistent Reasoning Engine)
- **JOB-127**: github.com/9KMan/JOB-20260701230408-000127 (production-platform reference scaffold)

## Status

RESEARCH COMPLETE — stack selection locked in. Implementation reference scaffold in `app/`.