# Via AI MCP Server

Via AI's MCP server gives AI assistants access to relationship intelligence -- helping users discover connection paths, search for people and companies, and manage their professional network through natural conversation.

## Features

- **People Search**: Find people by name, email, or LinkedIn profile across Via's professional network graph
- **Company Search**: Look up companies by name or domain with metadata including industry, employee count, and domains
- **Connection Path Discovery**: Find the strongest paths connecting you to anyone at a target company through shared work history, education, email interactions, calendar meetings, LinkedIn connections, and more
- **ICP-Based Targeting**: Discover paths to people matching your Ideal Customer Profile (ICP) at target companies -- surface decision-makers most relevant to your sales motion
- **Inner Circle Management**: Organize your key contacts into circles (tags), add/remove members, and leverage your inner circle's network for warm introductions
- **Path Explanations**: Generate natural language descriptions of how two people are connected, suitable for email introductions

## Setup

### Prerequisites

- A [Via AI](https://www.connectvia.ai) account

### Claude.ai / Claude Desktop

Via AI is available as a connector in Claude. Search for "Via AI" in the connectors directory and follow the OAuth flow to connect your account.

### Claude Code

Add to your Claude Code configuration:

```json
{
  "mcpServers": {
    "via-ai": {
      "type": "url",
      "url": "https://mcp.connectvia.ai/mcp"
    }
  }
}
```

### Authentication

Via AI uses **OAuth 2.0 Authorization Code Flow**. When you first connect, you'll be redirected to Via AI's login page to authorize access. Tokens are automatically refreshed.

## Tools

### Queries (Read-Only)

| Tool | Description |
|------|-------------|
| `SearchPeopleByNameOrLinkedInOrEmail` | Search for people by name, LinkedIn slug, or email. Returns up to 20 results with profile metadata and circle memberships. |
| `FindPeopleByEmailsOrLinkedIn` | Look up one or more people by email or LinkedIn slug. Returns profiles in the same order as input. |
| `SearchCompaniesByNameOrDomain` | Search for companies by name or domain. Returns up to 5 results per query with employee count, industries, and domains. |
| `FindIcpConnectionPaths` | Discover connection paths to people matching your ICP at a target company. Returns ranked paths through mutual connections. |
| `FindTopConnectionPaths` | Find the strongest connection paths to anyone at a target company, regardless of ICP criteria. |
| `FindConnectionPathsToPeople` | Find connection paths to one or more specific people by their ID. |
| `GenerateConnectionPathExplanation` | Generate a natural language explanation of a connection path, suitable for introductions. |
| `GetAuthenticatedUser` | Get your Via AI profile, ICP criteria, and onboarding status. |
| `GetUserCircles` | List all your circles (tags) for organizing inner circle members. |
| `GetUserCircleMembers` | List members of your inner circle with contact details and employment info. Supports pagination and tag filtering. |

### Mutations (Write)

| Tool | Description |
|------|-------------|
| `AddPersonToUserCircle` | Add a person to your inner circle, optionally assigning them to circles. |
| `RemovePersonFromUserCircle` | Remove a person from your inner circle. |
| `CreateUserCircle` | Create a new circle (tag) for organizing contacts. |
| `DeleteUserCircle` | Delete a circle. Members are not removed from the inner circle. |
| `UpdatePersonCircleMembership` | Update which circles a person belongs to (replace operation). |
| `UpdateAuthenticatedUserIcpCriteria` | Update your ICP targeting criteria in natural language. |
| `SubmitAgentFeedback` | Submit feedback about your experience using Via AI tools. |

## Usage Examples

### Example 1: Finding warm introductions to a target company

**User prompt:** "How am I connected to people at Stripe? I'm looking for warm introductions to their engineering leadership."

**What happens:** Claude uses `SearchCompaniesByNameOrDomain` to find Stripe's domain, then calls `FindIcpConnectionPaths` with the domain to discover ranked connection paths. Each path shows the chain of people connecting you to Stripe employees matching your ICP, along with evidence like shared work history, email interactions, and meeting history.

**Result:** "You have 3 strong paths to engineering leaders at Stripe:
1. You -> Sarah Chen (worked together at Acme Corp 2019-2022, 47 emails exchanged) -> James Liu (VP Engineering at Stripe)
2. Your inner circle member Alex Park -> David Kim (Stanford '15 classmates) -> Maria Santos (Staff Engineer at Stripe)
..."

### Example 2: Researching a prospect before outreach

**User prompt:** "Look up john.smith@acme.com and tell me how we're connected."

**What happens:** Claude calls `FindPeopleByEmailsOrLinkedIn` with the email to find John's profile, then uses `FindConnectionPathsToPeople` to discover connection paths. Finally, it calls `GenerateConnectionPathExplanation` to create a natural language summary of the strongest path.

**Result:** "John Smith is a Senior Product Manager at Acme Corp. You're connected through your colleague Sarah Chen -- they worked together at TechCo from 2018 to 2021 and still exchange emails regularly. Sarah would be a great person to ask for an introduction."

### Example 3: Organizing your network with circles

**User prompt:** "Create a circle called 'Q1 Prospects' and add the top 3 people from my ICP matches at Datadog to it."

**What happens:** Claude calls `CreateUserCircle` to create the "Q1 Prospects" circle, then uses `SearchCompaniesByNameOrDomain` to find Datadog, followed by `FindIcpConnectionPaths` to identify ICP matches. It then calls `SearchPeopleByNameOrLinkedInOrEmail` for each person and `AddPersonToUserCircle` to add them to the new circle.

**Result:** "Done! I created the 'Q1 Prospects' circle and added 3 ICP matches from Datadog:
1. Lisa Wang - VP of Sales
2. Michael Torres - Head of Partnerships
3. Priya Patel - Director of Business Development"

## Privacy Policy

[Via AI Privacy Policy](https://www.connectvia.ai/privacy)

## Support

- **Email:** help@connectvia.ai
- **Website:** [connectvia.ai](https://www.connectvia.ai)
