<!--
Sync Impact Report (1.0.4, 2026-08-08):
- Version change: 1.0.3 → 1.0.4 (PATCH: additive observability pin; no principle weakened, removed,
  or reinterpreted).
- Added: LLM-call observability. Section 20 mandates eval thresholds (tool-selection accuracy,
  unsupported-action rate, conflict-identification accuracy) that span/metric/log telemetry cannot
  evidence — nothing captured what the model proposed or why it was refused. Model calls are now
  recorded as OTel spans with token counts, cost, proposed option labels, and Policy Engine verdicts,
  with capture of prompts and completions gated behind the redaction filter.
- A dedicated LLM-observability tool (e.g. Langfuse) is permitted only self-hosted inside the account
  boundary and collector-fed; it is NOT an agent framework and is not covered by the 1.0.3
  prohibition. A vendor SDK in application code remains forbidden by the OTLP-only rule.
- Governance note: AGENTS.md Section 15 was updated first; this file follows.

Sync Impact Report (1.0.3, 2026-08-08):
- Version change: 1.0.2 → 1.0.3 (PATCH: additive stack pins; no principle weakened, removed, or
  reinterpreted).
- Added: "no agent framework" as an explicit prohibition. Previously implicit — the stack named the
  MCP SDK, Bedrock, and a hand-written state machine but never forbade a framework, so an
  implementer could have added one without breaking any written rule. A framework that owns the
  agent loop would make in-handler authorization and the single-LLM-component boundary unauditable.
- Added: telemetry pipeline. OpenTelemetry and CloudWatch were both named, but CloudWatch does not
  ingest OTLP traces, so the two did not compose into a working pipeline. Now pinned: OTLP-only from
  application code, ADOT collector sidecar, X-Ray for traces, CloudWatch for logs and metrics,
  Jaeger locally, and mandatory sampling of any trace rooted at a `disruption` event.
- Governance note: AGENTS.md Sections 15, 17, and 19 were updated first; this file follows.

Sync Impact Report (1.0.2, 2026-08-08):
- Version change: 1.0.1 → 1.0.2 (PATCH: contract-path relocation; no principle weakened, removed,
  or reinterpreted).
- Contract path moved from `specs/001-stillon-core/contracts/openapi.yaml` to the stable,
  feature-independent `packages/contracts/openapi.yaml`. Rationale: a per-feature contract path
  forces the codegen configuration to be edited whenever a numbered feature is added, which makes
  the generated-code CI gate fragile. `packages/contracts/` was already the MCP-schema SSOT, so this
  consolidates every contract behind one path codegen reads and never has to relearn.
- Supersedes the 1.0.0 note "Contract path: AGENTS.md pins specs/001-stillon-core/contracts/..." —
  that constraint no longer applies. Contracts MUST NEVER live under `specs/<feature>/`; if Spec Kit
  scaffolds one, it is a scratch staging area and the authoritative file lands at the stable path.
  `docs/contracts/openapi.yaml` remains NOT adopted: an OpenAPI spec that generates code is a build
  input, not documentation.
- Templates requiring updates: `.specify/templates/plan-template.md` — its file-tree section MUST
  name `packages/contracts/openapi.yaml` and `packages/generated/`, NOT a per-feature contracts dir.
  This supersedes the 1.0.0 template note below.
- Governance note: AGENTS.md Sections 12, 16, 17, and 18 were corrected first; this file follows.

Sync Impact Report (1.0.1, 2026-08-08):
- Version change: 1.0.0 → 1.0.1 (PATCH: clarifications and additive stack pins; no principle
  weakened, removed, or reinterpreted).
- Reconciled two clauses that had drifted from AGENTS.md after its rounds 4-6 revisions:
  - Deployment: `/readiness` no longer includes partner MCP reachability; partner health moves to a
    separate `/dependencies` endpoint that MUST NEVER fail readiness (AGENTS.md Section 22).
  - Authorization: "chain version check" replaced by the entity-appropriate version check —
    `workflow_version` for approvals and workflow transitions, `chain_version` for chain mutations
    (AGENTS.md Section 9).
