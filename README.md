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

# About NuBerea

NuBerea helps pastors, students, and believers study Scripture more deeply using AI while remaining rooted in biblical text and historic Christian theology.

[https://nuberea.com](https://nuberea.com)
