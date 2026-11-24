# Linq.es Official MCP Server (`https://linq.es/mcp`)

[![mcptry](https://mcptry.com/badge/io.github.giulianobr/linqes-mcp)](https://mcptry.com/servers/io.github.giulianobr/linqes-mcp)
[![Linq.es MCP connector](https://glama.ai/mcp/connectors/io.github.giulianobr/linqes-mcp/badges/score.svg)](https://glama.ai/mcp/connectors/io.github.giulianobr/linqes-mcp)

Official Model Context Protocol (MCP) server manifests, IDE plugins, and agent skills for **[Linq.es](https://linq.es)** - the server-rendered Link-in-Bio, `/s/` URL shortener, QR code, vCard, and lead capture platform.

- **Remote MCP Endpoint:** `https://linq.es/mcp` (`streamable-http`, JSON-RPC 2.0, `protocolVersion: "2025-03-26"`)
- **Documentation:** [https://linq.es/integrations/mcp](https://linq.es/integrations/mcp)
- **Authentication:** Personal API Key (`Bearer lnq_live_...`) generated in Linq.es Account Settings (`/admin/account-settings/:profile`)

No local Node.js runtime, Python environment, or Docker container is required. Connect any MCP-compatible client directly over HTTPS.

---

## Quickstart Configuration

### Claude Code CLI
```bash
claude mcp add --transport http linqes https://linq.es/mcp \
  --header "Authorization: Bearer ${LINQES_API_KEY}"
```

### Claude Desktop / `.mcp.json`
```json
{
  "mcpServers": {
    "linqes": {
      "type": "http",
      "url": "https://linq.es/mcp",
      "headers": {
        "Authorization": "Bearer ${LINQES_API_KEY}"
      }
    }
  }
}
```

### Cursor (`~/.cursor/mcp.json` or `.cursor/mcp.json`)
```json
{
  "mcpServers": {
    "linqes": {
      "url": "https://linq.es/mcp",
      "headers": {
        "Authorization": "Bearer lnq_live_YOUR_PERSONAL_API_KEY"
      }
    }
  }
}
```

### Gemini CLI (`~/.gemini/settings.json`)
```json
{
  "mcpServers": {
    "linqes": {
      "httpUrl": "https://linq.es/mcp",
      "headers": {
        "Authorization": "Bearer $LINQES_API_KEY"
      }
    }
  }
}
```

### OpenAI Codex CLI (`~/.codex/config.toml`)
```toml
[mcp_servers.linqes]
url = "https://linq.es/mcp"
bearer_token_env_var = "LINQES_API_KEY"
```

---

## Available MCP Tools (10 Tools)

| Tool | Description |
|---|---|
| `list_profiles` | List all Link-in-Bio profiles owned by the authenticated account (`profileName`, `publicUrl`, `theme`, `viewCount`). |
| `get_profile_analytics` | Retrieve 7-to-90-day views, link clicks, CTR, top referrers, countries, devices, and daily time series. |
| `list_profile_links` | List every link card and social icon on a profile, including click counts, schedules, and device URLs. |
| `add_profile_link` | Add a new link card or social icon with optional badges, iOS/Android routing, countdowns, and RFC3339 schedules. |
| `update_profile_link` | Update title, URL, active visibility, or routing rules on an existing profile link and purge edge cache. |
| `delete_profile_link` | Delete a link by key and immediately invalidate cached public profile HTML. |
| `import_bio_links` | Import public links from an existing Linktree, Beacons, or external bio URL into a Linq.es profile. |
| `list_shortlinks` | List `/s/:code` branded shortlinks, click totals, device overrides, and GA4 UTM parameters. |
| `create_shortlink` | Create a `/s/:code` shortlink with custom vanity slug, iOS/Android redirects, and GA4 UTM tags. |
| `list_leads` | Retrieve recent contacts (Name, Email, WhatsApp/Phone, timestamp) captured through inline Lead Magnet forms. |

Full tool schemas and workflows are documented in [`skills/linqes-mcp/SKILL.md`](skills/linqes-mcp/SKILL.md) and at [https://linq.es/integrations/mcp](https://linq.es/integrations/mcp).

---

## Repository Manifests

```text
linqes-mcp/
├── .claude-plugin/
│   └── plugin.json          # Claude Code Plugin Directory manifest
├── .cursor-plugin/
│   └── plugin.json          # Cursor Marketplace plugin manifest
├── .mcp.json                # Shared Streamable HTTP MCP config (${LINQES_API_KEY})
├── gemini-extension.json    # Gemini CLI Extensions gallery manifest
├── glama.json               # Glama maintainer verification & awesome-mcp-servers CI
├── server.json              # Official MCP Registry manifest (io.github.giulianobr/linqes-mcp)
├── skills/
│   └── linqes-mcp/
│       └── SKILL.md         # Agent skill covering lnq_live_ auth + all 10 MCP tools
├── LICENSE                  # MIT License
└── README.md                # Setup & registry reference
```

### Publishing / Updating the Official MCP Registry

```bash
brew install mcp-publisher
mcp-publisher login github
mcp-publisher publish
```

## Support & Contact

- **Website:** [https://linq.es](https://linq.es)
- **MCP Documentation:** [https://linq.es/integrations/mcp](https://linq.es/integrations/mcp)
- **Help Center:** [https://linq.es/help](https://linq.es/help)
- **Contact:** `hello@linq.es`
