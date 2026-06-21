# MCP Tools

The `microsoft-learn` MCP server gives agents direct access to official Power Apps documentation without leaving the conversation. Configure it in `.vscode/mcp.json` before starting work.

---

## Available tools

### microsoft_docs_search

Search official Microsoft Learn documentation. Returns up to 10 content chunks with title, URL, and excerpt.

Use this first for any question about Power Apps code apps, pac CLI behaviour, connectors, or SDK APIs.

```
microsoft_docs_search("Power Apps code apps <topic>")
```

Examples:
```
microsoft_docs_search("Power Apps code apps connectors data source")
microsoft_docs_search("Power Apps code apps getContext user identity")
microsoft_docs_search("pac code init environment")
```

### microsoft_docs_fetch

Fetch a full Microsoft Learn page as Markdown. Use when search results are incomplete or you need the full content of a specific page.

```
microsoft_docs_fetch("https://learn.microsoft.com/en-us/power-apps/developer/code-apps/<page>")
```

Key pages to fetch:

| Page slug | When to fetch |
|---|---|
| `overview` | Understanding what code apps are and their constraints |
| `architecture` | App lifecycle, build pipeline, host model |
| `how-to/npm-quickstart` | Scaffolding and first deploy steps |
| `how-to/connect-to-data` | General connector and data source patterns |
| `how-to/connect-to-dataverse` | Dataverse-specific connection and OData patterns |
| `system-limits-configuration` | Size limits, timeouts, allowed APIs |

### microsoft_code_sample_search

Search for official code samples and snippets on Microsoft Learn.

```
microsoft_code_sample_search("Power Apps code apps <topic>")
```

Use when you need a working example of a specific pattern (connector call, telemetry, auth context).

---

## When to use these tools

- Before writing any connector or SDK code: search first, then write
- When a `pac` command fails with an unexpected error: fetch the relevant how-to page
- When unsure about system limits (bundle size, API allow-list, CSP restrictions): fetch `system-limits-configuration`
- When a generated service file does something unexpected: search for the connector's documented behaviour

---

## Setup

The MCP server is configured per-project in `.vscode/mcp.json`. If the file does not exist in the project root, create it:

```json
{
  "servers": {
    "microsoft-learn": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@microsoft/learn-mcp-server"]
    }
  }
}
```

Restart the Claude Code session after adding the file.
