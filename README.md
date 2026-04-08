# NuBerea AI

Tools for building **AI-powered theological study experiences** using NuBerea.

This repository contains resources for integrating NuBerea’s biblical research tools with AI assistants, developer frameworks, and the **Model Context Protocol (MCP)**.

---

# What is NuBerea?

NuBerea is an AI-powered theological assistant designed to help users explore Scripture with deeper insight.

It provides tools for:

* Biblical cross-references
* Greek and Hebrew word insights
* Scripture-grounded theological explanations
* AI-assisted sermon preparation and research

Learn more at **[https://nuberea.com](https://nuberea.com)**

---

# Model Context Protocol (MCP)

NuBerea provides an MCP server that allows AI clients to securely access NuBerea’s theological tools.

Using MCP, assistants such as ChatGPT, Claude, or custom agents can interact with NuBerea’s research capabilities as structured tools.

Documentation:
[https://nuberea.com/docs/mcp/](https://nuberea.com/docs/mcp/)

---

# Quick Start

### 1. Create an account

Create or sign in to a NuBerea account.

```
https://nuberea.com/login
```

---

### 2. Configure your MCP client

Connect your MCP client to the NuBerea MCP server.

```
https://auth.aws-dev.streamsappsgslbex.com/mcp
```

---

### 3. Example MCP configuration

```json
{
  "mcpServers": {
    "nuberea": {
      "url": "https://auth.aws-dev.streamsappsgslbex.com/mcp"
    }
  }
}
```

Once connected, your AI client can call NuBerea tools for theological study and biblical research.

---

# Example Use Cases

NuBerea MCP enables AI systems to support:

* Biblical exegesis
* Theological research
* Sermon preparation
* Scripture cross-reference exploration
* AI-assisted Bible study

---

# Authentication

NuBerea MCP uses token-based authentication with refresh tokens to maintain secure, long-lived sessions for AI clients.

See the documentation for setup instructions.

[https://nuberea.com/docs/mcp/](https://nuberea.com/docs/mcp/)

---

# Documentation

Full documentation is available at:

[https://nuberea.com/docs/mcp/](https://nuberea.com/docs/mcp/)

The docs include:

* MCP server configuration
* Authentication setup
* Tool definitions
* Client integration examples

---

# MCP Registry

NuBerea is published to the [official MCP Registry](https://registry.modelcontextprotocol.io) as `com.streamsapps/nuberea`.

**Registry API:**
```
https://registry.modelcontextprotocol.io/v0.1/servers?search=com.streamsapps/nuberea
```

## Publishing a New Version

Publishing is automated via GitHub Actions. To release a new version:

1. Update `server.json` if needed (description, icons, etc.)
2. Tag and push:

```bash
git tag v1.0.2
git push origin v1.0.2
```

The workflow (`.github/workflows/publish-mcp.yml`) will:
- Extract the version from the tag (e.g. `v1.0.2` → `1.0.2`)
- Inject it into `server.json`
- Authenticate via GitHub OIDC (no secrets needed)
- Validate and publish to the MCP Registry

**Authentication:** Uses DNS verification against `streamsapps.com` (ECDSA P-384). The private key is stored as the `MCP_PRIVATE_KEY` GitHub secret. The corresponding public key is published as a TXT record on `streamsapps.com`.

**Note:** The `server.json` version in the repo can stay at any value; the workflow overwrites it from the git tag at publish time.

---

# About NuBerea

NuBerea helps pastors, students, and believers study Scripture more deeply using AI while remaining rooted in biblical text and historic Christian theology.

[https://nuberea.com](https://nuberea.com)
