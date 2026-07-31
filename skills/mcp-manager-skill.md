---
name: Mcpmanager
description: Use when building, deploying, or managing MCP gateways and servers; configuring identity and access control; setting up gateway rules and data loss prevention; integrating AI clients with MCP servers; or troubleshooting MCP deployments. Reach for this skill when working with Model Context Protocol infrastructure, governance, security policies, or observability.
metadata:
    mintlify-proj: mcpmanager
    version: "1.0"
---

# MCP Manager Skill

## Product summary

**MCP Manager** is an enterprise control plane for the Model Context Protocol (MCP). It sits as a governed gateway between AI clients (Claude, ChatGPT, Cursor, custom agents) and MCP servers (GitHub, Jira, Slack, internal databases, custom tools). Every request passes through the gateway where it is authenticated, inspected against rules, logged, and routed to the right server with the correct identity. The core unit is a **gateway** — one URL that aggregates multiple servers, applies policies, and logs all traffic. Key files and concepts: gateways (one URL, many servers), servers (Remote, Managed, Workstation), identities (per-user or shared), gateway rules (regex, Presidio, custom engines), and feature provisioning (tool allowlists). Primary docs: https://docs.mcpmanager.ai

## When to use

Reach for this skill when:
- **Setting up MCP infrastructure**: Adding servers (Remote, Managed, Workstation), creating gateways, provisioning access to teams
- **Configuring identity and access**: Setting per-user vs. shared identity schemes, managing OAuth flows, creating API tokens for headless agents
- **Implementing security policies**: Building gateway rules (regex, Presidio, custom engines), filtering PII, blocking prompt injection, pinning tools
- **Troubleshooting deployments**: Debugging authentication failures, connection issues, rule behavior, or log analysis
- **Scaling governance**: Choosing gateway topologies (org-wide, per-team, per-server, per-use-case), managing roles and capabilities, setting up RBAC
- **Observability and compliance**: Reading audit logs, exporting to SIEMs, understanding request attribution, verifying rule enforcement

## Quick reference

### Server types and when to use each

| Server Type | Where it runs | Best for | Setup complexity |
| --- | --- | --- | --- |
| **Remote** | Already at a URL (SaaS or self-hosted) | SaaS tools, existing servers | Lowest — paste URL, authenticate |
| **Managed** | Your infrastructure, MCP Manager-generated command | Custom servers, per-user instances | Medium — run command, configure SSH |
| **Workstation** | Local machine, encrypted tunnel | Local files, editors, hardware | Highest — tunnel setup, per-machine |

### Gateway rules: detection methods and actions

| Detection Method | What it catches | Actions available | When to use |
| --- | --- | --- | --- |
| **Regex** | Structured patterns (SSNs, API keys, IDs) | Block, Redact, Replace, Mask, Hash | Fast, deterministic, no external call |
| **Microsoft Presidio** | Contextual PII (names, emails, addresses) | Block, Replace | Unstructured data, model-driven |
| **Custom engine** | Nuanced policy (jailbreak, toxicity, domain logic) | Engine decides | Complex judgment, external service |

### Identity schemes: per-user vs. shared

| Scheme | Who authenticates | Best for | Trade-off |
| --- | --- | --- | --- |
| **Per-user identity** | Each user brings their own credential | Individual accountability, least privilege | Users must authenticate once per server |
| **Shared identity** | One service account for everyone | Frictionless reads, shared resources | No individual attribution |

### Feature provisioning modes

| Mode | What passes | When to use |
| --- | --- | --- |
| **Allowing all** | Every tool, prompt, resource the server offers | Simple setup, broad access |
| **Allow if conditions are met** | Only explicitly allowlisted capabilities | Least privilege, tool pinning, type-based gating |
| **Blocking all** | Nothing of that type | Disable prompts/resources you don't need |

### Common CLI and URL patterns

```text
# Gateway picker URL (user chooses gateway)
https://app.mcpmanager.ai/gateway/v1/mcp

# Gateway locked URL (specific gateway)
https://app.mcpmanager.ai/gateway/v1/mcp/<gateway-id>

# API token format
API_<token-string>

# OAuth callback (fixed for all servers)
https://app.mcpmanager.ai/api/v1/mcpm/inbound/oauth/callback
```

## Decision guidance

### When to use Remote vs. Managed vs. Workstation

| Scenario | Choose |
| --- | --- |
| Connecting a SaaS vendor (GitHub, Slack, Atlassian) | Remote |
| Running a command-based server in your infrastructure | Managed |
| Needing per-user or per-group instances | Managed |
| Accessing local files, editors, or hardware | Workstation |
| Already running a server at a URL | Remote |

