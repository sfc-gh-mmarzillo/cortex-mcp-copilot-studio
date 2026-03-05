# Expose Cortex Agent to Microsoft Copilot Studio

This skill guides you through connecting your Snowflake Cortex Agent to Microsoft Copilot Studio using Snowflake's Managed MCP (Model Context Protocol).

## Quick Start: Using This Skill

### Option 1: Install Globally (Recommended)

Clone this repo into your Cortex Code skills directory:

```bash
git clone https://github.com/sfc-gh-mmarzillo/cortex-mcp-copilot-studio.git \
  ~/.snowflake/cortex/skills/expose-agent-to-copilot-studio
```

Then in Cortex Code, simply ask:
```
Expose my agent to Copilot Studio
```

### Option 2: Install Per-Project

Clone into your project's `.cortex/skills` directory:

```bash
git clone https://github.com/sfc-gh-mmarzillo/cortex-mcp-copilot-studio.git \
  .cortex/skills/expose-agent-to-copilot-studio
```

### Trigger Phrases

The skill activates when you mention:
- "copilot studio"
- "MCP server"
- "expose agent"
- "microsoft integration"
- "oauth copilot"

---

## What This Skill Does

This skill automates the setup required to expose a Snowflake Cortex Agent to Microsoft Copilot Studio. It handles:

1. **MCP Server Creation** - Creates a Snowflake Managed MCP server that wraps your Cortex Agent
2. **Permission Grants** - Configures the necessary database, schema, and object permissions
3. **OAuth Integration** - Sets up secure OAuth 2.0 authentication for Copilot Studio
4. **Configuration Guidance** - Provides step-by-step instructions for the Copilot Studio UI

## Prerequisites

Before running this skill, ensure you have:

- **A Cortex Agent** already created in Snowflake
- **ACCOUNTADMIN role** access (or equivalent privileges)
- **Microsoft Copilot Studio** access at https://copilotstudio.microsoft.com

## Workflow Overview

| Step | Description |
|------|-------------|
| 1 | Gather your agent's location (database, schema, name) and account info |
| 2 | Create an MCP Server exposing your agent |
| 3 | Grant necessary permissions (with security review) |
| 4 | Create OAuth security integration |
| 5 | Retrieve OAuth credentials (Client ID & Secret) |
| 6 | Configure the connection in Copilot Studio |
| 7 | Update the OAuth redirect URL |
| 8 | Complete the connection and test |

## What Gets Created

| Object | Naming Convention |
|--------|-------------------|
| MCP Server | `<DATABASE>.<SCHEMA>.<AGENT>_MCP_SERVER` |
| OAuth Integration | `<AGENT>_COPILOT_OAUTH` |

## Architecture

```
┌─────────────────────┐         ┌──────────────────────┐
│  Microsoft Copilot  │         │      Snowflake       │
│       Studio        │         │                      │
│                     │  OAuth  │  ┌────────────────┐  │
│  ┌───────────────┐  │◄───────►│  │ MCP Server     │  │
│  │  Your Copilot │  │   MCP   │  │                │  │
│  │               │──┼────────►│  │ ┌────────────┐ │  │
│  └───────────────┘  │         │  │ │Cortex Agent│ │  │
│                     │         │  │ └────────────┘ │  │
└─────────────────────┘         │  └────────────────┘  │
                                └──────────────────────┘
```

## Security Considerations

- The skill defaults to using the `PUBLIC` role for simplicity
- You can opt for a **custom role** during the permission grant step
- OAuth tokens are configured with 24-hour refresh validity
- Always review permissions before granting access

## Cleanup

To remove the integration, run these SQL commands:

```sql
DROP MCP SERVER IF EXISTS <DATABASE>.<SCHEMA>.<AGENT>_MCP_SERVER;
DROP SECURITY INTEGRATION IF EXISTS <AGENT>_COPILOT_OAUTH;
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Object does not exist" in Copilot Studio | Verify MCP server exists with `SHOW MCP SERVERS IN SCHEMA` |
| OAuth errors | Check redirect URL matches exactly in security integration |
| Permission denied | Verify role grants with `SHOW GRANTS TO ROLE PUBLIC` |
| Agent not responding | Test agent directly in Snowflake first |

## Resources

- [Snowflake MCP Documentation](https://docs.snowflake.com/en/developer-guide/mcp/overview)
- [Cortex Agents Documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agent)
- [Microsoft Copilot Studio](https://copilotstudio.microsoft.com)

## License

MIT
