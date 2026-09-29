---
name: app-detect-and-plan
description: Inspect the current repository, detect its runtime, and produce a
  validated Network Solutions deployment plan with an exact hosting plan and
  quote. Read-only — stops before authentication and purchase.
version: 1.7.5
---

# App Detect and Plan (Network Solutions)

Use this Skill when the user asks to deploy, host, or "get this app live" on
**Network Solutions**, or asks what Network Solutions hosting their project
needs.

**Do NOT use this Skill when:**

- The user names another hosting provider (AWS, Vercel, etc.) — say Network
  Solutions tools do not apply and stop. Do **not** inspect the repository,
  list catalog runtimes, plan, or call MCP tools.
- The request is unrelated coding work (refactors, tests, general debugging) —
  do not activate. Do **not** inspect, list catalog runtimes, plan, or call tools.

## What this Skill does

1. Selects a safe execution path (CLI or MCP tools).
2. **Inspects the repository yourself first** — you are the detector. Read only
   safe metadata (manifest files and dependency names), never secrets or file
   contents. Produce a freeform stack label (or labels) plus a short evidence
   list.
3. Fetches the supported-runtime **catalog** from Network Solutions and
   **matches** your freeform findings to catalog `runtime` IDs.
4. On a single match, **declares** that catalog runtime (and recipe) to the
   read-only planning tool, which validates it and returns an exact hosting
   plan, recipe, SKU, and quote.
5. Presents the plan **with the evidence from step 2** and **stops at the
   purchase boundary**.

Supported stacks come from `deployment_list_supported_runtimes` — never invent
catalog IDs. Detect freeform first; declare **only** IDs present in the catalog
response (`data.runtimes[].runtime`).

## Prerequisites and path selection

Reach canonical planning tools in this **availability order**:

1. **Configured `network-solutions-deploy` MCP connection** (public gateway tools).
2. **`netsol-cli`** (≥0.2.0) when installed and compatible.
3. **Trusted Network Solutions hosted continuation** when offered to the user.

If none of the three is available, tell the user how to add the MCP server or
run `npm i -g @network-solutions/cli` so `netsol` is on PATH and use `npx netsol`, and stop. Never substitute a private endpoint, demo tool,
or guessed plan.

When the CLI path is intended, do not run a separate Node/npm version preflight before the bootstrap below. If `npm` or `npx` is unavailable, or `npx netsol version` / global CLI install fails because of Node or npm version limits, give Node.js LTS (>= 22.12) install guidance, stop, and wait for user confirmation.

**CLI bootstrap (strict order):** Run each step as a **separate** shell command — do not chain the version probe with repository scans, `command -v`, bare `netsol version`, or install.

1. Probe first:

```bash
npx netsol version
```

2. **Only if** step 1 fails because the CLI is not installed, install globally, then retry:

```bash
npm i -g @network-solutions/cli
npx netsol version
```

(Alternatively: `pnpm add -g @network-solutions/cli` or `bun add -g @network-solutions/cli` when the user uses those package managers.)

Do **not** run `npm i -g @network-solutions/cli` (or any install) until step 1 has been attempted. Do **not** use `command -v netsol` or bare `netsol version` **instead of** the first `npx netsol version` probe.

Require `data.cli_version` >= `0.2.0` from the version event.

**CLI install anti-patterns:** Do not run `npm install --prefix` to `/tmp`, `/private/tmp`, or any side directory. Do not invoke `node_modules/.bin/netsol` by absolute path. Do not install before the `npx netsol version` probe.

Workflow steps in this skill use **`npx netsol …`**.

Planning does not require MCP OAuth. When the CLI path is intended, do not
sign in to the MCP server before plan — use **`npx netsol capabilities plan`** after
catalog match (public plan steps need no login).

### Use-case routing (when multiple paths are available)

| Use case | Prefer | Why |
| --- | --- | --- |
| Full read-only plan | Inspect + catalog match, then `npx netsol capabilities plan [path] --runtime <catalog-id> [--recipe-id <id>]` | Detect first; pass a catalog id; CLI re-validates against the catalog |
| MCP available but CLI missing | MCP tools (Path B below) | Detect first; fetch catalog; declare a catalog ID |
| Both MCP and CLI available | **CLI for planning** after catalog match; MCP only if CLI is missing or fails with "command not found" | CLI re-validates your declaration |

Never call tools excluded from the compatibility baseline (see
`netsol-cli/contracts/compatibility/README.md`) or any write, purchase, auth,
deploy, DNS, log, or rollback tool from this Skill.

