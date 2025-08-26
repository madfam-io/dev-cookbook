# SOFTWARE\_SPEC.md — MADFAM/Aureo Labs “AI‑Enhanced Codebases & Apps Cookbook”

**Owner:** Aureo Labs (MADFAM)
**Author:** MADFAM CEO’s Executive Assistant
**Version:** 0.1 (Draft for review)
**Last Updated:** 2025‑08‑26 (America/Mexico\_City)

---

## 0) Executive Summary

A web application + CLI/tooling that codifies MADFAM’s proprietary (open‑source) methodology to go from zero → enterprise‑grade production in record time while meeting Aureo Labs’ gold‑standard quality. The app ships a browsable “Cookbook” of step‑by‑step **recipes** with scaffolds, code snippets, guardrails, and **evals**, plus an opinionated **CLI** and repo templates to generate production‑ready code.

**Primary outcomes**

* Reduce feature lead time by ≥40% via meta‑programming templates and AI‑assisted scaffolds.
* Increase quality (eval pass‑rate ≥95% on golden tasks) with integrated safety/guardrail checks.
* Enforce platform standards (security, observability, cost) by default.

**Non‑goals**

* Not a hosted multi‑tenant IDE.
* Not an experiment tracker for model training (basic eval dashboards only).
* Not a vendor‑locked AI platform; must remain provider‑agnostic.

---

## 1) Personas & UX Principles

**Personas**

* **Product/Tech Lead (decision‑maker):** needs clarity, trade‑offs, checklists, ROI.
* **Full‑stack/ML Engineer (builder):** wants copy‑pasteable code, CLI scaffolds, tests that pass.
* **Security/Compliance (reviewer):** needs auditability, SBOM, data maps, DLP controls.
* **Support/Ops (operator):** needs runbooks, SLOs/alerts, rollback mechanics.

**UX Principles**

* “Recipe first”: every feature flows from a recipe card with **When to use → Scaffold → Implement → Guardrails → Evals → Ops → Launch checklist**.
* EN/ES parity; WCAG 2.2 AA accessibility; frictionless copy/share/export.
* All code snippets verified by CI; “Copy with env vars” and “Open in StackBlitz/Devcontainer” actions.

---

## 2) Scope & Success Criteria

**In‑scope**

* Public cookbook webapp; Admin authoring UI; `madfam` CLI; monorepo template; eval harness; minimal cost/telemetry dashboard.

**Success metrics (v1)**

* Onboarding < 15 minutes with green CI on template repo.
* P95 page latency ≤ 1.0s, 99.9% uptime.
* ≥20 production‑grade recipes shipped (R1–R10 seeds provided).
* ≥95% pass on golden evals for sample projects.
* Security gates active (SAST, secrets, SBOM) on default CI.

---

## 3) System Overview

**Components**

1. **Webapp (Cookbook UI):** Next.js (App Router) with Tailwind + shadcn/ui, EN/ES i18n, docs/recipes viewer, search, filters, deep links.
2. **API Layer:** TypeScript (NestJS) or Python (FastAPI)—pick TS for tighter monorepo integration. Serves recipes, search, authz, eval results.
3. **Content Store:** PostgreSQL (primary) + pgvector (semantic search) + S3‑compatible blob for images/assets.
4. **AI Layer:** Provider‑agnostic client with prompt templates, function/tool calling, output validators, caching and cost caps.
5. **Evals Service:** Deterministic harness, golden sets, A/B multi‑model runner, dashboards.
6. **CLI (`madfam`):** Scaffolds repos/features, runs evals, enforces policy, proposes PRs.
7. **Observability:** OpenTelemetry traces/logs/metrics; Sentry; Prometheus/Grafana; LLM telemetry collector.
8. **CI/CD:** GitHub Actions with required gates; release trains and feature flags.

**High‑level flow**
User selects a recipe → reads rationale & copy‑pastes scaffold → runs `madfam recipe add` → code, prompts, tests, and evals generate → CI runs security/quality/evals → deploy to preview → guardrails & SLOs enforced out‑of‑the‑box.

---

## 4) Functional Requirements

### 4.1 Cookbook (Public UI)

