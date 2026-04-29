# MCP Server-Side Policy Enforcement Extension Proposal

This is my proposal for a new MCP extension: **Server-Side Policy Enforcement**.

It introduces a lightweight Policy Enforcement Point inside the MCP Server so that governance (authorization, actor validation, approval workflows, discovery filtering, etc.) happens at the execution point instead of relying only on the client or gateways.

**Full Proposal (SEP draft):**
[SEP-XXXX-Server-Side-Policy-Enforcement.md](./SEP-XXXX-Server-Side-Policy-Enforcement.md)

**Key goals:**
- True Zero Trust client posture
- Actor-based governance (`human_required`, `supervised_ai_required`)
- External validation contract (flexible backend)
- Optional static RBAC for small/local servers
- Optional structured audit
- Support for `require_approval_via: task | elicitation | none`

Feedback is very welcome!

I plan to open a formal SEP PR in the official MCP repository after gathering initial comments.

---

Author: Sebastian Martinez (individual contributor)  
Created: 2026-04-28
