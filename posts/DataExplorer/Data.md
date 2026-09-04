# Data Explorer

An open-source-first enterprise RAG platform for governed answers, business
documents, and data-driven presentations. The build includes FastAPI, LangGraph,
tenant-aware dense/sparse retrieval, citations, governed DOCX/PPTX publishing,
Ollama/OpenAI/Anthropic adapters, JWT authentication, layered guardrails, and
audited Text2SQL with independent human approval.

## Quick start

Prerequisites: Python 3.11+ and a running Ollama instance.

```powershell
uv sync
Copy-Item .env.example .env
ollama pull llama3.1:8b
ollama pull embeddinggemma
uv run dataexplorer
```

The API is available at `http://127.0.0.1:8000`; OpenAPI is at `/docs`.

Start the enterprise workspace in another terminal:

```powershell
uv run dataexplorer-ui
```

The workspace client is available at `http://127.0.0.1:8501` and includes
governed chat, knowledge ingestion, document/deck publishing, Text2SQL approvals,
and evaluation. Start `uv run dataexplorer-admin` for the separate observability
console at `http://127.0.0.1:8502`. It requires an `observability-admins` or
`platform-admins` entitlement. `docker compose up` starts both UIs, the API,
Ollama, PostgreSQL, Redis, and Qdrant.

User, tenant, and group claims are resolved by the API and are not editable in
the workspace client. In development only, environment variables provide the
identity headers; staging and production require validated JWT claims.

LLM telemetry stores correlation ID, user, tenant, operation, provider, model,
tokens, latency, grounding, retries, and estimated API cost—never prompt,
response, retrieved passage, or document bodies. Provider/model rates come from
the `DATAEXPLORER_LLM_*_COST_PER_MILLION_USD` JSON maps. Missing rates are shown
as unpriced rather than assigned an invented cost.

Ingest a development document:

```powershell
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/v1/documents `
  -Headers @{"X-User-ID"="analyst-1"; "X-Tenant-ID"="acme"; "X-Groups"="finance"} `
  -ContentType application/json `
  -Body '{"document_id":"policy-1","title":"Travel policy","text":"Employees may claim approved rail travel.","allowed_groups":["finance"]}'
```

Query it:

```powershell
Invoke-RestMethod -Method Post -Uri http://127.0.0.1:8000/v1/query `
  -Headers @{"X-User-ID"="analyst-1"; "X-Tenant-ID"="acme"; "X-Groups"="finance"} `
  -ContentType application/json `
  -Body '{"question":"What travel can employees claim?"}'
```

Run the isolated test suite with `uv run pytest`. Header-based identity is
development-only. Staging and production configuration requires OIDC/JWT and
rejects development identity headers.

## Evaluation dataset

The reproducible [SEC Enterprise RAG Test Pack](datasets/sec_enterprise/README.md)
combines Microsoft, Walmart, and Delta filings/XBRL with synthetic policies,
stale versions, duplicate content, tenant boundaries, restricted test records,
and prompt-injection fixtures. Generate the safe offline corpus with
`uv run python scripts/build_sec_enterprise_dataset.py --offline`, then ingest it
with `uv run python scripts/ingest_sec_enterprise_dataset.py` after Ollama is ready.
Live SEC downloads require a declared `DATAEXPLORER_SEC_USER_AGENT` containing
your organization and operational contact, as required by SEC fair-access rules.

Select a model route with `DATAEXPLORER_MODEL_PROVIDER=ollama|openai|anthropic|router`.
Managed providers are limited to public/internal content; confidential and
restricted content fails closed unless an eligible private/local route exists.
The OpenAI adapter uses Responses with storage disabled, and Claude generation
delegates embeddings to the configured embedding provider.

Publishing is a three-step API workflow: create `/v1/artifacts`, obtain an
independent decision, then render. Artifact specifications require a source ID
for every factual section, constrain filenames, retain approval metadata, and
hash outputs. DOCX rendering is built in. PPTX rendering is enabled only when
the configured Node and artifact-tool runtime paths are present.

## Implemented security boundaries

- Tenant and group authorization is applied before evidence reaches a model.
- Hybrid dense/sparse retrieval uses RRF, absolute relevance, trust weighting,
  expiry checks, and a configurable lexical or FlashRank reranker.
- High-confidence document/query injection patterns are blocked, model output is
  citation-checked, and common PII and secrets are redacted.
- Staging and production require JWT issuer, audience, and JWKS settings. Request
  rate and token budgets use a replaceable policy component.
- SQL follows proposal, independent approval, and execution stages. SQLGlot AST
  validation permits one bounded `SELECT` over allowlisted tables, columns, and
  functions. The PostgreSQL adapter opens a read-only transaction, applies a
  timeout and tenant session context, checks plan cost, and caps rows.

## Production deployment

Development defaults remain in memory. Production configuration refuses to
start unless PostgreSQL persistence, Redis policy enforcement, Qdrant retrieval,
JWT identity, and their required connection settings are configured. Qdrant
uses dense/sparse named vectors with server-side RRF, tenant/ACL prefilters, and
a defense-in-depth authorization check before evidence reaches a model. Redis
enforces request and daily-token limits through one atomic script. PostgreSQL
stores audit events, SQL approvals, and artifact approvals; apply
`migrations/001_governance.sql` through the included worker job.

The Terraform module under `infra/terraform` provisions private networking,
separate Cloud Run API/UI services and a worker job, Artifact Registry, versioned GCS, private Cloud SQL,
TLS-enabled Memorystore, KMS, Secret Manager, monitoring, and budgets. Before
the first deployment, create the remote-state bucket, copy
`terraform.tfvars.example`, and add versions for the generated database, OpenAI,
and Qdrant secrets. Cloud Build deploys immutable commit-tagged images; GitHub
CI runs tests, container builds, Terraform formatting, and validation.

The API emits structured request logs, propagates correlation IDs, and exposes
Prometheus-format counters and latency sums at `/metrics`. No prompt, evidence,
or restricted document content is placed in those telemetry records.