- Added: Build and test toolchain pins mirroring AGENTS.md Section 17 (uv, pytest + pytest-cov,
  asyncio, alembic, Node LTS + pnpm, GitHub Actions, slim non-root multi-stage containers).
- Governance note: both reconciliations follow the amendment rule — AGENTS.md was corrected first,
  and this file was brought into agreement with it.

Sync Impact Report (1.0.0, 2026-08-07):
- Version change: TEMPLATE (unfilled) → 1.0.0
- Rationale: Initial ratification. The file previously contained only placeholder tokens; this is
  the first concrete constitution, seeded from AGENTS.md Section 2 principles plus the pinned
  constraints in AGENTS.md Sections 8-15, with cross-project standards adopted where compatible.
- Principles Added (9):
  - I. Human-Governed Autonomy (NON-NEGOTIABLE)
  - II. Deterministic Authority
  - III. Contract-First / API-First Architecture
  - IV. Verified Execution & Transaction Safety
  - V. Comprehensive Auditability (NON-NEGOTIABLE)
  - VI. Security, Privacy & Prompt-Injection Defense
  - VII. Rigorous Quality Assurance
  - VIII. Spec-Driven Delivery & Scope Integrity
  - IX. Reproducible, Containerized Deployment
- Sections Added:
  - Architecture & Lifecycle Constraints (replaces [SECTION_2_NAME])
  - Development Standards & Quality Gates (replaces [SECTION_3_NAME])
  - Governance (filled)
- Sections Removed: none (all template placeholders resolved)
- Cross-project standards adopted from the Test-ify constitution: centralized contract SSOT with
  CLI codegen, zero-hardcoded-secrets, 401 → login redirect, >=80% unit coverage, mandatory
  CHANGELOG.md, centralized docs/, containerized-deployment deliverables, amendment-by-PR
  governance with a Sync Impact Report.
- Cross-project standards deliberately NOT adopted (conflict with AGENTS.md pins, which win):
  - Contract path: AGENTS.md pins `specs/001-stillon-core/contracts/openapi.yaml` +
    `packages/contracts/`, NOT `docs/contracts/openapi.yaml`.
  - Auth mechanism: AGENTS.md pins OIDC/OAuth for guests and OAuth 2.1 + PKCE (S256) for the
    public Assistant MCP surface, NOT JWT-in-localStorage.
  - "All code MUST use relative imports": omitted; incompatible with the pinned Python 3.12
    package layout. Replaced by the single-source-of-truth import rule in Principle III.
- Templates requiring updates:
  - .specify/templates/plan-template.md (pending: ensure the file-tree section names
    `specs/[###-feature]/contracts/openapi.yaml` and `packages/generated/` explicitly)
- Deferred items: none. No TODO placeholders remain.
-->
# StillOn (powered by Maestro) Constitution

## Core Principles

### I. Human-Governed Autonomy (NON-NEGOTIABLE)
**The agent proposes; the authorized human disposes.**
No reservation change, cancellation, exchange, or charge MAY occur without explicit customer
approval unless the Autonomy Policy pre-authorizes it. The three tiers are binding:
- **AUTONOMOUS** — monitor, calculate impact, search availability, place reversible holds, and
  shift an existing reservation by <= 60 minutes **only when ALL hold**: the guest explicitly
  pre-authorized it, the original merchant is retained, party composition is unchanged, cost delta
  is exactly $0.00, and all hard constraints pass. Execute, verify by read-back, then notify.
- **PROPOSE-AND-WAIT** — substitute/new merchant, any non-zero cost delta, removing or replacing an
  activity, changing party composition, or any option outside stored autonomy preferences. No
  partner mutation MAY occur before an approval is recorded.
- **NEVER AUTONOMOUS** — cancelling a confirmed booking, accepting a nonrefundable commitment,
  sharing PII with a new partner, initiating payment or refund movement, overriding a hard
  constraint. The agent MAY only prepare and request.

Approvals are recorded **only** by `services/api`, authenticated, ownership-checked, and
version-checked. The Notification MCP MUST NEVER record or fabricate an approval. Consent is
captured upstream: without an explicit opt-in record the monitor MUST NOT monitor and the
orchestrator MUST NOT act. Rationale: authority is the product boundary; if it can be bypassed by
any channel, nothing else in this document matters.

