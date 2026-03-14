# Cortex Agent → Microsoft Copilot Studio (MCP)

Connect a Snowflake Cortex Agent to Microsoft Copilot Studio using Snowflake's Managed MCP server with OAuth authentication.

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

## Cortex Code Skill

This repo contains a Cortex Code skill. To use it:

```bash
cortex skill add skills/expose-agent-to-copilot-studio
```

Then ask Cortex Code:

> "How do I expose my agent to Copilot Studio?"

## What's Inside

| Path | Purpose |
|------|---------|
| `skills/expose-agent-to-copilot-studio/SKILL.md` | Interactive skill definition (8 steps) |
| `skills/expose-agent-to-copilot-studio/skill_evidence.yaml` | Promotion tracking |

## Prerequisites

- An existing Cortex Agent in Snowflake
- ACCOUNTADMIN role access (or equivalent privileges)
- Microsoft Copilot Studio access (https://copilotstudio.microsoft.com)

## Quick Reference

| Object | Name Pattern |
|--------|-------------|
| MCP Server | `<DB>.<SCHEMA>.<AGENT>_MCP_SERVER` |
| OAuth Integration | `<AGENT>_COPILOT_OAUTH` |

## Resources

- [Snowflake MCP Documentation](https://docs.snowflake.com/en/developer-guide/mcp/overview)
- [Cortex Agents Documentation](https://docs.snowflake.com/en/user-guide/snowflake-cortex/cortex-agent)
- [Microsoft Copilot Studio](https://copilotstudio.microsoft.com)
