<!-- Generated from Via's internal docs. Edits made in this public repo are overwritten on the next sync. -->

# Via AI MCP Server

Via AI's MCP server gives AI assistants read-only access to your professional network --
finding people and companies, and showing how well-connected you are to each one, through
natural conversation -- plus a few explicit actions, such as saving a result as a Target
List or following a person or company.

## Features

- **People search**: Find people by name, persona, role, title, function, seniority, or
  location, or look up one exact person by email or LinkedIn profile URL.
- **Company search**: Look up companies by name or domain, with employee count, industries,
  and domains.
- **Access on every row**: Every person or company result carries a compact summary of how
  well you can reach them -- no extra query required.
- **Pathways**: Select a person from a result to open the routes connecting you to them,
  with the evidence behind each one (shared work history, education, email, meetings, and
  more).
- **Insights**: Ask for a specific rollup -- your strongest connections, your best-connected
  companies, or where your network clusters by function or location.

## Setup

### Prerequisites

- A [Via AI](https://www.connectvia.ai) account

### Claude.ai / Claude Desktop

Add Via AI as a custom connector:

1. Open Claude and click **Customize** in the left sidebar.

   ![Customize in the Claude sidebar](docs/assets/mcp/customize-sidebar.png)

2. Select **Connectors**, click the **+** button, and choose **Add custom connector**.

   ![Add custom connector from the Connectors panel](docs/assets/mcp/add-custom-connector.png)

3. In the **Add custom connector** dialog, enter a name (e.g. `Via AI`) and the server URL
   `https://mcp.connectvia.ai/mcp`, then click **Add**.

   ![Add custom connector dialog with the Via AI URL](docs/assets/mcp/connector-dialog.png)

4. Follow the OAuth flow to authorize your Via AI account.

### Claude Code

```
claude mcp add --transport http via https://mcp.connectvia.ai/mcp
```

Rendered results and Pathways need an MCP Apps-capable client such as Claude.ai or Claude
Desktop; in Claude Code the same tools return data rows.

Or add it to your Claude Code configuration directly:

```json
{
  "mcpServers": {
    "via": {
      "type": "http",
      "url": "https://mcp.connectvia.ai/mcp"
    }
  }
}
```

### Authentication

Via AI uses OAuth 2.0 through WorkOS AuthKit, with standard MCP resource-server discovery.
When you first connect, you'll be redirected to Via AI's login page to authorize access.
Tokens are automatically refreshed.

### Account readiness

New accounts finish a short onboarding step (Terms acceptance and profile) at
[app.connectvia.ai](https://app.connectvia.ai). `get_authenticated_user` returns your
profile and, where enabled, your Terms and onboarding status.

## Companion skill / plugin

Via publishes an official **Network Workflow** skill that teaches the assistant to use one
typed query per request, read the Access summary already on each row, and open Pathways
only from a selected person -- instead of re-querying or building its own tables. The
**Via** plugin bundles that skill with the connector for one-step setup.

Both are available once you're signed in at [app.connectvia.ai](https://app.connectvia.ai):
open the **Agent** panel, choose the **Connect** tab, and use the **Downloads** section of
the **Via for Claude** card. The plugin installs by uploading its ZIP in Claude's
**Customize -> Plugins**; the skill unzips alongside your custom connector.

## Tools

<!-- BEGIN GENERATED TOOLS (scripts/render_mcp_public_readme.py) -->

### Read-only queries

| Tool                       | Description                                                                                                                                             |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `run_network_query`        | Find People or Companies using your network and optional company columns from your files, with Access on every row and Pathways from a selected person. |
| `search_people`            | Look up people by name, persona, or role, or find exact people by email or LinkedIn URL.                                                                |
| `search_companies`         | Look up companies by name or domain, with employee count, industries, and domains.                                                                      |
| `find_network_insights`    | Compute a specific insight: strongest connections, best-connected companies, or function/location breakdowns.                                           |
| `get_authenticated_user`   | Read your Via profile and your Terms/onboarding status.                                                                                                 |
| `get_mcp_status`           | Check that your Via connection is ready, or inspect a previous result.                                                                                  |
| `read_signals`             | Read your Via activity feed and summary, a specific person, and your saved follows and subscriptions.                                                   |
| `render_network_result`    | Display a completed People or Companies result.                                                                                                         |
| `read_network_result_page` | Read more rows from a result, or check whether it has finished computing.                                                                               |

### Actions

These write to your Via workspace only (Target Lists, follows); they never contact anyone.

| Tool                  | Description                                                                                                  |
| --------------------- | ------------------------------------------------------------------------------------------------------------ |
| `run_product_command` | Run one supported Via action, such as saving a result to a Target List or following a person or company.     |
| `campaign_workspace`  | Save a People or Companies result as a Target List, list your Target Lists, or open one to see its activity. |

<!-- END GENERATED TOOLS -->

## Usage Examples

### Example 1: Finding warm introductions to a target company

**User prompt:** "How am I connected to people at Stripe? I'm looking for warm
introductions."

**What happens:** Claude calls `run_network_query` once for people at Stripe and answers
from the returned rows. Each row carries an Access summary -- how well you can reach that
person today. If you ask to browse or interact with the result, Claude displays that same
saved result without repeating the search. Selecting a promising person in the interactive
view opens their Pathways: the routes connecting you to them, with the evidence behind each
one, such as shared work history or email activity.

### Example 2: Researching a prospect before outreach

**User prompt:** "Look up john.smith@acme.com and tell me how we're connected."

**What happens:** Claude calls `search_people` with `identifiers: [{"email":
"john.smith@acme.com"}]` for an exact match -- profile only, no network query. To answer
how you're connected, Claude then runs one `run_network_query` for John; select him in the
result to open his Pathways, showing the strongest route in along with its supporting
evidence.

## Privacy Policy

[Via AI Privacy Policy](https://www.connectvia.ai/privacy)

## Support

- **Email:** help@connectvia.ai
- **Website:** [connectvia.ai](https://www.connectvia.ai)