### II. Deterministic Authority
**The LLM converses; policy disposes.**
- `services/orchestrator` (Maestro) is the **only** component that MAY call an LLM. The Disruption
  Monitor, Chain Graph Service, and Policy Engine MUST contain no LLM call.
- `packages/policy-engine` MUST be invoked server-side, in-process, inside **every** mutation
  handler before the partner is contacted. It MUST NOT be exposed as a model-callable MCP tool.
  Rationale: a tool the model must remember to call can be omitted; an in-handler library call
  cannot.
- Every LLM-generated recovery plan MUST pass deterministic validation for schedule feasibility,
  availability, cost, policy, and customer constraints before any execution. An option violating a
  hard constraint scores `null`, is discarded before ranking, and MUST NEVER be shown to the guest.
- Detection, scoring, tie-breaking, and authorization MUST be reproducible: identical inputs
  produce identical verdicts. Tests MUST use the deterministic fake model provider and fixed mock
  seeds; no test outcome may depend on live model output.
- A refusal MUST be logged with the `rule_id` that produced it and surfaced as an option
  constraint. The same action MUST NEVER be retried under a different framing.

### III. Contract-First / API-First Architecture
**The schema is the single source of truth; code is generated from it, never the reverse.**
- `packages/contracts/openapi.yaml` is the SSOT for the REST surface and MUST be
  authored and reviewed **before any implementation code** for that surface exists.
- `packages/contracts/` is the SSOT for all contracts — the REST `openapi.yaml` and every MCP tool
  JSON Schema — at a stable, feature-independent path. Contracts MUST NEVER live under
  `specs/<feature>/`, and codegen configuration MUST NOT require editing when a feature is added.
  Servers, services, and tests
  MUST import from it. **A schema defined twice is a defect.**
- Backend Pydantic models MUST be generated from `packages/contracts/openapi.yaml` with
  datamodel-code-generator; the
  frontend TypeScript client MUST be generated with OpenAPI Generator (`typescript-fetch`).
  Generator versions and configuration MUST be pinned in `plan.md`.
- Generated output lives under `packages/generated/` and MUST NEVER be hand-edited. `make
  codegen-check` MUST fail CI when regeneration produces an uncommitted diff.
- All partner integrations MUST use versioned, schema-validated contracts. Every MCP tool MUST
  declare input schema, output schema, typed error model, timeout, retry policy, and idempotency
  behavior. Every mock MUST be swappable for a live adapter **by configuration alone**, with no
  orchestrator code change.
- To change behavior, edit the contract first, regenerate, then implement. A PR that edits
  implementation code ahead of its governing contract MUST be rejected.

### IV. Verified Execution & Transaction Safety
**Nothing is successful until the partner says so.**
- A partner action MUST NEVER be reported as successful until the resulting partner state has been
  confirmed by read-back (`get_reservation`, `get_ticket_order`, `get_pickup_status`).
- Every partner mutation MUST be idempotent, with the key derived from
  `(plan_id, activity_id, action_type)` so a retry reuses it. Replaying a key MUST return the
  original result and make no second partner-side change. Partner-side duplicates MUST be 0.
- Every mutation that **allocates new partner inventory** MUST consume an active, unexpired hold
  (`HOLD_REQUIRED` / `HOLD_EXPIRED` otherwise). Cancellations and hold releases require no hold but
  still require policy authorization, an idempotency key, and read-back.
- Timeouts and bounded retries (<= 2, exponential backoff with full jitter, only on timeout or 5xx),
  a per-partner circuit breaker, optimistic concurrency (`STALE_VERSION` on mismatch), and saga
  execution with a compensating action registered **before** each step are mandatory.
- After any ambiguous timeout the system MUST reconcile by read-back **before** any retry or
  compensation. A blind compensating mutation MUST NEVER be issued.
- Partial execution MUST NEVER end silently inconsistent: `PARTIALLY_COMPLETED` MUST always
  progress to `ESCALATED` with an operator payload naming what is inconsistent.

