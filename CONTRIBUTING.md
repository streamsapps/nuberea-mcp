# Contributing

This repository publishes NuBerea as both:

* an MCP server definition via `server.json`
* a Claude Code marketplace with the NuBerea plugin under `plugins/nuberea`

## Repository layout

Key files and directories:

* `server.json` defines the MCP registry metadata.
* `.claude-plugin/marketplace.json` defines the Claude Code marketplace.
* `plugins/nuberea/.claude-plugin/plugin.json` defines the NuBerea Claude plugin.
* `README.md` is the user-facing overview and installation guide.

## Local validation

Before opening a change, validate the Claude marketplace from the repository root:

```bash
claude plugin validate .
```

If you want to test the marketplace from a local clone, add and install it locally:

```bash
claude plugin marketplace add ./
claude plugin install nuberea@nuberea
```

To refresh a local install after changes:

```bash
claude plugin marketplace update nuberea
claude plugin update nuberea
```

Restart Claude Code after updating so the new plugin version is loaded.

To remove a local test install when you are done:

```bash
claude plugin uninstall nuberea
claude plugin marketplace remove nuberea
```

## Editing guidelines

When changing marketplace or plugin metadata, keep these files aligned:

* `LICENSE` and any `license` fields in Claude manifests
* `server.json` and published registry details in the README
* `.claude-plugin/marketplace.json` and `plugins/nuberea/.claude-plugin/plugin.json`

Use SPDX identifiers in manifest `license` fields. This repository is licensed as `Apache-2.0`.

## Releasing

NuBerea is published to the MCP Registry as `com.streamsapps/nuberea`.

Publishing is automated via GitHub Actions. To release a new version:

1. Update `server.json` if needed.
2. Validate the Claude marketplace with `claude plugin validate .`.
3. Tag and push the release:

```bash
git tag v1.0.2
git push origin v1.0.2
```

The workflow in `.github/workflows/publish-mcp.yml` will:

* extract the version from the tag
* inject it into `server.json`
* authenticate via GitHub OIDC
* validate and publish to the MCP Registry

Authentication uses DNS verification against `streamsapps.com`. Keep the publish workflow credentials configured in GitHub Actions.

The `server.json` version committed in the repository can remain unchanged between releases because the workflow overwrites it from the git tag at publish time.