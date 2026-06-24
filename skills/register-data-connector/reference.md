# NuBerea catalog — SDK CLI reference

Every command runs through the NuBerea SDK CLI, published on npm as
`@nuberea/sdk` (catalog commands require **v0.0.8+**). Run it with
`npx -y -p @nuberea/sdk@latest nuberea <command>`, or a bare `nuberea` if
v0.0.8+ is installed globally. The CLI handles OAuth (`nuberea login`) and
reuses the cached token; it talks to the `/v1` catalog control plane on the
NuBerea MCP host. Add `--json` for raw JSON, `--base-url <url>` to target a
non-default host.

The same operations are available programmatically via `client.catalog.*` in the
SDK (`import { NuBerea } from '@nuberea/sdk'`).

## Global options

| Flag / env | Effect |
|------------|--------|
| `--json` | Emit raw JSON instead of pretty-printed |
| `--base-url <url>` / `NUBEREA_BASE_URL` | Override the MCP host (default: dev) |
| `--token <t>` / `NUBEREA_ACCESS_TOKEN` | Use a pre-set token (CI; skip `login`) |

## Tenants

| Command | SDK method | Notes |
|---------|-----------|-------|
| `catalog tenants` | `catalog.listTenants()` | Owned tenants + status counts |
| `catalog tenant-create "<name>" [--tier <t>]` | `catalog.createTenant(name, tier?)` | Returns `{ tenantId, … }` |
| `catalog tenant-delete <tenantId>` | `catalog.deleteTenant(id)` | Cascade-deletes trust/connectors/tools |

## AWS trust (OIDC) — `glue_athena` only

Secretless: the customer role trusts NuBerea's OIDC issuer via
`sts:AssumeRoleWithWebIdentity`. NuBerea stores no key.

| Command | SDK method | Returns |
|---------|-----------|---------|
| `catalog trust-aws <tenantId>` | `catalog.createAwsTrust(id)` | `audience`, `subject`, `oidcTemplate` |
| `catalog trust-role <tenantId> <roleArn>` | `catalog.setTrustRoleArn(id, arn)` | `{ status: "pending" }` |
| `catalog trust-verify <tenantId>` | `catalog.verifyAwsTrust(id)` | `{ status: "active" \| "error", reason?, assumedRoleArn? }` |

`trust-aws` returns the IAM artifacts the customer applies in **their** AWS
account:

```json
{
  "tenantId": "t_…",
  "status": "pending",
  "audience": "<aud NuBerea mints>",
  "subject": "tenant:t_…:workload:mcp",
  "oidcTemplate": {
    "mode": "oidc",
    "oidcProvider": { "url": "<issuer>", "clientIdList": ["<aud>"], "thumbprintNote": "…" },
    "trustPolicy":      { "Version": "2012-10-17", "Statement": [ … ] },
    "permissionPolicy": { "Version": "2012-10-17", "Statement": [ … ] }
  }
}
```

The customer: (1) creates an IAM OIDC identity provider for `oidcProvider.url`,
(2) creates a read-only IAM role whose trust policy is `trustPolicy` and whose
permission policy is `permissionPolicy` (least-privilege Glue/Athena/S3; Athena
staging stays on the customer's own bucket, preserving zero-copy). `roleArn`
must match `^arn:aws:iam::\d{12}:role/.+`.

## Connectors

| Command | SDK method |
|---------|-----------|
| `catalog connectors <tenantId>` | `catalog.listConnectors(id)` |
| `catalog add-hf <tenantId> <repo> <table> [files...] [--auth gated] [--revision <ref>] [--hf-resource <user>]` | `catalog.createHfConnector(id, …)` |
| `catalog add-glue <tenantId> <region> <workgroup> [--databases a,b] [--tables a.x,b.y]` | `catalog.createGlueConnector(id, …)` |
| `catalog validate <tenantId> <connectorId>` | `catalog.validateConnector(id, cid)` |
| `catalog connector-delete <tenantId> <connectorId>` | `catalog.deleteConnector(id, cid)` |

### Hugging Face (`add-hf`)

- `repo` = `owner/name`; `table` = the logical name tool SQL selects `FROM`.
- `files...` = literal Parquet path(s)/glob under the repo on the chosen
  revision. Don't trust `/api/datasets/<repo>/parquet` (it normalizes the last
  segment). A glob like `default/train/*.parquet` avoids guessing shard names
  (`0.parquet` vs `0000.parquet` vs `train-00000-of-00001.parquet`).
- `--auth gated` uses NuBerea's issuer as a Hugging Face **Trusted Publisher**
  (CI/CD Access) to mint a short-lived, read-only token per query;
  `--hf-resource` is the HF account that exchange is scoped to. **Private** repos
  are not supported.

### AWS Glue + Athena (`add-glue`)

- Requires an **active trust** first.
- `--databases` = Glue databases in scope; `--tables` = optional finer
  `db.table` allowlist (comma-separated).

Both create with `status:"pending"`; run `validate` to activate. `validate`
returns a `ValidateResult`:

```json
{
  "status": "active",
  "tables":  [ { "database": "sales", "table": "orders", "columns": [ { "name": "region", "type": "string" } ] } ],
  "columns": ["pair_id", "source_ref", "…"],
  "sample":  [ { "…": "…" } ]
}
```

(`tables` for `glue_athena`; `columns` + `sample` for `hf_dataset`.)

## Tools

A tool is a parameterized, SELECT-only SQL template. The model fills typed
parameters at call time; it never writes raw SQL.

| Command | SDK method |
|---------|-----------|
| `catalog tools <tenantId>` | `catalog.listTools(id)` |
| `catalog suggest-tools <tenantId> <connectorId> [--max <n>] [--prompt "<text>"]` | `catalog.suggestTools(id, cid, …)` |
| `catalog register-tool <tenantId> '<json>'` | `catalog.registerTool(id, input)` |

`suggest-tools` returns `{ connectorId, suggestions: [ draft… ], generatedAt }`
where each draft already matches the `register-tool` JSON shape (plus a
`rationale`). Register the ones the user approves.

`register-tool` JSON (the `ToolRegistrationInput` shape):

```json
{
  "name": "orders_by_region",
  "connectorId": "c_…",
  "description": "Orders for a region",
  "inputSchema": {
    "type": "object",
    "properties": { "region": { "type": "string", "description": "US region code" } },
    "required": ["region"]
  },
  "sqlTemplate": "SELECT region, total FROM sales.orders WHERE region = :region",
  "paramBindings": { "region": { "sqlParam": "region", "type": "string" } },
  "rowLimit": 100
}
```

SQL safety rules (enforced server-side — the injection boundary):

- **SELECT-only**, single statement, no comments. Referenced tables must be
  within the connector's scope (Glue) or the single `tableName` (HF).
- Named params (`:region`) bind as typed literals; the row limit is injected by
  the server. HF mode additionally denies functions like `read_parquet` /
  `read_csv` / `glob` / `install` / `load` / `pragma` and any external URI.
- `paramBindings[*].type` ∈ `string | number | integer | boolean | date`;
  optional `default` and `max` (caps numeric/limit params).

Toggle / remove (SDK only; no dedicated CLI subcommand): `catalog.setToolStatus(id, name, enabled)` and `catalog.deleteTool(id, name)`.

## Activation rule

A tenant tool is exposed through the NuBerea MCP server only when **all** of
these are active: the tool (`status:"active"`), its connector
(`status:"active"`), and — for `glue_athena` — the AWS trust. HF connectors need
no trust. After registering, the user should reconnect/refresh their MCP client
so the new tool appears.
