# Structured audit profile

## Status

Non-normative implementation profile.

## Purpose

This profile describes optional audit events for MCP server-side policy enforcement.

Audit is important for enterprise deployments, but the core SEP should not require a specific audit backend or event schema.

## Recommended audit events

An implementation may record events for:

- policy evaluation;
- policy denial;
- approval required;
- approval validated;
- actor validation failure;
- discovery filtering;
- protected capability execution;
- external policy decision errors.

## Example event

```json
{
  "event_type": "mcp.policy.decision",
  "timestamp": "2026-04-29T09:30:00Z",
  "server": "example-mcp-server",
  "principal": "user:alice@example.com",
  "actor_type": "human",
  "capability": {
    "type": "tool",
    "name": "restart_service"
  },
  "action": "execute",
  "decision": "require_approval",
  "reason_code": "PRODUCTION_CHANGE_REQUIRES_APPROVAL",
  "environment": {
    "deployment": "production"
  }
}
```

## Privacy and security considerations

Audit events should avoid recording sensitive arguments unless explicitly required.

Implementations should consider:

- redacting secrets;
- minimizing personal data;
- separating policy reason codes from detailed internal policy logic;
- protecting audit logs from tampering;
- preserving correlation IDs for investigations.

## Non-goals

This profile does not define:

- a required log format;
- a required SIEM integration;
- a required storage backend;
- a required retention period.