* Browse/search (keyword, tag, level, stack, domain).
* Recipe detail page with sections: When to Use, Prereqs, Scaffold (CLI + code), Implement, Guardrails, Evals, Ops, UX/i18n, Checklists.
* Copy actions: “Copy code”, “Copy with envs”, “Copy `.env.sample`”, “Open in DevContainer”.
* Toggle code in **TypeScript**/**Python** where available.
* EN/ES toggle; printable/exportable to PDF/Markdown.

### 4.2 Admin (Authoring)

* Role: **Admin/Maintainer** can create/update recipes and publish versions with changelogs.
* Markdown/MDX editor with schema‑validated front‑matter and live preview.
* Versioning of prompts and evals per recipe; diff view and rollback.
* Draft → Review → Publish workflow with CODEOWNERS routing; scheduler for staged releases.

### 4.3 Evals & Telemetry

* Run recipe‑scoped evals locally (CLI) and in CI; upload results to dashboard.
* Show per‑recipe metrics: pass‑rate, latency, cost/token, groundedness, refusal‑correctness, safety.
* Red‑team pack execution (prompt injection, jailbreak, PII leak).

### 4.4 CLI / Dev Agents

* `madfam init` (monorepo + CI + IaC + baseline prompts/evals).
* `madfam recipe add <R#|slug>` (scaffold files, tests, flags, docs).
* `madfam eval run [suite]` (emit junit, HTML report; compare to baseline).
* `madfam pr review|fix` (repo‑aware suggestions; PR‑only writes; audit logged).
* `madfam ops budget --enforce` (enforce token/cost caps).

### 4.5 AuthZ & Governance

* Roles: **Viewer, Contributor, Maintainer, Admin**.
* OIDC login (optional for public), RBAC with policy for authoring actions.
* Audit logs for publish/edit/eval runs.

---

## 5) Non‑Functional Requirements

* **Performance:** P95 ≤ 1.0s (cached pages), P99 ≤ 1.5s; TTI ≤ 2s on median device/network.
* **Availability:** ≥99.9% monthly.
* **Security:** OWASP ASVS L2; SBOM attached to releases; container scans clean or waived.
* **Privacy:** PII minimization; no secrets in logs; DLP filters on admin inputs; encryption in transit/at rest.
* **i18n/A11y:** EN/ES parity; WCAG 2.2 AA.
* **Cost controls:** Feature‑level budgets; circuit breakers; cost telemetry.

---

## 6) Data Model (Initial)

### 6.1 Entities

* **Recipe** `(recipes)`

  * `id` (uuid), `slug` (unique), `title`, `summary`, `level` (intro/intermediate/advanced), `tags[]`, `stack[]` (ts/python), `status` (draft/published), `version`, `changelog`, `owner`, `created_at`, `updated_at`.
  * `front_matter` (jsonb), `body_mdx` (text), `assets[]` (urls), `feature_flag`.
* **Step** `(recipe_steps)` — ordered list with `title`, `body_mdx`, `code_refs[]`.
* **CodeSnippet** `(code_snippets)` — `language`, `path_hint`, `content`, `checksum`.
* **Prompt** `(prompts)` — `name`, `version`, `template`, `variables[]`, `schema`, `tests[]`.
* **EvalSuite** `(eval_suites)` — `name`, `version`, `metrics[]`, `thresholds`, `cases[]`.
* **EvalRun** `(eval_runs)` — `suite_id`, `recipe_id`, `model`, `params`, `started_at`, `metrics`, `report_url`.
* **Guardrail** `(guardrails)` — `type` (validator/filter/policy), `config`, `status`.
* **User** `(users)` — auth, profile, role; audit trails `(audit_logs)`.

### 6.2 Recipe Front‑matter (JSON Schema sketch)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["slug","title","summary","level","tags","prereqs"],
  "properties": {
    "slug": {"type":"string"},
    "title": {"type":"string"},
    "summary": {"type":"string"},
    "level": {"enum":["intro","intermediate","advanced"]},
    "tags": {"type":"array","items":{"type":"string"}},
    "stack": {"type":"array","items":{"enum":["ts","python"]}},
    "prereqs": {"type":"array","items":{"type":"string"}},
    "scaffold": {"type":"object","properties":{
      "cli": {"type":"string"},
      "files": {"type":"array","items":{"type":"string"}}
    }},
    "evals": {"type":"object","properties":{
      "suite": {"type":"string"},
      "thresholds": {"type":"object"}
    }},
    "guardrails": {"type":"array"}
  }
}
```

---

## 7) APIs (OpenAPI outline)

```
GET    /api/recipes?query=&tag=&level=&stack=      → list recipes (paginate)
GET    /api/recipes/{slug}                          → recipe detail (mdx + front‑matter)
POST   /api/recipes                                 → create (Admin)
PUT    /api/recipes/{id}                            → update (Maintainer+)
POST   /api/evals/run                               → trigger eval run for suite
GET    /api/evals/runs?recipeId=                    → list eval runs
GET    /api/search?q=                               → hybrid search (BM25 + vector)
GET    /api/telemetry/summary                       → cost/latency/eval KPIs
GET    /api/auth/me                                 → user/role
```

**Response types** are typed with TypeScript and generated clients for the webapp/CLI.

---

## 8) Architecture & Tech Choices

* **Monorepo:** pnpm + Turborepo.
* **Frontend:** Next.js (App Router), Tailwind, shadcn/ui, MDX; Playwright E2E.
* **Backend:** NestJS (Fastify adapter); Zod schemas; Prisma ORM (Postgres + pgvector).
* **AI Layer:** provider‑agnostic client; prompt templates in repo (`/packages/ai/prompts`), function/tool calling, zod‑validated outputs, caching, retries/backoff.
* **Search:** Hybrid (Postgres full‑text + pgvector).
* **Auth:** OIDC (Auth0/Cognito/Keycloak), RBAC/ABAC; audit logs.
* **Infra:** Docker; IaC Terraform; environments dev/stage/prod; feature flags (Flagsmith/LaunchDarkly).
* **Observability:** OpenTelemetry → collector → Grafana/Tempo/Loki; Sentry for FE/BE errors.
* **Docs:** Recipes as MDX; static export for public pages; admin behind auth.

**Directory layout**

```
apps/
  web/        # Next.js UI
  api/        # NestJS API
packages/
  ui/         # shared components
  ai/         # providers, prompts, tools, validators, cost tracker
  core/       # types, schemas, utils
  cli/        # `madfam` (oclif)
infra/
  terraform/  # IaC
  k8s/        # optional manifests
.github/
  workflows/  # CI/CD
```

---

## 9) Prompting, Tools & Guardrails

* **Prompt templates:** versioned files with typed variables; render via mustache/TS template literals; tracked in git with changelogs.
* **Output contracts:** zod schemas per tool/task; hard‑fail on invalid outputs.
* **Tool registry:** declarative metadata (permissions, cost hints, idempotency) for safe function‑calling.
* **Content filters:** PII redaction, toxicity filters, allow‑listed domains for retrieval tools.
* **Caching:** prompt/result cache with TTL; embedding cache; offline mode for privacy.
* **Cost caps:** per‑request and per‑feature budgets; circuit breakers; anomaly alerts.

---

## 10) Evaluations & Quality Gates

* **Golden sets:** curated examples per recipe with expected outputs and rationales.
* **Metrics:** exact match, rubric, groundedness, refusal correctness, toxicity, latency, cost.
* **Harness:** deterministic seeds, multi‑model runners, regression detection; JUnit + HTML reports.
* **CI Gate:** block merge unless eval pass‑rate ≥95% on goldens for changed components.
* **Red‑team packs:** prompt injection, jailbreak, data exfiltration; must meet thresholds before publish.

---

## 11) CI/CD & Release Management

* **Branching:** trunk‑based with short‑lived branches; Conventional Commits; signed commits.
* **Pipelines:** verify → build → test (unit/e2e/evals) → security (SAST, secrets, SBOM, container scan) → deploy (PR preview → stage canary → prod progressive).
* **Releases:** semantic‑release/Changesets; weekly train; feature flags & canaries; SLSA provenance + cosign image signing.

---

## 12) Security, Privacy & Compliance

* **Secure SDLC:** threat modeling per recipe; PR approvals (CODEOWNERS); dependency and container scanning.
* **Runtime:** CSP, SRI, Trusted Types; egress allow‑list; WAF; DLP on admin routes.
* **Secrets:** SOPS/Vault; OIDC for CI; no secrets in prompts/logs; rotation policy.
* **Data:** classification (Public/Internal/Confidential/Restricted); data map; retention defaults; RLS for multi‑org authoring (future).
* **Audit:** end‑to‑end request tracing with prompt digests and tool call logs.

---

## 13) Internationalization & Accessibility

* **i18n:** EN/ES with shared message catalogs; locale‑aware dates/numbers; RTL‑ready.
* **A11y:** keyboard nav, focus states, semantic landmarks, high contrast mode; axe checks in CI.

---

## 14) Observability & Cost Management

* **Tracing:** UI → API → model/tool calls with spans; attach token counts and model names.
* **Metrics:** latency, error rate, cache hit rate, token usage, cost per request/feature; burn‑rate SLOs.
* **Alerts:** SLO burn; cost anomalies; safety violations; eval regressions.
* **Runbooks:** top 10 failure modes (timeouts, rate limits, provider outage, schema drift, cache storms).

---

## 15) Feature Flags & Config

* Flags: `evals.required`, `ai.provider`, `ai.cache.enabled`, `public.search.enabled`, `recipes.experimental`, `admin.enabled`.
* Config via env + typed config module; `.env.sample` auto‑generated by CLI.

---

## 16) Environment Variables (initial)

| Key                            | Description                   | Scope   |
| ------------------------------ | ----------------------------- | ------- |
| `DATABASE_URL`                 | Postgres connection           | api     |
| `VECTOR_EXTENSION`             | pgvector enabled toggle       | api     |
| `BLOB_BUCKET`                  | S3 bucket name                | api     |
| `OIDC_ISSUER/CLIENT_ID/SECRET` | Auth                          | web/api |
| `TELEMETRY_ENDPOINT`           | OTel collector                | all     |
| `LLM_PROVIDER`                 | provider selector             | api     |
| `LLM_API_KEY`                  | provider key (Vault injected) | api     |
| `FLAGS_ENDPOINT`               | feature flags SDK URL         | web/api |

---

## 17) Acceptance Criteria (v1)

* [ ] 20+ recipes seeded (R1–R10 variants), each with guardrails and eval suites.
* [ ] CLI scaffolds repos and at least 5 feature recipes end‑to‑end; CI green on template.
* [ ] Search returns relevant recipes (MAP\@10 ≥ 0.7 on seed queries).
* [ ] Evals dashboard shows pass‑rate, latency, cost; CI blocks below thresholds.
* [ ] EN/ES parity on public pages; axe scan passes.
* [ ] SBOM attached to releases; container scans clean or waived.
* [ ] P95 page latency ≤ 1.0s; uptime ≥ 99.9%.

---

## 18) Milestones & Timeline

* **M0 (Week 0):** Confirm stack/guardrails; finalize spec.
* **M1 (Week 1):** Monorepo + CLI skeleton; 5 core recipes; CI gates.
* **M2 (Week 2):** Admin authoring, search, eval harness; 12 recipes.
* **M3 (Week 3):** Observability/cost dashboards; red‑team packs; 20 recipes.
* **M4 (Week 4):** Harden & public launch; governance docs; contribution guide.

---

## 19) Risks & Mitigations

* **Vendor drift / API changes:** provider‑agnostic layer; contract tests.
* **Content sprawl:** schema‑validated front‑matter; review SLAs; ownership in CODEOWNERS.
* **Eval brittleness:** maintain golden sets; regression budgets; HITL sampling.
* **Cost spikes:** per‑feature caps; circuit breakers; anomaly alerts.
* **Security regressions:** mandatory scans, dependency pinning, SBOM provenance.

---

## 20) Open Questions (for Aldo & Leads)

1. Final decision: **NestJS vs FastAPI** (defaulting to NestJS for monorepo cohesion).
2. Cloud baseline (Vercel + managed Postgres vs self‑hosted K8s?).
3. License model (AGPL core + commercial add‑ons vs permissive + CLA).
4. Public contributions: do we run external PR evals on our infra or require local reports?
5. Any EU data residency requirements at launch?

---

## 21) Appendices

### A) Sample Wireframes (descriptions)

* **Home:** hero + search + featured recipes + “Start in 15 minutes” CTA.
* **Library:** left filters (tags/level/stack) + cards grid + sort by popularity/recent.
* **Recipe:** title + quick actions (copy, open in devcontainer, run CLI) + sticky ToC + eval status badge.
* **Admin:** editor (front‑matter + MDX), diff viewer, evals panel, publish dialog.

### B) Example Recipe Front‑matter (YAML)

```yaml
slug: rag-hybrid-search
title: RAG with Hybrid Search (BM25 + Embeddings)
summary: Production-ready RAG with fresh indexing, citations, and guardrails.
level: intermediate
stack: [ts]
tags: [ai, rag, search]
prereqs: [postgres, pgvector]
scaffold:
  cli: madfam recipe add rag-hybrid-search
  files: [apps/api/src/rag/*.ts, packages/ai/prompts/rag/*.md]
evals:
  suite: rag_hybrid_v1
  thresholds: { groundedness: 0.9, cost_per_query: 0.002 }
```

### C) PR Template (excerpt)

* Why / Outcome
* What changed
* Tests (unit/e2e/evals)
* Risk & Rollback
* Screenshots / Logs

### D) Feature Flag Register (initial)

* `recipes.experimental` (gate WIP content)
* `evals.required` (block publish below thresholds)
* `ai.cache.enabled`
* `public.search.enabled`
* `admin.enabled`
