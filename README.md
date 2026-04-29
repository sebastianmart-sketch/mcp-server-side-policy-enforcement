# MCP Server-Side Policy Enforcement Extension Proposal

This is my proposal for a new MCP extension: **Server-Side Policy Enforcement**.

It introduces a lightweight Policy Enforcement Point inside the MCP Server so that governance, including authorization, actor validation, approval workflows and discovery filtering, happens at the execution point instead of relying only on the client or gateways.

## Full proposal

[SEP-XXXX: Server-Side Policy Enforcement Extension for MCP Servers](./SEP-XXXX-Server-Side-Policy-Enforcement.md)

## Key goals

- True Zero Trust client posture
- Server-side enforcement at discovery and execution time
- Actor-based governance, including `human_required` and `supervised_ai_required`
- External validation contract with flexible backends
- Optional static RBAC for small/local servers
- Optional structured audit
- Support for `require_approval_via: task | elicitation | none`

## Status

This is an early community draft. Feedback is very welcome.

I plan to open a formal SEP PR in the official MCP repository after gathering initial comments.

---

Author: Sebastian Martinez, individual contributor  
Created: 2026-04-28
