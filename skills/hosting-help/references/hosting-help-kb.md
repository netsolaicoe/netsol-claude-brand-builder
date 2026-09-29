# Network Solutions Hosting Help — Knowledge Base

Dual-path answers: **dashboard first**, then **CLI**, then caveats. This file is
the primary source for the hosting-help Skill.

## Dashboard navigation

**Click path (preferred):**

1. Sign in at Network Solutions.
2. Open **Hosting** in the left sidebar.
3. Open your VPS plan (for example "Standard VPS - NVMe 4").
4. Click **View projects** to see all projects (Healthy / Needs attention counts).
5. Click **MANAGE** on a project.
6. Use tabs: **Overview** | **Deployments** | **Configuration** | **Logs** |
   **Settings**.

**Projects URL template** (use only when the user already has `<hosting-id>` from
their browser address bar — never guess it):

```text
https://www.networksolutions.com/my-account/hosting/self-managed-hosting/projects?hostingId=<hosting-id>
```

`<hosting-id>` appears in the URL after the user opens Hosting. An unauthenticated
visit prompts login first.

There is no CLI command to list all projects across the account. The dashboard
projects list is the full inventory view. `netsol target list` lists VPS targets
(hosting servers), not individual application projects.

## Set an environment variable

In the Network Solutions dashboard
1. Go to Hosting and open your VPS plan, then click View projects.
2. Find your project in the list and click MANAGE.
3. Open the Configuration tab.
4. Click ADD VARIABLE for a plain value, or ADD SECRET for an API key or password.
5. Enter the name and value, for example:
   PORT = 3001
6. Save, then click REDEPLOY — variables only apply to new deployments.

Using the CLI
```bash
netsol env set PORT=3001
```

List what is configured and what is still missing:
```bash
netsol env check
```

Then apply it with a redeploy:
```bash
netsol deploy . --approve
```

Your Node code reads it using:
```javascript
const port = Number(process.env.PORT) || 3000;
```

Secrets such as API keys and database passwords must be added with **ADD SECRET**
in the dashboard. Do not commit them to Git, do not paste them into chat, and do
not pass them as CLI flags.

**Unlike Vercel:** there is no interactive `netsol env add` prompt. Use
`netsol env set NAME=VALUE` for plain variables only.

## Remove an environment variable

In the Network Solutions dashboard
1. Go to Hosting → your VPS plan → View projects → MANAGE on your project.
2. Open the Configuration tab.
3. Remove the variable from the list and save.
4. Click REDEPLOY — the change applies only to new deployments.

Using the CLI
```bash
netsol env unset PORT
```

Then redeploy:
```bash
netsol deploy . --approve
```

## List or check environment variables

In the Network Solutions dashboard
1. Go to Hosting → your VPS plan → View projects → MANAGE on your project.
2. Open the Configuration tab to see configured variables (values for secrets are
   not shown after save).

Using the CLI
```bash
netsol env list
netsol env check
netsol env diff
```

These commands show **names** and set/missing status only — never secret values.

**Unlike Vercel:** there is no `netsol env ls`. Use `env list` or `env check`.

## Initialize environment variable names from a template

If the project has no `.env.example` yet:

Using the CLI
```bash
netsol env init
```

This creates a starter scaffold from recipe hints. Review and edit `.env.example`
(add every name your app uses; mark optional names with `# @optional` and
sensitive names with `# @secret`), then run `netsol env check` before packaging.

There is no dashboard equivalent for creating the local manifest file — that
lives in your repository.

## Secrets and sensitive values

**Plain variables** — dashboard ADD VARIABLE or `netsol env set NAME=VALUE`.

**Secrets** (API keys, database passwords, tokens) — dashboard **ADD SECRET**
only. Never use `netsol env set` for secrets (the value would appear in shell
history). Never paste secrets into chat or agent messages.

Using the CLI (opens the hosting console deeplink for secret entry):
```bash
netsol env open
```

