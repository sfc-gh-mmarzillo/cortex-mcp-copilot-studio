---
name: expose-agent-to-copilot-studio
description: "Expose a Cortex Agent to Microsoft Copilot Studio via Snowflake Managed MCP. Use when: user wants to connect agent to Copilot Studio, set up MCP server for Copilot, integrate Snowflake with MS 365, enable OAuth for Copilot Studio. Triggers: copilot studio, MCP server, expose agent, microsoft integration, oauth copilot."
---

# Expose Cortex Agent to Microsoft Copilot Studio

Exposes an existing Snowflake Cortex Agent to Microsoft Copilot Studio via Snowflake Managed MCP (Model Context Protocol).

## Prerequisites

- An existing Cortex Agent in Snowflake
- ACCOUNTADMIN role access (or equivalent privileges)
- Microsoft Copilot Studio access (https://copilotstudio.microsoft.com)

## Workflow

### Step 1: Gather Agent Information

**Ask user for the following:**

```
To expose your agent to Copilot Studio, provide:

1. DATABASE     : Database containing your agent
2. SCHEMA       : Schema containing your agent  
3. AGENT        : Your Cortex Agent name
4. ACCOUNT      : Snowflake account identifier (e.g., abc12345)
5. WAREHOUSE    : Warehouse for query execution
```

If user doesn't know their agent location, suggest:
```sql
SHOW AGENTS IN ACCOUNT;
```

**Store values as:**
- `<DATABASE>` - Database name
- `<SCHEMA>` - Schema name
- `<AGENT>` - Agent name
- `<ACCOUNT>` - Account identifier
- `<WAREHOUSE>` - Warehouse name

**Stop**: Confirm all values before proceeding.

---

### Step 2: Create MCP Server

**Execute:**

```sql
USE ROLE ACCOUNTADMIN;

CREATE OR REPLACE MCP SERVER <DATABASE>.<SCHEMA>.<AGENT>_MCP_SERVER
  FROM SPECIFICATION $$
    tools:
      - title: "<AGENT>"
        name: "<AGENT>"
        type: "CORTEX_AGENT_RUN"
        identifier: "<DATABASE>.<SCHEMA>.<AGENT>"
        description: "Cortex Agent exposed via MCP"
  $$;
```

**Verify creation:**
```sql
SHOW MCP SERVERS IN SCHEMA <DATABASE>.<SCHEMA>;
```

---

### Step 3: Grant Permissions

**Review these grants with the user before executing:**

```sql
-- Required grants for MCP access
GRANT USAGE ON DATABASE <DATABASE> TO ROLE PUBLIC;
GRANT USAGE ON SCHEMA <DATABASE>.<SCHEMA> TO ROLE PUBLIC;
GRANT USAGE ON MCP SERVER <DATABASE>.<SCHEMA>.<AGENT>_MCP_SERVER TO ROLE PUBLIC;
GRANT USAGE ON AGENT <DATABASE>.<SCHEMA>.<AGENT> TO ROLE PUBLIC;
GRANT USAGE ON WAREHOUSE <WAREHOUSE> TO ROLE PUBLIC;
GRANT SELECT ON ALL TABLES IN SCHEMA <DATABASE>.<SCHEMA> TO ROLE PUBLIC;
```

**Optional grants (ask user if needed):**

```sql
-- If agent uses a semantic model stage:
GRANT READ ON STAGE <DATABASE>.<SCHEMA>.<STAGE_NAME> TO ROLE PUBLIC;

-- If agent uses Cortex Search:
GRANT USAGE ON CORTEX SEARCH SERVICE <DATABASE>.<SCHEMA>.<SERVICE_NAME> TO ROLE PUBLIC;
```

**Stop**: Get explicit approval before granting. User may prefer a custom role instead of PUBLIC.

**If user wants a custom role:**
```sql
CREATE ROLE IF NOT EXISTS <CUSTOM_ROLE>;
-- Replace PUBLIC with <CUSTOM_ROLE> in all grants above
-- Then grant the custom role to users who need access
```

---

### Step 4: Create OAuth Integration

**Execute:**

```sql
CREATE OR REPLACE SECURITY INTEGRATION <AGENT>_COPILOT_OAUTH
    TYPE = OAUTH
    ENABLED = TRUE
    OAUTH_CLIENT = CUSTOM
    OAUTH_CLIENT_TYPE = 'CONFIDENTIAL'
    OAUTH_REDIRECT_URI = 'https://placeholder.example.com/callback'
    OAUTH_ISSUE_REFRESH_TOKENS = TRUE
    OAUTH_REFRESH_TOKEN_VALIDITY = 86400;
```

The redirect URI is a placeholder - it will be updated in Step 7 after Copilot Studio generates the real one.

---

### Step 5: Get OAuth Credentials

**Execute:**

```sql
SELECT SYSTEM$SHOW_OAUTH_CLIENT_SECRETS('<AGENT>_COPILOT_OAUTH');
```

**Present credentials to user:**

```
Save these credentials - you'll need them for Copilot Studio:

OAUTH_CLIENT_ID:     <value from query>
OAUTH_CLIENT_SECRET: <value from query>
```

**Stop**: Ensure user has copied both values before proceeding.

---

### Step 6: Configure Copilot Studio (Manual Steps)

**Present these instructions to the user:**

```
COPILOT STUDIO CONFIGURATION
============================

1. Open https://copilotstudio.microsoft.com

2. Open your Copilot -> "Actions and connectors" -> "+ Add action" -> "MCP Server"

3. Fill in the connection details:

   Server name:        <any descriptive name>
   Server description: <any description>
   Server URL:         https://<ACCOUNT>.snowflakecomputing.com/api/v2/databases/<DATABASE>/schemas/<SCHEMA>/mcp-servers/<AGENT>_MCP_SERVER
   
   Authentication:     OAuth 2.0
   Type:               Manual
   
   Client ID:          <OAUTH_CLIENT_ID from Step 5>
   Client secret:      <OAUTH_CLIENT_SECRET from Step 5>
   Authorization URL:  https://<ACCOUNT>.snowflakecomputing.com/oauth/authorize
   Token URL template: https://<ACCOUNT>.snowflakecomputing.com/oauth/token-request
   Refresh URL:        https://<ACCOUNT>.snowflakecomputing.com/oauth/token-request
   Scopes:             session:role:PUBLIC
   Redirect URL:       (leave as auto-generate)

4. Click "Create"

5. SUCCESS dialog appears - COPY the Redirect URL shown

6. Click "Cancel" (do NOT click Next yet - we need to update Snowflake first)

Return here with the Redirect URL from step 5.
```

**Stop**: Wait for user to provide the Redirect URL from Copilot Studio.

---

### Step 7: Update Redirect URL

**Once user provides the Redirect URL, execute:**

```sql
ALTER SECURITY INTEGRATION <AGENT>_COPILOT_OAUTH 
    SET OAUTH_REDIRECT_URI = '<REDIRECT_URL_FROM_COPILOT_STUDIO>';
```

**Verify the update:**
```sql
DESCRIBE SECURITY INTEGRATION <AGENT>_COPILOT_OAUTH;
```

Confirm `OAUTH_REDIRECT_URI` shows the new URL.

---

### Step 8: Complete Connection

**Present final instructions:**

```
COMPLETE THE CONNECTION
=======================

1. Return to Copilot Studio

2. Open your MCP connection (it may show as incomplete)

3. Click "Next" or "Connect"

4. Sign in with your Snowflake credentials

5. Done! Your agent is now available in Copilot Studio.

TEST YOUR AGENT
===============

In Copilot Studio, try invoking your agent with a test question.
The agent should respond using your Snowflake Cortex Agent.
```

---

## Quick Reference

### Generated Object Names

| Object | Name |
|--------|------|
| MCP Server | `<DATABASE>.<SCHEMA>.<AGENT>_MCP_SERVER` |
| OAuth Integration | `<AGENT>_COPILOT_OAUTH` |

### Key URLs (replace placeholders)

| URL | Value |
|-----|-------|
| Server URL | `https://<ACCOUNT>.snowflakecomputing.com/api/v2/databases/<DATABASE>/schemas/<SCHEMA>/mcp-servers/<AGENT>_MCP_SERVER` |
| Authorization URL | `https://<ACCOUNT>.snowflakecomputing.com/oauth/authorize` |
| Token URL | `https://<ACCOUNT>.snowflakecomputing.com/oauth/token-request` |

### Scopes

| Role | Scope Value |
|------|-------------|
| PUBLIC | `session:role:PUBLIC` |
| Custom role | `session:role:<CUSTOM_ROLE>` |

---

## Troubleshooting

### "Object does not exist" in Copilot Studio
- Verify MCP server exists: `SHOW MCP SERVERS IN SCHEMA <DATABASE>.<SCHEMA>;`
- Check grants were applied: `SHOW GRANTS ON MCP SERVER <DATABASE>.<SCHEMA>.<AGENT>_MCP_SERVER;`

### OAuth errors
- Verify redirect URL matches exactly: `DESCRIBE SECURITY INTEGRATION <AGENT>_COPILOT_OAUTH;`
- Ensure OAuth integration is enabled: `ENABLED = TRUE`

### Permission denied
- Check role grants: `SHOW GRANTS TO ROLE PUBLIC;`
- Verify warehouse access: `SHOW GRANTS ON WAREHOUSE <WAREHOUSE>;`

### Agent not responding
- Test agent directly in Snowflake first
- Check agent has proper tool access and semantic model permissions

---

## Cleanup (if needed)

To remove the integration:

```sql
DROP MCP SERVER IF EXISTS <DATABASE>.<SCHEMA>.<AGENT>_MCP_SERVER;
DROP SECURITY INTEGRATION IF EXISTS <AGENT>_COPILOT_OAUTH;
-- Revoke grants manually if using custom role
```

---

## Stopping Points

- After Step 1: Confirm agent coordinates
- After Step 3: Review permissions before granting (security checkpoint)
- After Step 5: Ensure credentials are saved
- After Step 6: Wait for Redirect URL from user
- After Step 7: Verify redirect URL update

## Output

- MCP Server exposing Cortex Agent
- OAuth security integration for Copilot Studio
- Working connection between Copilot Studio and Snowflake Agent