---

## Detecting the runtime (detect first, then catalog)

Order matters:

1. **Inspect the repository** at its root **before** fetching the catalog.
2. Run a **preflight inspect** (manifest-first; evidence = files present, not
   framework brands):
   1. **Choose project root** — repo root, or the user-selected package
      directory in a monorepo.
   2. **Monorepo markers** (when at repo root): `turbo.json`,
      `pnpm-workspace.yaml`, `lerna.json`, or `workspaces` in root `package.json`
      → **ask which package root** to deploy before planning. Until the user
      chooses a package root, do **not** assign a stack label from nested paths
      such as `apps/web/package.json`.
   3. **Strong signals at the chosen root:**
      - **Node.js** — `package.json` exists (evidence: lockfile if present —
        `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`; optional
        `scripts.start` or `main`). Use the short freeform label **`Node.js`**
        (not “Node.js application”) — do not infer or name a framework brand.
      - **FastAPI** — `fastapi` appears in `requirements.txt` or
        `pyproject.toml`. Use the short freeform label **`FastAPI`**.
      - **Other stacks** (Ruby, Django, Java, …) — name them from safe metadata
        only with short labels such as **`Ruby`**, **`Java/Maven`** (e.g. Ruby
        from `main.rb` / `Gemfile`; Java/Maven from `pom.xml`).
   4. Produce **freeform** stack label(s) and a short **evidence list**. Copy
      evidence strings **verbatim** from the repository inventory (e.g.
      `package.json: root manifest`, not paraphrases like “root package.json
      manifest”).
3. **Then** obtain the catalog and **match** freeform findings to catalog
   `runtime` IDs (see **Catalog id vs freeform** below).

Rules:

- Read manifest files and dependency **names** only. Never open `.env`, keys,
  or credentials; never read source bytes, versions, or secret values. When
  `.env` is listed in the repository inventory, add a `refuse_secret_access`
  step, set secret refusal flags, and **omit every `.env` line from the evidence
  list** — evidence stays limited to safe manifest entries only (e.g.
  `package.json: root manifest`).
- A `Dockerfile`, `Procfile`, `docker-compose.yml`, or CI config is **not** a
  supported runtime signal. A repository that contains only those is a
  container-only repo — that is an **unknown** outcome, not a match.
- After a **valid catalog plan**, the Deploy skill may create a root `Dockerfile`
  for image build and record local user consent under
  `.networksolutions/dockerfile.json`. That is a post-plan packaging step — it
  does **not** change detection rules and must **not** be suggested here as a
  way to force an unsupported stack to match.
- After the catalog match, decide exactly one outcome:
  - **one match** — declare that catalog runtime (e.g. `nodejs`, `fastapi`) via
    Path A or Path B. Only the **plan/recommend** step carries the declared
    catalog runtime; inspection, catalog listing, and user-choice prompts do
    **not** declare a runtime yet.
  - **unknown** — zero catalog matches. Stop with `UNSUPPORTED_STACK`. No plan
    call; no best-effort deploy. See **Reporting an unsupported stack** below.
  - **multiple** — more than one catalog match (e.g. Node.js app with
    `package.json` and FastAPI backend) → ask which supported stack to deploy;
    do not guess and do not recommend a plan until the user chooses.

### Reporting an unsupported stack

**Reuse** the freeform stack + evidence from step 1 (no second detection pass)
and give the developer exactly three things, then stop:

1. **What you found** — the stack name with its evidence (e.g. "Java/Maven via
   `pom.xml`"). When step 1 identified nothing, say so plainly and name the
   files you did see (e.g. "only a `Dockerfile`, no recognized runtime
   manifest").
2. **Why you cannot proceed** — that runtime is not in the Network Solutions
   supported-runtime catalog.
3. **The supported runtimes** — from the catalog (`data.runtimes[].runtime`) or
   the error's `data.supported`. Include those catalog runtime ids (e.g.
   `nodejs`, `fastapi`) and the literal status `UNSUPPORTED_STACK` in the user
   message. Reuse prior evidence strings **verbatim** in the message.

**Never suggest adding, creating, renaming, or scaffolding a detection file.**
Do not tell the developer to add `package.json`, `requirements.txt`,
`pyproject.toml`, or any other manifest so the app becomes detectable or
supported. Refuse scaffolding with the phrase **“Manifest scaffolding is
refused.”** without echoing filenames next to words like add, create, or
scaffold (even when refusing a user request to add those files).

### Catalog id vs freeform

Freeform inspection evidence is **not** a valid `--runtime` / declare value.

| Freeform evidence (inspect) | Catalog `runtime` id (declare / `--runtime`) |
| --- | --- |
| Short label `Node.js`; root `package.json` at chosen project root | `nodejs` |
| Short label `FastAPI`; dependency / requirement name `fastapi` | `fastapi` |
| Short label `Ruby` | (no catalog match → `UNSUPPORTED_STACK`) |
| Short label `Java/Maven` | (no catalog match → `UNSUPPORTED_STACK`) |

Hard rules:

- `--runtime` and MCP `runtime` must equal some `data.runtimes[].runtime` from
  the catalog.
- **Never** pass a dependency name, display label, or framework brand as the
  catalog runtime (e.g. do not use a package name from `dependencies` as
  `--runtime`). Dependency name `fastapi` happens to match catalog id `fastapi`;
  any Node.js app maps to catalog id **`nodejs`**.
- Do not invent ids. If the catalog has no match, that is **unknown** — stop.

### Example — unsupported Ruby app

After inspecting `main.rb` you already know the stack is Ruby. After the
catalog has no match, stop with a message like:

> This looks like a **Ruby** application (`main.rb`). Network Solutions managed
> deployment currently supports **nodejs** and **fastapi** only, so I can't
> proceed. Supported runtimes: `nodejs`, `fastapi`.

### Example — container-only or Java repository

A repo with only a `Dockerfile` (or a Java/Maven service with `pom.xml`) has no
catalog match. Name what you saw, give the reason, list the catalog, and stop:

> This repository has only a `Dockerfile` and no recognized runtime manifest, so
> I can't identify a supported stack. Network Solutions managed deployment
> currently supports **nodejs** and **fastapi** only. Status:
> `UNSUPPORTED_STACK`.

Do **not** append anything like "add `package.json` or `requirements.txt` and I
can continue".

---

## Path A — Preferred: `netsol-cli` (when installed, ≥0.2.0)

Complete **inspect → catalog match → single catalog id** before planning.

### A0. Catalog match (required before plan)

Prefer obtaining the catalog so you see `data.runtimes` before declaring:

```bash
npx netsol mcp call deployment_list_supported_runtimes
```

(Or call MCP `deployment_list_supported_runtimes` when the CLI path is not used
for that step.)

Then map freeform findings to a catalog `runtime` id using **Catalog id vs
freeform**. On **unknown** or **multiple**, stop or ask — do **not** run
`capabilities plan` yet.

### A1. Plan — `npx netsol capabilities plan [path] --runtime <catalog-id> [--recipe-id <id>]`

Pass only the **catalog** runtime id from A0. `--recipe-id` is optional; when
omitted the CLI derives it from the catalog entry.

```bash
npx netsol capabilities plan [path] --runtime nodejs --recipe-id nodejs-npm-start
```

Parse the NDJSON `plan_ready` event. On success (`status: ok` or `pending`),
present fields from `data` (see **Presentation checklist**). On
`UNSUPPORTED_STACK` or `VALIDATION_ERROR`, stop — no best-effort deploy.

The CLI fetches the catalog again and re-validates your declaration, then calls
canonical gateway tools on your behalf. That internal fetch is fine and does
**not** replace A0: you must still match and pass a real catalog id (never a
dep or display name). You do not need to duplicate `deployment_recommend_plan`
manually when this command succeeds.

---

## Path B — Fallback: MCP tools (CLI unavailable)

Use only these planning tools via the configured `network-solutions-deploy` MCP server:

- `deployment_list_supported_runtimes`
- `deployment_recommend_plan`
- `deployment_validate_plan` (optional)
- `deployment_get_recipe_requirements` (optional)

### B1. Detect the runtime yourself (first)

Inspect the repository as described in **Detecting the runtime** above. Keep the
freeform label(s) and evidence list. Never authenticate, and never read `.env`,
keys, or credentials. Refuse secret access **only** when `.env` appears in the
repository inventory; otherwise do not add a secret-refusal step and do not set
secret refusal flags.

### B2. Fetch the catalog (second)

Call `deployment_list_supported_runtimes` and read `data.runtimes`: each entry
has `runtime`, `recipe`, `min_plan`, and build/start commands. Match your
freeform findings to these **catalog ids** (not dep names). On **unknown** or
**multiple**, follow the outcomes above — do not call `deployment_recommend_plan`
until there is exactly one chosen catalog runtime. Always list the catalog
after inspection, including when asking for a package root or runtime choice.

### B3. Recommend plan (declare a catalog ID)

```json
{"runtime": "nodejs", "recipe_id": "nodejs-npm-start"}
```

Call `deployment_recommend_plan` with the matched catalog runtime (and
optionally `recipe_id`). Record this planning step as **`recommend_plan`** —
never as `capabilities_plan` (CLI-only). You may include an optional `evidence_summary`
(manifest files and dependency names) — it is audit-only and never changes the
recommendation. Never send `project_signals`; never send `repo_path` in
production paths; never declare a freeform name or dep name that is not a
catalog `runtime` id.

### B4. Optional enrichment

- `deployment_validate_plan` with `deployment_plan_id` → `required_env_vars` (names only).
- `deployment_get_recipe_requirements` with `recipe_id` → required files and env-var **names** (never values).

---

## Result interpretation

| Result | Action |
| --- | --- |
| `status: requires_action` (or CLI `plan_ready` pending) | Valid plan exists — present checklist fields; save handles |
| Agent-side `unknown` (no catalog match) | Reuse step-1 stack + evidence; give reason; list catalog; stop; never call recommend/plan; never suggest adding detection files |
| `error_code: UNSUPPORTED_STACK` | Reuse step-1 detection if available; list `data.supported` if present; stop; never best-effort deploy; never suggest adding detection files |
| `error_code: VALIDATION_ERROR` | Your declaration was rejected (unknown recipe or runtime/recipe mismatch) — report the literal `VALIDATION_ERROR`, list catalog runtime ids to recheck, and keep the catalog runtime id you attempted in your declared runtime; do not retry blindly |
| Other errors | Report `summary` and `error_code`; retry once if `retryable` |

## Presentation checklist

### When invoked by Deploy to Network Solutions

Return plan, runtime, recipe, SKU, quote, evidence, commands, environment names,
and workflow handles to the calling deployment workflow **internally**. Do not
present this section's standalone plan summary. The deploy workflow's stricter
four-field output contract overrides the presentation rules below. The
user-facing message for deploy-internal handoff must be **generic only** — never
include runtime, recipe, SKU, quote reference, deployment plan id, workflow id,
or any other plan fact value. Example:

> A validated plan was returned internally to the calling deployment workflow;
> user-facing deployment output remains controlled by that workflow.

### On a successful plan

Before stopping, show the user:

- **Runtime** and **recipe** (and `recipe_id` when returned)
- **The evidence from local inspection** (manifest files, key dependency names)
- **Build/start commands** when returned
- **Plan name and SKU** (`data.product_requirement` or CLI `plan_ready` equivalent)
- **Quote reference and expiry** (`quote_reference`, `quote_expires_at`)
- **Required environment variable names** only (from validate/recipe tools)
- **`workflow_id`** and **`deployment_plan_id`** from `state_handles` or CLI `plan_ready` data

Finish with: "To proceed, purchase/confirm the hosting plan — the Deploy to
Network Solutions Skill continues from workflow `<workflow_id>`."

### On `unknown`

Show the **reused** freeform stack name and evidence from the first inspection
(or plainly what you saw when nothing was identifiable), state that it is not in
the supported catalog, and list catalog runtimes — the three parts in
**Reporting an unsupported stack**. Do not invent a plan, and do not suggest
creating `package.json`, `requirements.txt`, or any other detection file to make
the app supported.

Preserve `correlation_id` when returned for support.

## Hard boundaries

- **Read-only:** no authentication, checkout, provisioning, deployment, DNS
  changes, log reads, restarts, or rollbacks.
- **No secrets:** never read, request, store, or output secret values or credentials.
- **No permission to deploy:** purchase and the Deploy Skill's one combined
  Dockerfile/package/deployment approval happen outside this read-only Skill.
- **No invented state:** never guess runtime, recipe, SKU, quote, or handles.
  Declare only catalog `runtime` ids after match — never freeform labels or
  dependency names as `--runtime`. On **unknown** stop (reuse prior detection);
  on **multiple** ask the user.
- **No manifest scaffolding:** on an unsupported or unidentifiable stack, report
  what you found, the reason, and the supported runtimes — never suggest adding,
  creating, or renaming a detection file (`package.json`, `requirements.txt`,
  `pyproject.toml`, …) to make the app detectable.
- **Stop on missing tools:** if required planning tools are unavailable in both
  MCP and CLI paths, stop with clear setup instructions naming **MCP** and
  **`netsol-cli`**; make no planning tool calls.