### V. Comprehensive Auditability (NON-NEGOTIABLE)
**Traceability from requirement to verified result.**
- Every artifact MUST be traceable: AGENTS.md requirement <-> `spec.md` requirement <-> `tasks.md`
  task <-> automated test <-> execution result. A task mapping to no acceptance criterion is out of
  scope and MUST be deleted.
- The audit trail MUST be append-only: every decision, tool call, policy verdict, approval, and
  state transition is recorded and MUST NEVER be updated or deleted. Every executed plan MUST be
  replayable from audit events alone.
- Logs MUST be structured JSON, one event per line, no multiline free text, carrying `ts`, `level`,
  `service`, `event`, `chain_id`, `disruption_id`, `plan_id`, `tool_call_id`, `correlation_id`.
- Every lifecycle transition MUST be logged as
  `{entity, from, to, reason, chain_id, disruption_id, plan_id, tool_call_id, correlation_id,
  chain_version}`. **An unlogged transition is a defect.**
- OpenTelemetry traces (OTLP only, exported via the ADOT collector) MUST span channel adapter ->
  API -> orchestrator -> MCP server -> partner,
  one trace per disruption. A log line or span missing an available correlation ID is a defect.
- The metrics in AGENTS.md Section 15 MUST be emitted and surfaced on the Terraform-defined
  CloudWatch dashboard under `infrastructure/observability/`.

### VI. Security, Privacy & Prompt-Injection Defense
**Zero hardcoded secrets; least data shared; partner text is data, never instruction.**
- Secrets MUST come only from environment variables locally and AWS Secrets Manager when deployed.
  No secret MAY be committed, logged, or written into plaintext Terraform state. CI MUST fail on a
  secret-scan hit.
- **PII minimization:** logs reference guests by opaque tokenized references only. Raw name, email,
  phone, and payment identifiers MUST stay in the platform's system of record and MUST NEVER be
  copied into StillOn tables, logs, traces, or LLM prompts. Redaction MUST happen at the logging
  boundary so one bad call site cannot leak. Each partner receives only the fields it needs.
- **Money movement is never ours.** StillOn MUST NEVER hold card data, call a PSP, or move funds;
  it MAY only trigger a platform-side workflow via an event.
- **Authentication (pinned):** OIDC/OAuth for guest authentication; OAuth 2.1 authorization-code
  flow with PKCE (S256) plus account linking for the public Assistant MCP surface, with a bearer
  token required on every call; machine-to-machine auth between deployed services; a distinct IAM
  task role per ECS service with no static AWS keys.
- **Session expiry handling:** any HTTP 401 from an `/api` endpoint MUST redirect the channel
  adapter's user to the login screen; the adapter MUST NOT retry silently or degrade to an
  unauthenticated path.
- **Authorization on every state-changing endpoint:** authenticated caller, ownership check that the
  caller is the trip owner for the `chain_id`, the version check appropriate to the entity
  (`workflow_version` for approvals and workflow transitions, `chain_version` for chain mutations),
  policy check. No unauthenticated mutation MAY exist. Rate limits are mandatory on guest-facing and public MCP endpoints.
- **Schema validation both directions:** every MCP tool input *and* output MUST be validated against
  its versioned schema and rejected with `VALIDATION_ERROR` on type mismatch or
  `additionalProperties`. A malformed partner response is a failure, not data.
- **Prompt-injection defense:** all partner-returned text is untrusted **data**. It MUST NEVER
  appear in a system prompt or any instruction position; it MUST be passed only inside a delimited
  labeled data block, stripped of control characters, capped at 200 characters per field, and MUST
  NEVER influence tool selection or authority.
- Low-level partner tools (`modify_reservation_using_hold`, `exchange_tickets_using_hold`,
  compensating actions) MUST NEVER be exposed on the northbound Assistant MCP surface.

### VII. Rigorous Quality Assurance
**A failing test is a finding, not an obstacle.**
- Every functional requirement MUST map to testable acceptance criteria, and every acceptance
  criterion MUST map to at least one automated test named in `tasks.md`.
- A test MUST NEVER be deleted, skipped, or weakened to make a suite pass. **A skipped test counts
  as a failure.** An acceptance criterion MUST NEVER be rewritten to match what was built.
