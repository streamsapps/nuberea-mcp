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

Using MCP, assistants such as ChatGPT, Claude, Cursor, Claude Desktop, or custom agents can interact with NuBerea’s research capabilities as structured tools.

Full documentation: [https://nuberea.com/docs/mcp/](https://nuberea.com/docs/mcp/)

---

# Quick Start (any MCP client)

This is the universal path for ChatGPT, Cursor, Claude Desktop, custom agents, and any other MCP-compatible client.

### 1. Create an account

Sign in or create a NuBerea account at [https://nuberea.com/login](https://nuberea.com/login).

### 2. Add NuBerea to your MCP client

Point your client at the NuBerea MCP server URL:

```
https://auth.aws-dev.streamsappsgslbex.com/mcp
```

Example MCP configuration:

```json
{
  "mcpServers": {
    "nuberea": {
      "url": "https://auth.aws-dev.streamsappsgslbex.com/mcp"
    }
  }
}
```

### 3. Authenticate

NuBerea MCP uses token-based authentication with refresh tokens to maintain secure, long-lived sessions. Your client will prompt you to sign in on first use. See the [docs](https://nuberea.com/docs/mcp/) for client-specific setup.

---

# Claude Code plugin

If you use **Claude Code**, NuBerea ships as a first-class plugin.

This repository follows Claude Code's marketplace layout:

* `.claude-plugin/marketplace.json` defines the marketplace catalog.
* `.claude-plugin/plugin.json` defines the NuBerea plugin.

## Install

Choose one of the marketplace sources below. The **Anthropic Community marketplace** is recommended for most users; use the **GitHub** source if you want preview builds direct from this repo.

| | Anthropic Community (recommended) | GitHub (this repo) |
|---|---|---|
| **Add marketplace** | `claude plugin marketplace add anthropics/claude-plugins-community` | `claude plugin marketplace add streamsapps/nuberea-mcp` |
| **Install** | `claude plugin install nuberea@claude-community` | `claude plugin install nuberea@nuberea` |
| **Update** | `claude plugin marketplace update claude-community`<br>`claude plugin update nuberea@claude-community` | `claude plugin marketplace update nuberea`<br>`claude plugin update nuberea@nuberea` |
| **Uninstall** | `claude plugin uninstall nuberea@claude-community`<br>`claude plugin marketplace remove claude-community` | `claude plugin uninstall nuberea@nuberea`<br>`claude plugin marketplace remove nuberea` |

> To scope a marketplace to a single repository instead of your user profile, append `--scope project` to the `marketplace add` command.

Restart Claude Code after updating so the new plugin version is loaded.

## Use

Start Claude Code:

```bash
claude
```

Authenticate NuBerea (first run only):

```text
/mcp
```

Select `nuberea` and complete the authentication flow.

Try a prompt such as:

```text
Use Nuberea to look up John 1:1 in Greek and summarize the lexical notes.
```

## Troubleshooting

List configured marketplaces:

```bash
claude plugin marketplace list
```

List installed plugins:

```bash
claude plugin list
```

Check whether the MCP server is connected:

```bash
claude mcp list
```

Or inside Claude Code, run `/mcp`.

If NuBerea shows `Needs authentication`, run `/mcp`, select `nuberea`, and complete authentication. If authentication succeeds but reconnection fails, restart Claude Code.

---

# Example Use Cases

NuBerea MCP enables AI assistants to perform scholarly biblical research on demand. Try prompts like:

### Greek & Hebrew Word Studies
* *"Look up the Greek word ἀγάπη (agape) in the LSJ lexicon and show me every occurrence in 1 John."*
* *"What does the Hebrew word חֶסֶד (chesed) mean in BDB, and where does it appear in the Psalms?"*
* *"Give me the morphology of every word in John 1:1 from the Macula Greek dataset."*

### Exegesis & Verse Analysis
* *"Show me John 3:16 with word-by-word Greek morphology, lemmas, and Strong's numbers."*
* *"Compare the Hebrew of Genesis 1:1 with the Septuagint Greek translation."*
* *"What manuscripts contain Mark 16:9–20, and how do their transcriptions differ?"*

### Cross-References & Theological Themes
* *"Find Old Testament cross-references for Romans 3:23 and explain the theological connection."*
* *"Trace the theme of 'covenant' across the Pentateuch using cross-reference data."*
* *"Show me every NT quotation of Isaiah 53."*

### Sermon & Study Preparation
* *"Help me prepare a sermon on Philippians 2:5–11 — give me the Greek structure, key word studies, and historic interpretations."*
* *"Build a Bible study outline on the parables of the kingdom in Matthew 13."*
* *"Summarize the textual variants in 1 Corinthians 13 across the major manuscript traditions."*

### Manuscript & Textual Criticism
* *"Show me the Dead Sea Scrolls readings for Isaiah 7:14."*
* *"Pull the CNTR transcription data for Codex Sinaiticus on John 1."*
* *"Compare Aland's synopsis entries for the resurrection accounts across the Synoptic Gospels."*

---

# Documentation

Full documentation is available at [https://nuberea.com/docs/mcp/](https://nuberea.com/docs/mcp/), including:

* MCP server configuration
* Authentication setup
* Tool definitions
* Client integration examples

---

# Contributing

Contributor and release workflow documentation lives in [CONTRIBUTING.md](CONTRIBUTING.md).

---

# About NuBerea

NuBerea helps pastors, students, and believers study Scripture more deeply using AI while remaining rooted in biblical text and historic Christian theology.

[https://nuberea.com](https://nuberea.com)

NuBerea is published to the [official MCP Registry](https://registry.modelcontextprotocol.io) as `com.streamsapps/nuberea` ([API](https://registry.modelcontextprotocol.io/v0.1/servers?search=com.streamsapps/nuberea)).
