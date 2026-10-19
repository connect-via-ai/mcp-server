<!-- Generated from Via's internal docs. Edits made in this public repo are overwritten on the next sync. -->

# Via AI MCP Server

Via AI's MCP server gives AI assistants read-only access to your professional network --
finding people and companies, and showing how well-connected you are to each one, through
natural conversation. It also exposes explicit workspace actions for lists, networks,
follows, research settings, result views and exports. Some actions delete or replace data;
Via enforces the applicable confirmation requirements. The connector does not send outreach.

## Features

- **People search**: Find people in your network by name, company, role, title,
  function, seniority, location, email or LinkedIn URL. Preserve all supplied
  constraints in one query. An exact identity match and a qualifying network
  relationship are different facts; missing relationship evidence stays unknown.
- **Company search**: Find companies by name, domain or ordinary search language,
  with employee count, industries and Access when available.
- **Access on every row**: Every person or company result carries a compact summary of how
  well you can reach them -- no extra query required.
- **Pathways**: Select a person from a result to open the routes connecting you to them,
  with the evidence behind each one (shared work history, education, email, meetings, and
  more).
- **Relationship enrichment**: Read selected people's relationship facts as a reusable
  payload for your CRM, agent, or workflow. `read_relationship_evidence` defaults to
  `detail: "brief"`: up to three whole facts and 1 KiB of facts per relationship.
  Identities, relationship direction, evidence status, and missing coverage remain
  explicit. `factoid_count` counts returned facts; `total_factoid_count` and
  `factoids_truncated` show when more are available. Request `detail: "full"` for
  all published facts, or use `row_ids` to select up to 200 retained people.
  Without `row_ids`, `limit` defaults to 25. Brief and full modes cover the same
  selected people. Another connected app can consume this payload; this read does
  not update your CRM or send email.
- **Target lists**: Save a complete People or Companies result, reopen a saved list,
  and review its targets, next actions, and history. Saved lists show their latest
  recorded Access while background checks update it. Opening a list also shows its
  saved Networks, source identities, and relationship rules; it does not change the
  Networks selected for your search or start another Access query. Saving never
  starts outreach.
  MCP tools use `read_target_list_workspace`, `write_target_list_workspace`,
  and `target_list_id`. Target List action names also use `target_list`.
  Existing integrations using `read_campaign_workspace`, `write_campaign_workspace`,
  `save_network_result_campaign`, original action names, or `campaign_id` remain
  supported with the same permissions and confirmation requirements. Those calls
  retain their existing result fields; newly discovered calls use the target-list contract.
  People and Company Lists stay separate: mixed or opposite-kind additions return
  `target_list_target_kind_mismatch` in structured error data without changing the List.
  The human reply names the destination kind, suggests a matching-kind target list,
  and says nothing was saved; internal codes stay out of the refusal text. Company Lists can
  retain unresolved source targets even when no canonical company is found.
  Resolver outages still fail without creating a List.
  Existing mixed Lists remain unchanged.
  To find People **at companies in a saved Company List**, use
  `run_network_query` with `deliverable: "people"` and
  `company_lists: ["List name or id"]`. All currently readable Company members
  constrain the employers, and a People List contributes each member's current
  employer; a preview is not the population. When the call names no
  `network_sources`, Via searches the List's shared networks by their stored keys
  and says so first ("Searched through: ... (shared on <List>)"). If fewer than 10
  direct matches come back and no degree was asked for, the same call also searches
  one warm or good introducer away and reports "N direct, M via one introducer".
  Use `any_of` for alternative role criteria. By contrast, `lists` selects members
  **on** Lists of the requested result kind.
  Already-supported empty Company saves retain Company kind and their source Networks.
- **Email hand-offs**: Your agent can list queued email briefs, read one and mark
  it drafted or skipped. Your separate mail tool owns the draft and send; Via
  never sends email. The connector only sees your own queued hand-offs.
- **Intro emails**: Your agent can read your own writing profile, including your
  intro email instructions and remembered wording, and write or rewrite an email
  asking a connection to introduce you. Each intro is saved as a draft in Via;
  nothing is sent, and the connector only reads your own profile and drafts.
- **Prompt suggestions**: The connector publishes starting prompts, including
  a guided walkthrough available from your client's prompt or resource picker.
  The reviewed walkthrough ships with each server release.

## Setup

### Prerequisites

