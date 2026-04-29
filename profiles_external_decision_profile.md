# External policy decision profile

## Status

Non-normative implementation profile.

## Purpose

This profile describes one possible pattern for delegating policy evaluation to an external trusted Policy Decision Point.

The protected MCP Server remains the Policy Enforcement Point.

## Roles

- MCP Server: owns and protects the capability.
- External Policy Decision Point: evaluates policy and returns a decision.
- MCP Client: requests capability discovery or execution, but is not the policy authority.

## Request shape

A policy decision request may include:

```json
{
  "subject": {
    "principal": "user:alice@example.com",
    "actor_type": "human",
    "groups": ["sre"]
  },
  "capability": {
    "type": "tool",
    "name": "restart_service"
  },
  "action": "execute",
  "arguments": {
    "service": "database"
  },
  "environment": {
    "deployment": "production"
  }
}
```

## Response shape

A policy decision response may include:

```json
{
  "decision": "require_approval",
  "constraints": {
    "actor_required": "human",
    "approval_required": true,
    "approval_via": "task"
  },
  "reason_code": "PRODUCTION_CHANGE_REQUIRES_APPROVAL"
}
```

## Required enforcement behavior

The MCP Server must enforce the result locally.

If the external decision service is unavailable, times out, returns an invalid response or returns an unsupported decision, the MCP Server must fail closed.

## Backend neutrality

The external service may use any backend, including:

- OPA/Rego;
- Cedar;
- database-backed policy;
- enterprise IdP policy;
- local policy files;
- custom application logic.

The core SEP does not require any specific backend.