### When to use per-user vs. shared identity

| Scenario | Choose |
| --- | --- |
| Every action must be attributed to a real person | Per-user |
| Reads are shared, writes are individual (add server twice) | Per-user for writes, Shared for reads |
| Frictionless shared access to a resource | Shared |
| Compliance requires non-repudiation | Per-user |

### When to use each gateway topology

| Scenario | Choose |
| --- | --- |
| Simple rollout, one link to manage | One org-wide gateway |
| Tools and policies per department | One gateway per team |
| Maximum isolation, easy evaluation | One gateway per server |
| Curated toolset for a workflow | One gateway per use case |

### When to use each rule detection method

| Scenario | Choose |
| --- | --- |
| Blocking known patterns (SSNs, API keys) | Regex |
| Detecting unstructured PII (names, emails) | Microsoft Presidio |
| Jailbreak, prompt injection, toxicity | Custom engine (Lakera Guard, Bedrock) |
| Domain-specific policy or field-level redaction | Custom engine (your own webhook) |

## Workflow

### Add a server and expose it through a gateway

1. **Add the server to MCP Manager** (not yet exposed to users)
   - Go to MCP Servers → Add
   - Choose server type: Remote (paste URL), Managed (provide SSH), or Workstation (local machine)
   - Authenticate (OAuth, token header, or no auth)
   - Name it and save

2. **Create a gateway** (the user-facing URL)
   - Go to Gateways → Add
   - Name the gateway
   - Provision it to one or more teams
   - Save

3. **Assign the server to the gateway**
   - Open the gateway → Servers tab
   - Click Assign a server
   - Select the server you added
   - Choose identity scheme (per-user or shared)
   - Confirm