Complete secret entry in the browser, then verify:
```bash
netsol env check
netsol deploy . --approve
```

## Config changes do not roll out automatically

Setting or changing environment variables **persists configuration only**. The
running app does not pick up new values until you redeploy.

Using the CLI
```bash
netsol env set NEW_VAR=value
netsol deploy . --approve
```

In the dashboard: save the variable on the Configuration tab, then click
**REDEPLOY**.

There is no config-only rollout — redeploy always queues a full rebuild. Config
is pinned at enqueue time (`config_version` / `config_digest`).

**Unlike Vercel:** there is no separate "save without redeploy" path that updates
a running container. Always redeploy after config changes.

## Environment scope

CLI commands accept `--environment <name>` (default `development`, stored in
`.networksolutions/netsol.config.json`).

**Unlike Vercel:** there are no Production / Preview / Development checkboxes in
the dashboard flow. Scope is a single environment name per deploy context.

Example:
```bash
netsol env check . --environment production
netsol deploy . --approve --environment production
```

## View project status and health

In the Network Solutions dashboard
1. Go to Hosting → View projects → MANAGE on your project.
2. Open the **Overview** tab for live URL, deployment ID, recipe, artifact
   digest, and health status.

Using the CLI
```bash
netsol deployment status [path]
netsol deployment health [path]
```

Pass a project path or a `dep_*` deployment ID. With no argument, the CLI reads
from `.networksolutions/netsol.config.json` in the current project.

## View deployment logs

In the Network Solutions dashboard
1. Go to Hosting → View projects → MANAGE on your project.
2. Open the **Logs** tab.

Using the CLI
```bash
netsol deployment logs [path] --phase build
netsol deployment logs [path] --phase runtime
netsol deployment logs [path] --phase all --limit 50
```

`--phase` defaults to `all`. `--limit` defaults to 50 (max 200). Logs are
redacted and bounded.

For **your** failing deployment with live evidence, use the Diagnose and Recover
Skill — this section documents the commands only.

## Redeploy an application

In the Network Solutions dashboard
1. Go to Hosting → View projects → MANAGE on your project.
2. Click **REDEPLOY** (top right on Overview or Configuration).

Using the CLI
```bash
netsol deploy [path]           # preview only
netsol deploy [path] --approve # publish after explicit approval
```

Redeploy ships a new version onto the **same locked target**. You need sealed
source (`netsol project package`) and matching env config (`netsol env check`)
before deploy.

## Roll back to a prior release

In the Network Solutions dashboard
1. Go to Hosting → View projects → MANAGE on your project.
2. Click **ROLL BACK** (next to REDEPLOY).

Using the CLI
```bash
netsol rollback [path]           # preview
netsol rollback [path] --approve # after explicit approval
```

Rollback is a separate governed action with its own approval. Use the Rollback
Skill for guided execution.

## Delete a project

In the Network Solutions dashboard
1. Go to Hosting → View projects → MANAGE on your project.
2. Open the **Settings** tab.
3. Click **DELETE** and confirm.

Using the CLI
```bash
netsol application delete [path] --approve
```

## Sign in to the CLI

Using the CLI
```bash
netsol login
```

Open the verification URL in your **system browser** (not an embedded IDE
browser). Credentials persist in `~/.netsol/`.

Check session:
```bash
netsol whoami
```

Sign out:
```bash
netsol logout
```

Dashboard sign-in uses the Network Solutions account portal — the same account
that owns the Hosting package.

## How deployment works (lifecycle)

Network Solutions deployment follows this pipeline:

1. **Plan** — detect runtime locally; call `netsol capabilities plan` or the App
   Detect and Plan Skill. Produces `deployment_plan_id` and `workflow_id`.
2. **Authenticate** — `netsol login` (account OAuth).
3. **Target** — `netsol target list` → user chooses VPS → `netsol target select`
   (binds `hosting_target_id`, allocates `application_id`).
