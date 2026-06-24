---
name: register-data-connector
description: >-
  Register a bring-your-own-data (BYO-data) connector on a user's NuBerea
  account so their own data becomes queryable through the NuBerea MCP server.
  Use this when a user asks to "connect my data", "add a data source", "register
  a connector", "expose my Hugging Face dataset / AWS data catalog to NuBerea",
  or to create the tenant → trust → connector → tool chain. This skill
  drives the NuBerea SDK CLI (`@nuberea/sdk`), which handles OAuth sign-in and
  the catalog control plane. Supports two connector kinds: hf_dataset (Hugging
  Face Parquet datasets, public or gated) and glue_athena (AWS Glue Data Catalog
  + Athena over the user's own S3, via secretless OIDC trust).
---

# Register a NuBerea data connector

This skill registers a **BYO-data connector** on the user's NuBerea account
using the **NuBerea SDK** (`@nuberea/sdk`). The SDK handles OAuth sign-in, token
storage/refresh, and the federated-catalog control plane, so you never deal with
raw tokens or HTTP. The user's data stays in their account; NuBerea queries it
zero-copy and **never stores customer secrets**.

The object model is a chain. A **connector** always belongs to a **tenant** (the
org unit owned by the signed-in user), and queryable **tools** are registered
against the connector:

```
tenant ──▶ (glue_athena only) AWS trust ──▶ connector ──▶ tool(s)
```

| Kind | Source | Trust step? |
|------|--------|-------------|
| `hf_dataset` | Hugging Face dataset Parquet (public or gated) | No |
| `glue_athena` | AWS Glue Data Catalog + Athena over the user's S3 | Yes (OIDC) |

## Setup: the `nuberea` CLI

All steps run through the SDK CLI. Resolve the command once and reuse it:

- If `nuberea` is on `PATH` (installed via `npm i -g @nuberea/sdk`), use it.
- Otherwise use `npx -y -p @nuberea/sdk nuberea …` (no install needed).

```bash
# pick whichever resolves; examples below say `nuberea`
command -v nuberea >/dev/null 2>&1 && echo "nuberea" || echo "npx -y -p @nuberea/sdk nuberea"
```

Then make sure the user is signed in (opens a browser the first time; the token
is cached in the OS keychain):

```bash
nuberea status   # ✅ Authenticated  /  ❌ Not authenticated
nuberea login    # only if not authenticated
```

To target a non-default host, pass `--base-url <url>` on any command or export
`NUBEREA_BASE_URL`. For CI/non-interactive use, a short-lived token can be
supplied via `NUBEREA_ACCESS_TOKEN` instead of `nuberea login`.

> Prefer a GUI? The same flow exists in the NuBerea web app under
> **Settings → Data connections**. This skill is the conversational/CLI path.

Add `--json` to any `catalog` command for raw JSON (handy for parsing IDs).
Full subcommand + payload details are in [reference.md](reference.md) — read it
before registering a tool.

## Workflow

Confirm connector details with the user before creating anything, and **echo
back the IDs** (`tenantId`, `connectorId`) the CLI returns — they are
server-generated and needed for later steps.

### Step 0 — Find or create the tenant

```bash
nuberea catalog tenants
# none yet? create one:
nuberea catalog tenant-create "<org or project name>"
```

Capture the `tenantId` from the output.

### Step 1 — Register the connector

**Hugging Face dataset** (no trust needed):

```bash
nuberea catalog add-hf <tenantId> <owner>/<name> <logical_table_name> "default/train/*.parquet"
# gated repos: append  --auth gated --hf-resource <hf-username>  (optionally --revision <ref>)
```

- The trailing args after the table name are the Parquet path(s)/glob under the
  repo. A glob like `default/train/*.parquet` is the safest default (DuckDB
  globs `hf://` paths). For `gated` repos see reference.md.

**AWS Glue + Athena** needs an active trust first (Step 1a–1c), then:

```bash
nuberea catalog add-glue <tenantId> <aws-region> <athena-workgroup> \
  --databases <glue_db> --tables <glue_db>.<table>
```

#### Step 1a–1c — AWS OIDC trust (glue_athena only)

```bash
nuberea catalog trust-aws <tenantId>     # returns audience, subject, and an OIDC IAM template
# → user applies the OIDC provider + read-only role in THEIR aws account
nuberea catalog trust-role <tenantId> arn:aws:iam::<acct>:role/<name>
nuberea catalog trust-verify <tenantId>  # status:"active" = NuBerea assumed the role
```

A failed verify returns a `reason`; fix the role/policy and retry.

### Step 2 — Validate the connector

```bash
nuberea catalog validate <tenantId> <connectorId>
```

`status:"active"` confirms access (returns discovered `tables`/schema for Glue,
or `columns` + a `sample` for HF). A non-active result includes a `reason` —
relay it, help the user fix scope/files/trust, then re-validate.

### Step 3 — Register tool(s) (optional but usually wanted)

A tool is a **parameterized, SELECT-only SQL template** the model later fills
with typed parameters — it never authors raw SQL at call time. Either:

```bash
# let NuBerea draft tools from the connector schema, then register approved ones
nuberea catalog suggest-tools <tenantId> <connectorId> --max 5

# or author one directly (see reference.md for the JSON shape + SQL rules)
nuberea catalog register-tool <tenantId> '{"name":"orders_by_region", ... }'
```

### Step 4 — Confirm

```bash
nuberea catalog connectors <tenantId>
nuberea catalog tools <tenantId>
```

A registered tool is exposed through the NuBerea MCP server only when the
**tool, its connector, and (for Glue) the trust are all active**. Tell the user
to reconnect / refresh their MCP client so the new tool appears.

## Guardrails

- **Never** invent `tenantId` / `connectorId` values — always use the IDs the
  CLI returns.
- Auth is handled by the SDK (`nuberea login`); never ask the user to paste a
  token into a command. For automation, set `NUBEREA_ACCESS_TOKEN` in the
  environment.
- NuBerea stores **no** customer secrets. For Glue, trust is OIDC role
  assumption; for gated HF, a short-lived token is minted per call. Private HF
  repos are intentionally unsupported.
- Tool SQL is SELECT-only, single-statement, scoped to the connector's tables,
  with a server-enforced row limit — the security boundary. Don't try to bypass it.
- Deletes are destructive: `catalog connector-delete` is rejected while any tool
  is still bound (delete the tools first). Confirm with the user before deleting.