- Unit-test line coverage MUST be at least 80% across `services/`, `mcp/`, and `packages/`, and
  MUST include every hard constraint (pass and fail), scoring weights and tie-break order, every
  valid and invalid lifecycle transition, and policy tier boundaries (60 vs. 61 minutes, $0.00 vs.
  $0.01 delta).
- Contract, integration, security, and end-to-end suites are mandatory, including one e2e test per
  failure scenario in AGENTS.md Section 13 (seven tests) and the reference-scenario happy path. E2E
  tests MUST be driven through `services/api` or the Assistant MCP surface only, never the web
  client.
- The agent-evaluation harness (`tests/eval/`, >= 20 seeded cases) MUST meet its thresholds, and
  **0 hard-constraint violations are tolerated** — any violation fails the suite.
- `make lint` (zero warnings), `make typecheck` (`mypy --strict` + `tsc`), `make codegen-check`, and
  `make security` MUST all be green before any work is reported done.
- `CHANGELOG.md` MUST be updated with every change. No commit MAY land on `main` without it.
- Drift signals (partner timeouts, injected faults, infrastructure issues) MUST be classified
  distinctly from genuine logic failures; only genuine logic or contract failures block a merge.

### VIII. Spec-Driven Delivery & Scope Integrity
**Elaborate through the gates; never silently reduce scope.**
- Work proceeds through the Spec Kit phases in order: constitution -> specify -> plan -> tasks ->
  implement, with `/speckit-clarify` and `/speckit-analyze` run as additional gates when available.
  `spec.md` MUST state WHAT and WHY only, with no technology choices.
- No application code MAY be written before its governing contract is approved (Principle III).
- At every phase boundary the agent MUST verify: every pinned AGENTS.md Section 8-15 constraint
  still holds; every acceptance criterion maps to a numbered requirement and an automated test;
  every delegated decision has a recorded rationale; no artifact weakens this constitution. A phase
  boundary MUST NEVER be crossed with an unresolved conflict — surface it.
- Scope MUST NEVER be silently reduced. If something cannot be implemented, implement everything
  else and report the gap in `docs/final-report.md`. Product features absent from AGENTS.md MUST
  NEVER be invented, and a pinned constraint MUST NEVER be dropped as a "simplification".
- Non-goals are binding: no autonomous purchase or cancellation outside the tiers, no real payment
  processing, no multi-owner approval workflows, no split-party itineraries, no cross-session
  preference learning or guest profiling, no predictive/weather-driven suggestions, no non-digital
  merchants, no model training or fine-tuning.
- New capabilities become new numbered features (`specs/002-...`). A completed feature spec MUST
  NEVER be modified to absorb new scope.
- The agent proceeds autonomously and stops to ask a human **only** when blocked by unprovisionable
  credentials or access, an irreversible production action, or a genuine product decision AGENTS.md
  does not resolve.

### IX. Reproducible, Containerized Deployment
**One non-interactive command per operation; staging automated, production gated.**
- Every operation MUST be exposed as exactly one deterministic, non-interactive `make` target
  runnable in CI with no prompts. The system MUST run locally from those commands alone.
- Docker + Docker Compose v2 locally; Terraform for AWS (ECS Fargate, RDS PostgreSQL, ElastiCache
  Redis, Secrets Manager, CloudWatch). Dockerfiles, compose files, and service startup scripts are
  mandatory deliverables.
- Automated deployment targets **staging only**. Every service MUST expose `/health` (liveness) and
  `/readiness` (owned critical dependencies only — PostgreSQL, Redis, required internal
  queue/state); ECS target health MUST use `/readiness`. Partner MCP health and circuit-breaker
  state MUST be reported by a separate `/dependencies` endpoint and MUST NEVER fail readiness. `make smoke-staging` MUST run after every deploy, and a failure MUST trigger
  rollback to the previous ECS task definition revision.
- `make deploy-production` MUST fail closed when `APPROVED_BY` and `APPROVAL_TICKET` are absent, and
  no automation MAY invoke it. Rationale: production deployment is irreversible and reserved for
  explicit human authorization.