4. **Provision tools** (decide what's exposed)
   - Open the server on the gateway
   - For each feature type (tools, prompts, resources):
     - Choose "Allow all" (default), "Allow if conditions are met" (allowlist), or "Block all"
     - If allowlisting, add specific tools and pin by name/title/description
   - Save

5. **Add gateway rules** (optional but recommended)
   - Open the gateway → Rules tab
   - Click Add new rule
   - Name it, choose detection method (regex, Presidio, custom engine)
   - Set detection hook (request, response, or both)
   - Configure the method and choose action (block, redact, replace, mask, hash)
   - Toggle alerts on if desired
   - Save

6. **Distribute the gateway URL to users**
   - Copy the gateway URL from the gateway overview
   - Share the picker URL (no gateway specified) for broad access, or locked URLs (gateway specified) for specific teams
   - Users connect their AI client and authorize once

### Create an API token for a headless agent

1. **Create a token-based host** (represents the agent)
   - Go to Apps & Agents → Add
   - Choose "Token-based host"
   - Name it (e.g., "Feedback bot")
   - Save

2. **Generate an API token**
   - Open the host → Connections tab
   - Click Add a connection
   - Choose the gateway the agent should reach
   - Authorize with any per-user servers (bring each user's identity)
   - Copy the token (starts with `API_`) and gateway URL
   - Store securely (secret manager, CI encrypted secrets, never source control)

3. **Use the token in the agent**
   - Agent presents the token as a Bearer token in the Authorization header
   - Token is scoped to that host and gateway connection only

### Set up a gateway rule to block PII

1. **Open the gateway → Rules tab**

2. **Click Add new rule**

3. **Configure the rule**
   - Name: "Block SSNs" (or similar)
   - Detection method: "Regular expression"
   - Detection hook: "Response" (or "Both" to catch it in requests too)
   - Pattern: `\b\d{3}-\d{2}-\d{4}\b` (SSN pattern)
   - Action: "Block"
   - Alerts: Toggle on

4. **Save**

5. **Test the rule**
   - Make a tool call that would return an SSN
   - Verify the call is blocked and an alert fires
   - Check the logs to see the rule activity

### Troubleshoot a failed connection

1. **Check the gateway's Connections tab**
   - Verify the host/agent is listed and enabled
   - Look for any disabled toggles (host, connection, identity)

2. **Check the server's status**
   - Open the gateway → Servers tab
   - Verify the server is assigned and enabled
   - Check if the identity is set (for per-user servers, each user must have authenticated)

3. **Review the audit logs**
   - Go to Logs
   - Filter by the gateway, server, or user
   - Look for `initialize` entries (connection handshake) and any errors
   - Check for `policy_enforced_abort` (rule blocked the call)

4. **Verify authentication**
   - For OAuth: check if the token is stale or revoked
   - For token headers: verify the token is correct and not expired
   - For managed servers: check SSH connectivity and the server's health

5. **Check feature provisioning**
   - Verify the tool the user is trying to call is in the allowlist
   - Look for `gateway_feature_blocked` or `gateway_feature_filtered` in logs

## Common gotchas

- **Servers are staged, not exposed**: Adding a server to MCP Manager doesn't make it available to users. You must create a gateway, assign the server to it, and provision the gateway to a team.

- **Feature provisioning defaults to "Allow all"**: When you first assign a server, every tool, prompt, and resource passes through. If you want least privilege, switch to "Allow if conditions are met" — but this immediately blocks everything until you add allowlist entries.

- **Rules apply to tools only**: Gateway rules run on `tools/call` requests and responses. They do **not** run on prompts (`prompts/get`) or resources (`resources/read`).

- **Regex rules run in-process, custom engines add latency**: A regex rule has no failure mode and adds ~0ms. A custom engine (Presidio, Lakera, Bedrock, your webhook) adds an external round-trip and has a 30-second timeout. Set failure mode to "Block" (default for custom engines) to fail closed on timeout.

- **Tool pinning by description freezes the tool**: If you pin a tool by name and description, and the upstream server later changes the description, the tool no longer matches and is dropped. This is intentional (defense against rug pulls), but it means you must re-approve if the vendor improves the description.

- **Identities are encrypted at rest, but tokens don't expire on their own**: OAuth tokens are refreshed automatically. API tokens and header tokens you provide do not expire unless you revoke them. Rotate them manually if needed.

- **Per-user identity requires each user to authenticate once**: If you set a server to per-user identity, each user must go through an OAuth flow or provide a token before they can use that server. This is a one-time step per server per user.

- **Shared identity with no identity selected returns an error**: If you set a server to shared identity but don't select which identity to use, calls fail with a clear error. An administrator must set the shared identity on the server within the gateway.

- **Disabling a host, connection, or identity takes effect immediately**: These toggles are checked on every request with no caching. Disable a host and the next call is rejected. Re-enable and the next call succeeds.

- **Rules are ordered and modifications chain**: Request-hook rules run in order; response-hook rules run in order. A "Block" action stops processing immediately. Modification actions (redact, replace, mask, hash) chain — each operates on the text the previous rule modified.

- **Workstation servers need outbound network access**: The encrypted tunnel from a workstation to the gateway requires outbound HTTPS. If the machine is behind a corporate proxy or secure web gateway, you may need to configure the tunnel to route through it.

## Verification checklist

Before deploying a gateway or making changes:

- [ ] **Server is added and authenticated**: Verify the server appears in MCP Servers and shows a green status
- [ ] **Gateway is created and provisioned**: Confirm the gateway exists and is assigned to the right team(s)
- [ ] **Server is assigned to the gateway**: Check the gateway's Servers tab lists the server
- [ ] **Identity scheme is set**: Verify per-user or shared identity is chosen, and the identity exists
- [ ] **Feature provisioning is correct**: Confirm tools, prompts, resources are set to the right mode (Allow all, Allow if conditions, or Block all)
- [ ] **Allowlist entries are added** (if using "Allow if conditions"): Verify the specific tools you want are listed
- [ ] **Gateway rules are in place** (if needed): Check the Rules tab has the rules you expect, in the right order, with alerts enabled
- [ ] **Test a tool call**: Make a real call through the gateway and verify it succeeds
- [ ] **Check the audit log**: Confirm the call appears in Logs with the right user, gateway, server, and tool
- [ ] **Verify rule enforcement**: If a rule should block or modify, trigger it and confirm the action in the logs and alerts

## Resources

- **Comprehensive page listing**: https://docs.mcpmanager.ai/llms.txt — full navigation of all documentation pages for agent reference
- **Introduction & core concepts**: https://docs.mcpmanager.ai/get-started/introduction — the mental model and architecture
- **MCP Servers overview**: https://docs.mcpmanager.ai/mcp-gateway-concepts/mcp-servers/overview — Remote, Managed, Workstation types and when to use each
- **Gateway Rules overview**: https://docs.mcpmanager.ai/features/gateway-rules/overview — detection methods, actions, failure modes, and rule ordering
- **Authentication & Identity**: https://docs.mcpmanager.ai/security/authentication-and-identity — the two-authentication model, per-user vs. shared, credential storage
- **Gateway Deployment Strategies**: https://docs.mcpmanager.ai/deployment/gateway-deployment-strategies — choosing a topology (org-wide, per-team, per-server, per-use-case)

---

> For additional documentation and navigation, see: https://docs.mcpmanager.ai/llms.txt