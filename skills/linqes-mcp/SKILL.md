---
name: linqes-mcp
description: Connect to the official Linq.es remote Streamable HTTP MCP server at https://linq.es/mcp to manage Link-in-Bio profiles, bio links, /s/ branded shortlinks with GA4 UTM parameters, 90-day click analytics, and captured leads. Use when automating profile updates, importing external Linktree or bio pages, generating campaign shortlinks, or querying conversion metrics and leads.
metadata:
  author: Team Linq.es
  version: 1.0.0
  homepage: https://linq.es/integrations/mcp
  endpoint: https://linq.es/mcp
  protocol_version: "2025-03-26"
---

# Linq.es Official MCP Server Skill

Linq.es hosts a remote Model Context Protocol (MCP) server over Streamable HTTP JSON-RPC 2.0 (`protocolVersion: "2025-03-26"`) at `https://linq.es/mcp`. No local Node.js package, Python runtime, or Docker container is required.

## 1. Authentication & Personal API Key Setup

MCP server access requires a Linq.es Pro account and a Personal API Key:

1. Sign in to Linq.es and open **Account Settings** (`/admin/account-settings/:profile`).
2. Generate a **Personal API Key**.
3. Copy the token immediately. Every valid token uses the `lnq_live_` prefix followed by 48 hexadecimal characters (57 characters total, for example `lnq_live_0123456789abcdef0123456789abcdef0123456789abcdef`). Do not use `linq_` as the prefix; the server strictly validates `lnq_live_`.
4. Store the key in your environment as `LINQES_API_KEY` and pass it in the HTTP header:
   ```text
   Authorization: Bearer lnq_live_YOUR_PERSONAL_API_KEY
   ```

Raw keys are shown once at creation time and stored server-side only as SHA-256 hashes. Keys can be rotated or revoked at any time in `/admin/account-settings/:profile`. Never hardcode or commit a live `lnq_live_` key into version control.

## 2. Client Configuration (Claude, Cursor, Codex, Windsurf, Gemini CLI)

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

### OpenAI Codex CLI (`~/.codex/config.toml`)
```toml
[mcp_servers.linqes]
url = "https://linq.es/mcp"
bearer_token_env_var = "LINQES_API_KEY"
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

### Windsurf (`~/.codeium/windsurf/mcp_config.json`)
```json
{
  "mcpServers": {
    "linqes": {
      "serverUrl": "https://linq.es/mcp",
      "headers": {
        "Authorization": "Bearer lnq_live_YOUR_PERSONAL_API_KEY"
      }
    }
  }
}
```

## 3. Connection Verification

Inspect public server discovery metadata (no token required):

```bash
curl -sS https://linq.es/mcp
```

Verify authenticated tool listing over JSON-RPC 2.0:

```bash
curl -sS -X POST https://linq.es/mcp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer ${LINQES_API_KEY}" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## 4. Complete Reference: All 10 Linq.es MCP Tools

Always call `list_profiles` first when the user has not specified an exact profile handle (`profile`), since a Pro account can own up to 5 independent profiles.

### 1. `list_profiles`
Lists all Link-in-Bio profiles owned by the authenticated account, returning `profileName`, `name`, `headline`, `theme`, `listedInExplore`, `publicUrl`, and `viewCount`.
- **Required arguments:** none (`{}`)

### 2. `get_profile_analytics`
Returns aggregated `totalViews`, `totalClicks`, `ctrPercent`, `referrers`, `countries`, `devices`, and daily time-series breakdown (`dailySeries`) for a profile.
- **Required arguments:**
  - `profile` (string): Profile handle (for example `"alex"`).
- **Optional arguments:**
  - `days` (integer): Lookback window in days (default `7`, max `90`).

### 3. `list_profile_links`
Lists every link on a Link-in-Bio profile, including `key`, `title`, `url`, `active`, `isSocial`, click count, schedule timestamps, and device routing URLs.
- **Required arguments:**
  - `profile` (string): Profile handle.

### 4. `add_profile_link`
Creates a new link card or header social icon on a profile and purges the edge/profile cache immediately (up to 100 links per profile on Pro).
- **Required arguments:**
  - `profile` (string): Target profile handle.
  - `title` (string): Button display label.
  - `url` (string): Default `http://` or `https://` destination URL.