- Terraform state MUST be remote, encrypted, and locked; RDS-managed master credentials MUST NEVER
  be held in Terraform state; RDS, Redis, and internal MCP services MUST be in private subnets
  behind security-group allow lists with KMS encryption at rest.
- Frugality is mandatory: tag every resource with owner, environment, and feature; record staging
  expiration metadata; configure an AWS Budget alert; provide single-command teardown
  (`make destroy-staging`) documented in `docs/deployment.md`.

## Architecture & Lifecycle Constraints

### Ownership Boundaries — Do Not Blur
- **State:** the Chain Graph Service is the only writer to chain tables and the sole issuer of chain
  versions. Maestro mutates chain state only through the Itinerary MCP.
- **Authorization:** `packages/policy-engine` owns it exclusively. Maestro MAY read tier definitions
  to explain them, never to decide them.
- **Approval:** only `services/api` records approvals, authenticated and version-checked.
- **Partner adapters:** each southbound MCP server owns its partner's protocol, retries, breaker,
  and error mapping. No partner client MAY exist outside `mcp/`.
- **Audit:** every service appends audit events; nothing deletes them.
- **Channels:** `apps/web` and the Assistant MCP surface are peer adapters over `services/api` and
  own presentation only. Channel-specific logic MUST NEVER appear in the orchestrator, the policy
  engine, or the southbound MCP layer.
- Internal service-to-service communication uses ordinary APIs and events, **not** MCP.

### Lifecycles
Two lifecycles persist as separate entities: a **chain** row and a **recovery_workflow** row keyed
by `disruption_id`. A chain MUST NEVER enter a recovery state, and recovery states MUST NEVER
outlive their disruption.

- **Chain:** `ACTIVE -> OBSERVING <-> (recovery workflow open) -> CLOSED`.
- **Recovery workflow:** `DETECTED -> ANALYZING -> OPTIONS_READY -> AWAITING_APPROVAL -> EXECUTING
  -> VERIFYING -> COMPLETED`, with `ESCALATED`, `CANCELLED`, and `PARTIALLY_COMPLETED` per AGENTS.md
  Section 9. Any transition not listed there MUST be rejected with `409 INVALID_TRANSITION`.
- **On entering `DETECTED`:** the deterministic templated heads-up MUST be sent via `notify_guest`
  before any planning, exactly once per `disruption_id`. This path MUST contain no LLM call, no
  booking-partner call, no recovery planning, and no policy decision — only the deterministic state transition and a server-templated Notification MCP call.
- Transient tool failures retry inside the MCP layer; a state is re-entered **only** on new
  information. Writes to both rows apply under an optimistic version check.

### Technology Stack (Pinned — Mandatory)
- **Runtime:** Python 3.12. Official MCP Python SDK, major version pinned in `plan.md`.
- **Transport:** Streamable HTTP for all deployed MCP servers, northbound and southbound; STDIO
  only for local developer tooling.
- **Model:** Claude via Amazon Bedrock by default, behind a model provider interface, with a
  deterministic fake provider for unit, contract, and most integration tests.
- **State:** PostgreSQL 16 authoritative. Redis 7 for cache, distributed locks, and expiring holds
  only — **never** authoritative booking state.
- **Agent framework:** none. Maestro is a bespoke orchestrator — the AGENTS.md Section 9 state
  machine, the MCP Python SDK for tool transport, and Bedrock behind the model provider interface.
  Libraries that own the agent loop, hide tool dispatch, or manage conversation state (LangChain,
  LangGraph, LlamaIndex, CrewAI, AutoGen, Bedrock Agents, and equivalents) MUST NOT be introduced:
  in-handler authorization and the single-LLM-component boundary are auditable only while we own the
  loop.
- **Telemetry:** application code emits OTLP only and MUST NOT call a vendor telemetry SDK. An ADOT
  collector sidecar exports traces to AWS X-Ray, with logs and metrics to CloudWatch; local runs use
  an OTLP collector plus Jaeger. Any trace rooted at a `disruption` event MUST be sampled. Every
  model call MUST be recorded as an OTel span with model id, token counts, latency, cost in cents,
  proposed option labels, and the Policy Engine verdict; prompts and completions MAY be captured
  only after the redaction filter. A dedicated LLM-observability tool is permitted only when
  self-hosted inside the account boundary and fed from the collector, never by a vendor SDK in
  application code.
