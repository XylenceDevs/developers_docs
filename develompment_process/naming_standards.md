# Naming Standards Guide (GitHub + AWS)

> **Goal:** short, predictable, lowercase/ASCII names across a multi‑repo setup with 3 environments (`dev`, `staging`, `prod`). Frontends deploy to **S3/CloudFront**; backends to **ECR/ECS**.

---

## 1) Core rules

* **kebab‑case**, lowercase, ASCII. No spaces/accents; avoid `_` unless a service forces it.
* **Stable slugs** for environments: `dev`, `staging`, `prod`.
* Prefer **consistency over creativity**; keep names short and scoped to the thing they identify.

---

## 2) Environments (everywhere)

| Env         | Slug      | Example                   |
| ----------- | --------- | ------------------------- |
| Development | `dev`     | `api.dev.xylence.com`     |
| Staging     | `staging` | `api.staging.xylence.com` |
| Production  | `prod`    | `api.xylence.com`         |

Use the slug in names, tags, and variables. Omit the slug in **public prod DNS** when it reads better.

---

## 3) GitHub (multi‑repo)

### 3.1 Repository names

* **Templates:** `template-frontend`, `template-backend`, `template-poc`
* **Product:** `front-<app>`, `back-<service>`
* **Platform:** `.github` (community health), `ci-workflows` (reusable Actions), `platform-docs`, `infra` (IaC)

**Examples:** `front-founder`, `front-admin`, `back-pulse-api`, `back-soundboard-api`, `platform-docs`.

### 3.2 Branches

* Permanent: `main`, `staging`, `dev`
* Work branches: `feat/<scope>-<summary>`, `fix/<scope>-<summary>`, `chore/...`, `refactor/...`
* Hotfix on prod: `hotfix/<summary>`

### 3.3 Release tags (SemVer)

* `vMAJOR.MINOR.PATCH` → `v0.3.2`
* Pre‑release: `v0.3.2-rc.1` (optional)

### 3.4 GitHub Environments

* `dev`, `staging`, `prod` (require reviewers in `prod`)

### 3.5 Secrets & Vars

* **UPPER_SNAKE_CASE**. Store per‑environment whenever possible.
* **Frontend (per env):** `AWS_REGION`, `AWS_ROLE_ARN` *or* keys, `S3_BUCKET`, `CF_DISTRIBUTION_ID`
* **Backend (per env):** `AWS_REGION`, `AWS_ROLE_ARN` *or* keys, `ECR_REPOSITORY`, `ECS_CLUSTER`, `ECS_SERVICE`, `ECS_CONTAINER_NAME`

---

## 4) AWS naming

Keep names short; use a consistent `xylence` prefix for shared/platform resources.

### 4.1 ECR

* **Repository:** `xylence/<service>` → `xylence/pulse-api`
* **Image tags:**

  * `sha-<12>` → `sha-1a2b3c4d5e6f` (primary for deploys)
  * `branch-<branch>` → `branch-staging`, `branch-dev` (sanitize `/` → `-`)
  * `vX.Y.Z` (on release)
* Avoid relying on `latest` in prod.

### 4.2 ECS

* **Cluster:** `xylence-<env>-cluster` → `xylence-staging-cluster`
* **Service:** `xylence-<service>-<env>-svc` → `xylence-pulse-api-staging-svc`
* **Task family:** `xylence-<service>` → `xylence-pulse-api`
* **Container name:** `<service>` → `pulse-api`
* **Log group (CloudWatch):** `/ecs/xylence-<service>` → `/ecs/xylence-pulse-api`

### 4.3 S3 (frontends)

* **Bucket:** `xylence-front-<app>-<env>-<suffix>` (suffix 3–6 chars if uniqueness needed)

  * Example: `xylence-front-admin-staging-8ab3`
* **Prefixes:** `assets/`, `pr-<n>/` for PR previews

### 4.4 CloudFront

* **Distribution comment:** `xylence-<app>-<env>`
* **CNAMEs:** `app.<env>.xylence.com` / `app.xylence.com` (prod)

### 4.5 Route 53

* **Records:** `<app>.<env>.xylence.com` → A/AAAA → ALB/CloudFront; omit env for public prod

### 4.6 IAM (for CI/CD)

* **OIDC role (per repo/env):** `gha-<repo>-<env>-deploy` → `gha-back-pulse-api-staging-deploy`
* **Policy:** `gha-<repo>-<env>-policy`

### 4.7 Secrets Manager / SSM

* **Secrets Manager:** `xylence/<service>/<env>/<key>` → `xylence/pulse-api/dev/openai/api-key`
* **SSM Parameter:** `/xylence/<env>/<service>/<param>` → `/xylence/staging/pulse-api/db_url`

### 4.8 Networking & ALB

* **VPC:** `xylence-<env>-vpc`
* **Subnets:** `xylence-<env>-pub-a`, `xylence-<env>-priv-b`
* **Security Group:** `xylence-<env>-<component>-sg` → `xylence-staging-ecs-sg`
* **ALB:** `xylence-<env>-alb`
* **Target Group:** `xylence-<service>-<env>-tg`

### 4.9 Datastores & async (if used)

* **RDS:** `xylence-<env>-pg` → `xylence-prod-pg`
* **DB name:** `xylence_<service>` → `xylence_pulse`
* **DB user:** `svc_<env>` → `svc_staging`
* **SQS:** `xylence-<service>-<env>-q` (+ DLQ: `...-dlq`)
* **SNS:** `xylence-<domain>-<env>-topic`
* **Lambda:** `xylence-<service>-<env>-fn`

### 4.10 AWS Tags (resource tags)

Apply at minimum:

* `Project: xylence`
* `Env: dev|staging|prod`
* `Service: pulse-api|admin-front|...`
* `Owner: <team or person>`
* `Repo: back-pulse-api|front-admin`

---

## 5) Artifacts & packages

* **Frontend build artifact (Actions):** `frontend-build`
* **Internal packages (if any):** npm `@xylence/<pkg>`; PyPI `xylence-<pkg>`

---

## 6) Conventional commits (recommended)

* `type(scope): message`

  * `feat(pulse-api): add detailed /health`
  * `fix(front-admin): handle SPA 404`
  * `chore(ci): pin actions by SHA`

---

## 7) Quick reference

* **Repo (back):** `back-pulse-api`
* **Repo (front):** `front-admin`
* **Branch:** `feat/pulse-auth`
* **Tag:** `v0.4.0`
* **ECR repo:** `xylence/pulse-api`
* **Image tag:** `sha-1a2b3c4d5e6f`
* **ECS service:** `xylence-pulse-api-staging-svc`
* **S3 bucket:** `xylence-front-admin-staging-8ab3`
* **CF alias:** `admin.staging.xylence.com`
* **Secret:** `xylence/pulse-api/staging/openai/api-key`

---

> Keep it boring and consistent. If a name feels long, shorten the **service** part first; never drop the **env** unless it is public prod DNS.
