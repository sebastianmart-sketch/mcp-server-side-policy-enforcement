# Approval profile

## Status

Non-normative implementation profile.

## Purpose

This profile describes how a server-side policy decision may require approval before executing a protected MCP capability.

Approval requirements are constraints, not standalone authorization decisions.

## Decision example

```json
{
  "decision": "require_approval",
  "constraints": {
    "approval_required": true,
    "approval_via": "task",
    "actor_required": "human"
  },
  "reason_code": "APPROVAL_REQUIRED"
}
```

## Approval evidence

Approval evidence may come from:

- a server-side approval record;
- a signed approval token;
- a trusted task approval state;
- a trusted elicitation result;
- an enterprise change-management system.

## Trust model

The MCP Server must not treat client-supplied approval claims as authoritative by default.

The MCP Server must validate approval evidence using server-trusted context.

If approval is required but cannot be validated, the server must deny execution or return a safe approval-required response.

## Possible approval mechanisms

Implementations may integrate approval with:

- MCP elicitation;
- MCP tasks, if available;
- enterprise change approval systems;
- local administrative prompts;
- ticketing systems;
- signed approval workflows.

The core SEP should not require one approval mechanism.