- **Optional arguments:**
  - `isSocial` (boolean): Set `true` to render as a header social icon.
  - `badgeText` (string): Highlight pill label (for example `"NEW"`).
  - `iosUrl` (string): iOS App Store or deep-link destination override.
  - `androidUrl` (string): Android Google Play or deep-link destination override.
  - `showCountdown` (boolean): Show a live countdown timer when `scheduleStart` is in the future.
  - `showClickBadge` (boolean): Display a public click count badge on the link card.
  - `scheduleStart` (string): Visibility start timestamp in RFC3339 format (for example `"2026-11-20T14:00:00Z"`).
  - `scheduleEnd` (string): Visibility expiration timestamp in RFC3339 format.

### 5. `update_profile_link`
Updates an existing profile link by its unique `key` and invalidates the cached public page. Call `list_profile_links` first if you need to look up the link `key` and current field values.
- **Required arguments:**
  - `key` (string): Unique link identifier from `list_profile_links`.
  - `profile` (string): Owning profile handle.
  - `title` (string): Updated display title.
  - `url` (string): Updated `http://` or `https://` destination URL.
  - `active` (boolean): `true` to keep visible, `false` to hide without deleting.
- **Optional arguments:**
  - `isSocial` (boolean), `badgeText` (string), `iosUrl` (string), `androidUrl` (string), `showCountdown` (boolean), `showClickBadge` (boolean).

### 6. `delete_profile_link`
Permanently deletes a link from a profile by `key` and purges the profile cache. Confirm with the user before deleting active links.
- **Required arguments:**
  - `key` (string): Unique key of the link to remove.

### 7. `import_bio_links`
Fetches a public external bio page (such as Linktree, Beacons, or Bio.link), extracts its public outbound links, adds them to the target Linq.es profile (up to the 100-link Pro profile cap), and refreshes the profile cache.
- **Required arguments:**
  - `profile` (string): Destination Linq.es profile handle.
  - `url` (string): Public source bio URL (for example `"https://linktr.ee/example"`).

### 8. `list_shortlinks`
Lists all `/s/:code` redirect shortlinks owned by the authenticated user (up to 500 active on Pro), returning `totalShortLinks`, `totalClicks`, destination URLs, device overrides, and UTM tags.
- **Optional arguments:**
  - `profile` (string): Filter shortlinks associated with a specific profile handle.

### 9. `create_shortlink`
Creates a new shortlink at `https://linq.es/s/:code` with optional custom vanity slug, device-aware iOS/Android redirects, and GA4 UTM query tags.
- **Required arguments:**
  - `targetUrl` (string): Destination `http://` or `https://` URL.
- **Optional arguments:**
  - `code` (string): Custom vanity slug (`3-50` alphanumeric, hyphen, or underscore characters; omit to auto-generate a 6-character Base62 code). Reserved system routes (`api`, `admin`, `login`, `vs`, `tools`, `mcp`, etc.) are rejected.
  - `title` (string): Internal label for dashboard organization.
  - `profile` (string): Optional profile handle association.
  - `iosUrl` (string): Optional iOS redirect override URL.
  - `androidUrl` (string): Optional Android redirect override URL.
  - `utmSource` (string): `utm_source` parameter appended on redirect.
  - `utmMedium` (string): `utm_medium` parameter appended on redirect.
  - `utmCampaign` (string): `utm_campaign` parameter appended on redirect.
  - `utmTerm` (string): `utm_term` parameter appended on redirect.
  - `utmContent` (string): `utm_content` parameter appended on redirect.

### 10. `list_leads`
Retrieves contacts (Name, Email, WhatsApp/Phone, timestamp) submitted through a profile's inline Lead Magnet form.
- **Required arguments:**
  - `profile` (string): Profile handle to inspect.
- **Optional arguments:**
  - `limit` (integer): Maximum recent leads to return (default `50`, max `200`).

## 5. Common Agent Workflows

1. **Campaign Launch (Shortlink + Bio Card):**
   - Call `list_profiles` to confirm the target `profile` handle.
   - Call `create_shortlink` with `targetUrl`, custom `code`, `utmSource: "bio"`, `utmMedium: "profile"`, and `utmCampaign`.
   - Call `add_profile_link` pointing to the destination or shortlink with `badgeText: "NEW"` and optional `scheduleStart` / `scheduleEnd`.
2. **Migration from Linktree or Beacons:**
   - Call `import_bio_links` with the user's existing public bio URL and target `profile`.
   - Call `list_profile_links` to review imported titles and URLs, then refine ordering or badges with `update_profile_link`.
3. **Weekly Attribution & Lead Review:**
   - Call `get_profile_analytics` with `days: 30` and `list_shortlinks` to compare CTR, top referrers, countries, and shortlink clicks.
   - Call `list_leads` with `limit: 50` to summarize recent inbound contacts.