- A [Via AI](https://www.connectvia.ai) account

### Claude.ai / Claude Desktop

Add Via AI as a custom connector:

1. Open Claude and click **Customize** in the left sidebar.

   ![Customize in the Claude sidebar](docs/assets/mcp/customize-sidebar.png)

2. Select **Connectors**, click **Add**, and choose **Add custom connector**.

   ![Add custom connector from the Connectors panel](docs/assets/mcp/add-custom-connector.png)

3. In the **Add custom connector** dialog, enter a name (e.g. `Via AI`) and the server URL
   `https://mcp.connectvia.ai/mcp`, then click **Continue**. On the next screen, keep the detected
   settings and click **Add**.

   ![Add custom connector dialog with the Via AI URL](docs/assets/mcp/connector-dialog.png)

4. Follow the OAuth flow to authorize your Via AI account.

### Claude Code

```
claude mcp add --transport http via https://mcp.connectvia.ai/mcp
```

Interactive results need an MCP Apps-capable client such as Claude.ai or Claude Desktop.
Data-only clients can read result rows and request a selected person's Pathways through tools.
Responses include each result row once, with page counts that match the returned rows.
The result-row tool accepts either the returned continuation cursor or the next-page
address. Paging reads the saved result and does not repeat the query.

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

### ChatGPT

Add Via as a custom MCP connector using `https://mcp.connectvia.ai/mcp`, then
complete the existing Via sign-in. Claude and ChatGPT use the same server URL,
authentication and tool catalog. Custom connector availability depends on your
client and account. Public directory publication is a separate release step.

### Shared tool interface

The default connection exposes your signed-in profile, network search, paging, selected-person Pathways,
relationship evidence, display, queued-email hand-offs and intro email drafts. Workspace actions are
individually named and publish their complete input schemas. Related fields and
bounded bulk selections stay together in one action. Existing permissions and
confirmation requirements remain enforced by the same Via services.

Ordinary requests return data. An explicit request to display or interact with a
result opens that saved result in the Via App, without another search. App
controls have four additional helpers that only the interactive view calls; hosts
without MCP Apps support may list them, and their descriptions name the tool to use instead. Your client may need
to refresh its connection to see the updated tool list. Expanded server profiles
can provide supplemental tools; the table below describes the default interface.

### Authentication

Via AI uses OAuth 2.0 through WorkOS AuthKit, with standard MCP resource-server discovery.
When you first connect, you'll be redirected to Via AI's login page to authorize access.
Tokens are automatically refreshed.

### Account readiness

New accounts finish a short onboarding step (Terms acceptance and profile) at
[app.connectvia.ai](https://app.connectvia.ai). Complete this in the Via app before
using the connector.

## Companion skill / plugin

Via publishes an official **Network Workflow** skill that teaches the assistant to use one
typed query per request, read the Access summary already on each row, and open Pathways
only from a selected person. Ordinary requests use Via's returned data. Interactive
requests render the retained result without another query or duplicate tables.
The **Via** plugin bundles that skill with the connector for one-step setup.

Both are available once you're signed in at [app.connectvia.ai](https://app.connectvia.ai):
open the **Agent** panel, choose the **Connect** tab, and use the **Downloads** section of
the **Via for Claude** card. The plugin installs by uploading its ZIP in Claude's
**Customize -> Plugins**; the skill unzips alongside your custom connector.

The companion is optional. Where exposed by your connection, `get_mcp_status`
reports its identity and version compatibility without installation instructions. Downloads remain a user choice in
Via's setup card; the connector does not require a skill installation to operate.

Skill version `2026.10.18.1` and plugin version `1.0.17` use the declared workflow
tools. The companion keeps other requested sources separate from Via's results.
It reads viewer-to-person evidence in batches and verifies introduction-route legs
through selected-Person Pathways. Missing evidence remains unknown.
Where compatibility checks are available, earlier skill versions report `update_required`. The assistant explains the version
mismatch and asks before an update; it does not download or install one without
explicit user approval. The connector remains available without the companion.

## Tools

Every tool publishes the same readable name in `title` and `annotations.title`.
`run_network_query.network_scope` explicitly declares its object type and retains
its referenced schema for network-source and relationship constraints.

<!-- BEGIN GENERATED TOOLS (scripts/render_mcp_public_readme.py) -->

### Read-only queries

| Tool                            | Description                                                                                                                                                     |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `run_network_query`             | Find People or Companies using your network and optional company columns from files stored in Via, with Access on each row and Pathways from a selected person. |
| `get_authenticated_user`        | Read your Via profile and your Terms/onboarding status.                                                                                                         |
| `read_outreach_handoffs`        | List the email hand-offs you queued in Via for your own agent to write, or read one with its full brief and draft.                                              |
| `get_writing_profile`           | Read your own writing profile, including your intro email instructions, remembered wording and custom parts.                                                    |
| `read_saved_network`            | Read one of your saved Networks and its members.                                                                                                                |
| `summarize_results`             | Summarize a result, saved Network or Target List by company, role, industry, seniority or location.                                                             |
| `explore_person_network`        | See the people a selected person knows whom you can reach, optionally filtered by role.                                                                         |
| `read_target_lists`             | Browse your Target Lists, saved results, and their targets and activity.                                                                                        |
| `render_network_result`         | Display a completed People or Companies result.                                                                                                                 |
| `read_network_result_page`      | Read more rows from a result, or check whether it has finished computing.                                                                                       |
| `open_selected_person_pathways` | Verify introduction routes and relationship evidence for one person on a saved result.                                                                          |
| `read_relationship_evidence`    | Read brief relationship facts or full detail for selected people, including connector preview second legs with verified and missing coverage.                   |

### Actions

These manage Via workspace data and settings, including lists, networks, follows and result views. Some actions delete or replace data and require confirmation. They do not send outreach messages.

| Tool                         | Description                                                                                             |
| ---------------------------- | ------------------------------------------------------------------------------------------------------- |
| `complete_outreach_handoff`  | Mark a queued email hand-off as drafted or skipped once your agent has handled it.                      |
| `draft_intro`                | Write or rewrite an email asking a connection to introduce you, saved as a draft in Via and never sent. |
| `refine_network_result`      | Filter, sort, trim, group or re-rank a result into a new result, keeping the original.                  |
| `create_target_list`         | Create a Target List with a name, an optional goal and optional starting members.                       |
| `update_target_list`         | Rename a Target List, change its goal, or add a note to its activity.                                   |
| `run_target_list_audit`      | Check how well you are connected to a Target List's members and keep them followed.                     |
| `create_saved_network`       | Save a group of people as a named Network from a result, a list of people or a file.                    |
| `update_saved_network`       | Add, remove or move people in one of your saved Networks, or rename it.                                 |
| `import_connections`         | Import LinkedIn connections from a file or pasted profiles, for you or a named person.                  |
| `curate_dossier_proofs`      | Surface, pin or remove matching proofs in one person's dossier.                                         |
| `follow_for_research`        | Follow a company or person to receive research updates on a cadence you choose.                         |
| `update_agent_settings`      | Change your network agent's mission, watched topics, or how often it looks and reports.                 |
| `remove_network_result_rows` | Remove matching rows from a result after you confirm the change.                                        |
| `export_results`             | Export a result or a Target List's targets to a private CSV download.                                   |
| `add_target_list_members`    | Add people or companies to a Target List after you confirm the change.                                  |
| `remove_target_list_members` | Remove people or companies from a Target List after you confirm the change.                             |
| `archive_target_list`        | Archive a Target List after you confirm the change.                                                     |
| `save_target_list_radar`     | Save radar criteria and cadence on a Target List after you confirm them.                                |
| `delete_saved_network`       | Delete one of your saved Networks after you confirm the change.                                         |
| `unfollow_research`          | Stop following a company or person for research after you confirm the change.                           |
| `set_active_icp`             | Set, clear or switch the ideal customer profile that guides your searches after you confirm it.         |
| `set_watcher_paused`         | Pause or resume one of your monitored automations after you confirm the change.                         |
| `save_result_as_target_list` | Save a People or Companies result as a Target List without starting outreach.                           |

<!-- END GENERATED TOOLS -->

## Usage Examples

When creating a Target List from results, select all matching rows or choose
individual rows as its starting members. A result link alone does not add them.
Narrow the results first to preserve a filter or limit, and choose quiet tending
to keep the list out of outreach. Agent settings changes save the mission,
watched topics and cadence together.

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

**What happens:** Claude calls `run_network_query` once with `deliverable: "people"` and
`person_identifiers: ["john.smith@acme.com"]` for that exact person. The row carries their
Access summary and likely introducers; select the row to open their Pathways, showing
the strongest route in along with its supporting evidence.

### Example 3: Comparing target accounts

**User prompt:** "Compare Stripe and HubSpot by how well my network can reach them."

**What happens:** Claude calls `run_network_query` once with a Companies result and the
2 named targets. It answers from each company's Access summary and reports unresolved
or incomplete results separately. You can ask to display the same result or explicitly
save it as a Target List. Saving does not start outreach.

When you ask for a summary of the current result, Via counts the complete
visible view, including rows beyond its first page. The summary respects the
result's relationship-strength selection and does not run another search.

## Working with other connectors

Via supplies network results, relationship evidence and Via workspace actions. Your
assistant can combine these with separately authorized CRM, prospecting, call-recorder
or messaging connectors. Those connectors own their reads, writes and sends; connecting
Via alone does not grant access to them or to your chat history.

Paste company names or domains for a direct query. File-column references require a file
already stored in Via with an authorized attachment identifier. They cannot read arbitrary
files attached to your assistant's chat.

## Troubleshooting

- **Connection or account issue:** Finish any required Terms/profile steps in Via.
  Reconnect the custom connector if its authorization has expired or you revoked it.
  `get_authenticated_user` shows your signed-in profile; expanded connections may also expose `get_mcp_status`.
- **Trial ended:** A personal account whose Via trial has ended gets a tool error with
  code `personal_plan_required` and a billing link. Subscribe in Via, then retry.
- **Results still running:** Continue reading the same result. Pending or failed work does
  not mean that your network has no matches.
- **No interactive table:** Use a client with MCP Apps support, or ask for data rows.
- **Missing relationship evidence:** Request it for selected rows or open the person's
  Pathways. Deferred evidence does not arrive through repeated page reads alone.
- **Need help:** Contact support below with your client and the error message.
  Do not send access tokens or private network exports.

## Privacy Policy

[Via AI Privacy Policy](https://www.connectvia.ai/privacy)

## Support

- **Email:** help@connectvia.ai
- **Website:** [connectvia.ai](https://www.connectvia.ai)