- **Build and test toolchain:** `uv` (committed `uv.lock`); `pytest` + `pytest-cov` as the sole
  coverage measurement; `asyncio` with no blocking I/O in request, tool, or adapter paths;
  `alembic` migrations, forward and reversible, never destructive against append-only audit tables;
  current Node LTS pinned in `.nvmrc` with `pnpm`; GitHub Actions invoking only AGENTS.md Section 19
  `make` targets; multi-stage containers on `python:3.12-slim` running as a non-root user.
- **Substitutable only with a documented rationale in `plan.md`:** web framework (FastAPI default)
  and the frontend (minimal React or server-rendered, demo UX only).

## Development Standards & Quality Gates

### Repository Layout
The structure in AGENTS.md Section 18 is binding. Cross-cutting rules: `packages/contracts/` is the
MCP schema SSOT and MUST be imported, never duplicated; `packages/generated/` is codegen output and
MUST NEVER be hand-edited; `packages/policy-engine/` MUST NEVER be registered as a model-callable
server.

### Documentation
- All documentation MUST be centralized in `docs/`, including `docs/deployment.md`,
  `docs/queries.md`, and `docs/final-report.md`.
- `README.md` MUST include architecture, running instructions, and usage, and MUST reflect the
  actual `make` targets and behavior.
- Spec Kit artifacts live under `specs/001-stillon-core/` and MUST remain mutually consistent with
  each other, with this constitution, and with the implementation.

### Definition of Done
Work is done only when: every AGENTS.md Section 21 acceptance criterion passes via automated test;
the reference scenario and all seven failure scenarios are demonstrated by automated tests; zero
unauthorized actions are possible and bypass tests fail closed; reliability rules are verified by
test and the audit trail is complete, replayable, and append-only; `make lint`, `make typecheck`,
`make codegen-check`, `make security`, and `make build` are green; `make deploy-staging` and `make
smoke-staging` pass with metrics visible on the dashboard; and `CHANGELOG.md`, `docs/`, and the
`specs/001-stillon-core/` artifacts match the implementation.

## Governance

This Constitution supersedes all other development practices. `AGENTS.md` remains the authoritative
source for product mission, pinned architecture, autonomy boundaries, security constraints, and
acceptance criteria; this Constitution encodes those pins as enforceable gates and MUST NOT weaken,
omit, or reinterpret any of them.

**Conflict precedence.** When two instructions conflict, apply in this order and record the
resolution in `docs/final-report.md`:
1. The Non-Negotiable Principles (I and V, and the prohibitions in VIII).
2. Security, privacy, and customer-authorization boundaries (Principles I and VI).
3. The Autonomy Policy tiers (Principle I).
4. Acceptance criteria (AGENTS.md Section 21).
5. Architecture and implementation details.

**Amendments.** Amendments require a Pull Request that updates this file, prepends an updated Sync
Impact Report, and receives team ratification. An amendment that would weaken an AGENTS.md pin MUST
instead amend `AGENTS.md` first, with rationale.

**Versioning policy (semantic).**
- **MAJOR** — backward-incompatible governance change: a principle removed or redefined.
- **MINOR** — a new principle or section added, or materially expanded guidance.
- **PATCH** — clarifications, wording, or non-semantic refinements.

**Compliance.** Every PR MUST verify compliance with Principles I-IX, and every Spec Kit phase
boundary MUST re-verify the pinned constraints. Complexity MUST be justified in `plan.md`.

**Violations that MUST be rejected outright:** hardcoded or logged secrets; raw PII in tables, logs,
traces, or prompts; implementation code edited ahead of its governing contract; hand-edited
`packages/generated/`; a mutation handler that does not invoke the Policy Engine server-side; a
southbound partner tool reachable through the northbound surface; an approval recorded outside the
authenticated version-checked API endpoint; success reported without read-back verification; a
deleted or skipped test; a commit to `main` without a `CHANGELOG.md` entry.

**Version**: 1.0.4 | **Ratified**: 2026-08-07 | **Last Amended**: 2026-08-08