4. **Entitlement** — purchase or reuse an owned VPS (`netsol checkout`,
   `netsol workflow wait`, `netsol workflow resume` when needed).
5. **Environment** — `netsol env init` / `netsol env check`; set values via
   dashboard or `netsol env set` (plain vars only).
6. **Dockerfile consent** — ensure root `Dockerfile`; `netsol project dockerfile
   approve` after user confirms.
7. **Package** — `netsol project package` seals an immutable `source_digest`
   (local CLI only; no MCP substitute).
8. **Preview** — `netsol deploy [path]` shows exact publish parameters; user
   approves.
9. **Publish** — `netsol deploy [path] --approve` → gateway → Hosting MCP →
   control plane queues an OCI build.
10. **Build** — isolated build worker produces a signed container image.
11. **Activate** — runtime agent on the VPS pulls the image and starts the
    container.
12. **Live** — reverse proxy routes the hostname; health checks pass; live URL
    is reported.

Purchase is not deploy approval. Each governed write (deploy, rollback, DNS)
requires its own explicit approval.

## Dockerfile and source packaging

The root `Dockerfile` defines how your app is built into a container image.

Using the CLI
```bash
netsol project dockerfile status [path]
netsol project dockerfile approve [path]
netsol project package [path]
```

Consent is recorded in `.networksolutions/dockerfile.json` (digest + timestamp
only — never the file body). Packaging requires local filesystem access; there
is no MCP-only packaging path.

Dry run (scan without upload):
```bash
netsol project package [path] --dry-run
```

## Redeploy vs new deployment vs target switch

- **Redeploy** — same project, same locked `hosting_target_id`, new
  `source_digest` or config. Use `netsol deploy . --approve`.
- **Target lock** — once a project has shipped a release, `hosting_target_id` is
  locked. `target select` with a different target is refused.
- **New deployment on a different VPS** — start a fresh project binding (new
  `.networksolutions/` state), not a target switch on an existing project.

## Supported stacks (MVP)

Network Solutions MVP supports **two** application stacks. Planning uses a
**catalog runtime id** — not an npm package name, PyPI package name, or framework
marketing name.

| Catalog runtime id | Recipe | What qualifies | Required files | Build | Start | Min plan |
| --- | --- | --- | --- | --- | --- | --- |
| `nodejs` | `nodejs-npm-start` v0.1.2 | Node.js apps — **Next.js, Express, and plain Node** | `package.json`, `package-lock.json` | `npm ci` + `npm run build` | `npm start` | `VPS_NVME_4` |
| `fastapi` | `fastapi-uvicorn` v0.1.2 | Python **FastAPI** (`fastapi` in `requirements.txt`, app in `main.py`) | `requirements.txt`, `main.py` | `pip install -r requirements.txt` | `uvicorn main:app --host 0.0.0.0 --port 8000` | `VPS_NVME_4` |

### Node.js and Next.js

- **Next.js apps use catalog id `nodejs`**, not `next` or `nextjs`.
- Detection looks for `package.json` at the project root. The agent declares
  `nodejs` when planning.
- Example:
  ```bash
  netsol capabilities plan [path] --runtime nodejs
  ```

### FastAPI

- The app must expose a FastAPI application in `main.py` and list `fastapi` in
  `requirements.txt`.
- **Django, Flask, and other Python frameworks are not supported** by the current
  recipes.
- Example:
  ```bash
  netsol capabilities plan [path] --runtime fastapi
  ```

### Check whether your app is supported

Use App Detect and Plan to inspect the repository and match against the catalog,
or declare the runtime yourself after inspection:

```bash
netsol capabilities plan [path] --runtime nodejs
netsol capabilities plan [path] --runtime fastapi
```

