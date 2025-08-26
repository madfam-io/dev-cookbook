# MADFAM / Aureo Labs — AI‑Enhanced Codebases & Apps Cookbook

> Production‑grade recipes, scaffolds, evals, and guardrails to go from zero → enterprise fast.

![status-badges](https://img.shields.io/badge/monorepo-turborepo-blue) ![node](https://img.shields.io/badge/node-%E2%89%A520.x-brightgreen) ![pnpm](https://img.shields.io/badge/pnpm-9.x-orange) ![license](https://img.shields.io/badge/license-TBD-lightgrey)

## Table of Contents

* [What is this?](#what-is-this)
* [Features](#features)
* [Stack](#stack)
* [Repo Layout](#repo-layout)
* [Quickstart](#quickstart)

  * [Prereqs](#prereqs)
  * [Bootstrap](#bootstrap)
  * [Run Dev](#run-dev)
  * [CLI (`madfam`)](#cli-madfam)
* [Environment](#environment)
* [Scripts](#scripts)
* [Testing & Evals](#testing--evals)
* [CI/CD](#cicd)
* [Security & Privacy](#security--privacy)
* [Contributing](#contributing)
* [Versioning & Releases](#versioning--releases)
* [Support & Contact](#support--contact)
* [License](#license)

---

## What is this?

This monorepo contains the **Cookbook webapp** and tooling for MADFAM/Aureo Labs’ AI‑assisted software methodology:

* A public **Next.js** site with step‑by‑step **recipes** (scaffolds, code, guardrails, evals).
* A **NestJS** API powering search, authoring, eval runs, and telemetry.
* A provider‑agnostic **AI layer** (prompts, tools, validators, cost caps).
* A repo‑native **CLI (`madfam`)** to scaffold features, run evals, and enforce policy.

For the full product spec, see [`/docs/SOFTWARE_SPEC.md`](./docs/SOFTWARE_SPEC.md).

---

## Features

* 📚 **Recipe library** (EN/ES) with copy‑pasteable code and one‑click scaffolds.
* 🤖 **CLI & dev agents** for repo‑aware PR reviews and safe codegen.
* 🧪 **Evals harness** with golden sets, red‑team packs, and CI gates.
* 🔐 **Security by default:** SBOM, scans, secrets policy, audit trails.
* 📈 **Observability:** OpenTelemetry traces, Sentry errors, cost/latency metrics.
* 🧩 **Provider‑agnostic AI:** function‑calling, schema‑validated outputs, caching.

---

## Stack

* **Monorepo:** pnpm + Turborepo
* **Webapp:** Next.js (App Router), Tailwind, shadcn/ui, MDX, Playwright
* **API:** NestJS (Fastify), Prisma (Postgres + pgvector)
* **AI:** TypeScript packages (prompts, tools, zod validators, cost tracker)
* **Data:** Postgres, pgvector; S3‑compatible blob storage
* **Infra:** Docker, Terraform (optional), feature flags (Flagsmith/LaunchDarkly)
* **Obs:** OpenTelemetry → collector → Grafana/Tempo/Loki; Sentry

---

## Repo Layout

```
apps/
  web/        # Next.js UI (public cookbook + admin authoring)
  api/        # NestJS API (search, authoring, evals, telemetry)
packages/
  ui/         # Shared React components
  ai/         # Providers, prompts, tools, validators, cost tracker
  core/       # Types, schemas, utilities, config
  cli/        # `madfam` CLI (oclif)
infra/
  docker/     # docker-compose for local Postgres/pgvector
  terraform/  # optional IaC
  k8s/        # optional manifests
docs/
  SOFTWARE_SPEC.md  # product spec
  protocol/         # methodology and recipes guidelines
.github/
  workflows/  # CI/CD pipelines
```

---

## Quickstart

### Prereqs

* **Node** 20.x LTS, **pnpm** 9.x
* **Docker** (for local DB)
* **Git** with signed commits (GPG/Sigstore) recommended

### Bootstrap

```bash
# 1) Clone
git clone https://github.com/madfam-labs/ai-cookbook.git
cd ai-cookbook

# 2) Install deps
pnpm install

# 3) Start infra (Postgres + pgvector)
docker compose -f infra/docker/compose.yml up -d

# 4) Seed envs
cp .env.example .env                 # root
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env

# 5) Generate & migrate DB (Prisma)
pnpm -w prisma generate
pnpm -w prisma migrate dev
```

### Run Dev

```bash
# Start everything (web + api) with turbo
pnpm dev

# Or run individually
pnpm --filter @madfam/api dev
pnpm --filter @madfam/web dev
```

Visit **[http://localhost:3000](http://localhost:3000)** for the webapp; API defaults to **[http://localhost:4000](http://localhost:4000)**.

### CLI (`madfam`)

```bash
# Build and link the CLI locally
pnpm --filter @madfam/cli build
pnpm --filter @madfam/cli link --global

# Use it
madfam init                          # scaffold new project from templates
madfam recipe add rag-hybrid-search  # add a feature recipe
madfam eval run rag_hybrid_v1        # run evals (HTML + JUnit reports)
madfam pr review                     # repo-aware review suggestions (PR-only writes)
```

> All CLI actions are **least‑privilege** and **dry‑run** by default; write actions go through PRs with CODEOWNERS.

---

## Environment

Key environment variables (more in `.env.example` files):

| Variable                      | Where | Description                             |
| ----------------------------- | ----- | --------------------------------------- |
| `DATABASE_URL`                | api   | Postgres connection                     |
| `VECTOR_EXTENSION`            | api   | Enable pgvector                         |
| `BLOB_BUCKET`                 | api   | S3 bucket name (MinIO/local ok)         |
| `LLM_PROVIDER`                | api   | AI provider id                          |
| `LLM_API_KEY`                 | api   | Injected via Vault/SOPS (do not commit) |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | all   | OTel collector endpoint                 |
| `FLAGS_ENDPOINT`              | all   | Feature flags SDK URL                   |

**Secrets policy:** No secrets in code, PRs, or logs. Prefer **SOPS/Vault**; CI uses short‑lived OIDC tokens.

---

## Scripts

Common workspace scripts (see `package.json`):

```bash
pnpm dev            # run web + api in watch mode
pnpm build          # build all packages/apps
pnpm lint           # eslint + prettier
pnpm typecheck      # tsc type checks
pnpm test           # unit tests
pnpm e2e            # Playwright tests (web)
pnpm eval           # run eval suites
pnpm release        # changesets/semantic-release
```

---

## Testing & Evals

* **Unit/Integration:** Vitest/Jest for packages and API; Playwright for web E2E.
* **Evals:** Golden sets + red‑team packs live in `packages/ai/evals/`. Run locally or in CI. CI **blocks merges** below thresholds.
* **Reports:** JUnit + HTML in `./artifacts/evals/`; dashboards available in the app.

```bash
pnpm test
pnpm e2e
pnpm eval --suite rag_hybrid_v1
```

---

## CI/CD

* **Pipelines:** verify → build → test (unit/e2e/evals) → security (SAST, secrets, SBOM, container scan) → deploy (PR preview → stage canary → prod progressive)
* **Policies:** trunk‑based, short‑lived branches; **Conventional Commits**; signed commits; CODEOWNERS approvals; auto‑release via Changesets.
* **Artifacts:** SBOM (Syft), image signing (cosign), SLSA provenance.

---

## Security & Privacy

* Threat modeling per feature/recipe; least‑privilege by default.
* PII minimization in prompts; DLP filters on admin/authoring.
* Audit trails for admin changes, eval runs, and agent actions.
* Report vulnerabilities: **[security@madfam.mx](mailto:security@madfam.mx)** (PGP key TBD). Please do not open public issues for sensitive reports.

---

## Contributing

1. Fork and create a feature branch: `feat/<slug>`
2. Follow **Conventional Commits** (`feat:`, `fix:`, `chore:`, etc.)
3. Add/Update tests and **evals** for changed behavior
4. Run `pnpm lint && pnpm typecheck && pnpm test && pnpm eval`
5. Open a PR with the provided template. Ensure CI is ✅.

External contributors may be asked to attach local **eval reports** if CI access is limited.

---

## Versioning & Releases

* Semantic Versioning (SemVer) with **Changesets** for changelog and tags.
* Weekly release train by default; hotfixes as needed.
* All releases ship SBOM and signed images.

---

## Support & Contact

* General questions: issues/discussions in this repo
* Security: **[security@madfam.mx](mailto:security@madfam.mx)**
* Commercial inquiries: **[contact@madfam.mx](mailto:contact@madfam.mx)**

---

## License

**TBD** — see `LICENSE` once finalized (e.g., AGPL for core + commercial add‑ons).

---

### Appendix: Troubleshooting

* **Ports busy?** Stop conflicting services or change ports in `.env`.
* **Migrations fail?** Ensure `docker compose` DB is healthy; rerun `pnpm -w prisma migrate dev`.
* **OTel not exporting?** Check `OTEL_EXPORTER_OTLP_ENDPOINT` and collector availability.

### Appendix: Code of Conduct

By participating, you agree to uphold our respectful, inclusive community standards (see `CODE_OF_CONDUCT.md`).
