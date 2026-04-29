# Static RBAC reference profile

## Status

Non-normative implementation profile.

## Purpose

This profile describes one possible static RBAC implementation for MCP server-side policy enforcement.

It is useful for:

- local MCP Servers;
- small deployments;
- demos and reference implementations;
- environments where a full external policy engine is unnecessary.

The core SEP does not require static RBAC, YAML or this schema.

## Model

A static RBAC policy maps principals or groups to roles, and roles to capability grants.

A grant may include:

- capability type;
- capability name;
- action;
- decision;
- actor constraints;
- approval constraints;
- environment constraints.

## Example

```yaml
roles:
  sre:
    grants:
      - capability: tool:restart_service
        actions: [execute]
        decision: require_approval
        constraints:
          actor_required: human
          approval_required: true
```

## Recommended behavior

A static RBAC implementation should:

- fail closed if the policy file cannot be loaded;
- fail closed if the policy file is malformed;
- fail closed if no grant matches a protected capability;
- treat client-provided role claims as untrusted unless validated by the server;
- keep execution enforcement separate from discovery filtering.

## Non-goals

This profile does not define:

- a universal RBAC schema;
- an enterprise role model;
- policy inheritance;
- policy conflict resolution beyond fail-closed defaults;
- a required storage format.