If the stack is not in the catalog, planning returns **`UNSUPPORTED_STACK`**. The
supported runtime ids are **`nodejs`** and **`fastapi`** only. Do not add or rename
manifest files to fake detection — fix the app stack or choose a supported
framework.

## Not supported in MVP

### Other application stacks

These are **not deployable** on Network Solutions MVP today:

- Ruby, Java/Spring, Go, PHP, Laravel
- Django, Flask, and Python stacks other than FastAPI (as defined above)
- Static-only sites without a recognized runtime manifest
- Dockerfile-only repositories with no matching `package.json` or
  `requirements.txt` + `main.py`

A root `Dockerfile` alone does **not** make an unsupported stack deployable.

### Databases (all types)

**Databases are not supported in MVP** — neither Network Solutions managed
databases nor external third-party databases.

- **No managed or co-located databases** — PostgreSQL, MySQL, MongoDB, and Redis
  provisioning on your VPS is **not available** in MVP. The platform design
  describes managed data services as a **future** capability (Epic 4); `data_*`
  provisioning tools are not shipped yet.
- **No external database connections** — do not configure `DATABASE_URL`,
  `MONGODB_URI`, `REDIS_URL`, or similar for production deploys on Network
  Solutions in MVP. Apps that **require a database at runtime** are not supported
  yet.

If the user asks how to add Postgres, MySQL, MongoDB, or Redis: state clearly that
database support is **not available in MVP** (managed or external). Do not
suggest connecting via environment variables. If they only need to know whether
their **application framework** (without a database) is supported, point them to
App Detect and Plan or the supported stacks table above.

### Other future capabilities (not MVP)

- **Custom domains and SSL** — planned (design Epic 5); not in MVP hosting help
  workflows.
- **Managed data services** — planned (design Epic 4); not in MVP.

## Error code reference (generic)

These codes appear in CLI and MCP responses. For **your** specific failure with
logs and status, use Diagnose and Recover Deployment.

| Code | Meaning | Typical fix |
| --- | --- | --- |
| `UNSUPPORTED_STACK` | Runtime not in catalog | Use a supported stack; see App Detect and Plan |
| `VALIDATION_ERROR` | Missing or stale plan, target, source, or env | Re-run the missing step (`capabilities plan`, `target select`, `env check`, `project package`) |
| `AUTHENTICATION_REQUIRED` | No valid session | `netsol login` |
| `SCOPE_DENIED` | Token lacks required scope | Re-login; check account grants |
| `OWNERSHIP_DENIED` | Target or release not yours | Choose your own target |
| `ENTITLEMENT_REQUIRED` | Hosting not purchased | Complete checkout / fulfillment |
| `APPROVAL_EXPIRED` | Deploy approval stale | Re-preview and approve |
| `PROVIDER_TIMEOUT` | Gateway or backend timeout | Retry identical call with same idempotency key |
| `RATE_LIMITED` | Rate limit hit | Wait for `Retry-After`, then retry |
| `CLI_VERSION_UNSUPPORTED` | CLI too old | Upgrade `netsol-cli` to >= 0.9.0 |

## CLI command quick reference

| Task | Command |
| --- | --- |
| Version | `netsol version` |
| Login | `netsol login` |
| Plan | `netsol capabilities plan [path] --runtime <id>` |
| List VPS targets | `netsol target list` |
| Bind target | `netsol target select <target_id> [path] --application-name <name>` |
| Env check | `netsol env check [path]` |
| Set plain env | `netsol env set NAME=VALUE` |
| Unset env | `netsol env unset NAME` |
| Package source | `netsol project package [path]` |
| Deploy preview | `netsol deploy [path]` |
| Deploy publish | `netsol deploy [path] --approve` |
| Status | `netsol deployment status [path]` |
| Health | `netsol deployment health [path]` |
| Logs | `netsol deployment logs [path] --phase all` |
| Rollback | `netsol rollback [path] --approve` |
| Delete project | `netsol application delete [path] --approve` |
