# SEP-XXXX: Server-side policy enforcement for MCP capabilities

## Status

Draft proposal for early community review.

## Summary

This proposal defines a minimal, backend-neutral extension for server-side policy enforcement in MCP.

The extension allows an MCP Server to advertise that it evaluates policy before exposing or executing protected MCP capabilities, including tools, resources, prompts and future capability types. The protected MCP Server remains the Policy Enforcement Point, while policy evaluation may be local or delegated to an external trusted policy service.

The proposal does not define a policy language, RBAC model, ABAC model, audit backend, identity provider or authorization protocol. It defines the enforcement contract that an MCP Server can implement regardless of the underlying policy system.

## Motivation

MCP enables clients and agents to discover and invoke capabilities across many systems. As MCP is adopted for operational, administrative and enterprise workflows, access control cannot rely only on the MCP Client, a gateway or user confirmation UI.

The server that owns a protected capability is the last responsible authority before that capability is exposed or executed.

This proposal addresses the gap between authentication and capability-level authorization.

MCP authorization can answer:

> Can this client obtain and present credentials to access this MCP Server?

Server-side policy enforcement answers:

> Given this authenticated context, should this specific capability be listed, read or executed now, with these parameters, for this actor, tenant, environment and approval state?

## Core principle

An MCP Server that supports this extension MUST NOT trust the MCP Client by default as the authority for:

- authorization decisions;
- actor validation;
- approval state;
- policy selection;
- enforcement mode;
- capability visibility.

The MCP Client may present identity, session, approval or request context, but the protected MCP Server MUST validate any security-relevant context using server-trusted evidence before relying on it.

If the server cannot validate the required context, the request MUST fail closed.

## Goals

This proposal aims to standardize a minimal MCP extension for:

- advertising server-side policy enforcement support;
- enforcing policy before protected capability execution;
- optionally filtering capability discovery based on policy;
- supporting actor and approval constraints without trusting client claims;
- allowing local or external policy decision backends;
- defining safe fail-closed behavior;
- providing a common decision model for MCP implementations.

## Non-goals

This proposal does not define:

- a policy language;
- a required RBAC or ABAC schema;
- a required audit log format;
- a required policy engine;
- a required identity provider;
- a replacement for OAuth, enterprise IdPs or MCP authorization;
- a replacement for gateway enforcement;
- a universal human/AI attestation mechanism.

## Relationship to existing MCP authorization

MCP authorization governs whether a client can obtain and present credentials to access an MCP Server.

This extension governs whether the MCP Server exposes or executes a specific MCP capability after authentication, identity and request context have been resolved.

These layers are complementary.

| Layer | Main question | Replaced by this extension? |
|---|---|---|
| MCP authorization / OAuth | Can a client access the MCP Server? | No |
| Enterprise IdP policy | Is this principal authenticated and authorized at the identity layer? | No |
| Gateway or proxy enforcement | Can traffic reach the server or method? | No |
| MCP server-side policy enforcement | Can this capability be exposed or executed now? | Yes, this extension defines that contract |
| Client confirmation UI | Did the user approve an action in the client? | No, but the server may require trusted approval evidence |

## Relationship to gateways

Gateways and proxies are valuable policy enforcement points, but they are not always present and should not be the only enforcement point.

MCP Servers may run locally, over stdio, in embedded environments, in air-gapped networks, on the same host as the client or inside administrative workflows where the gateway is bypassed or unavailable.

For this reason, the protected MCP Server remains the final enforcement point for the capabilities it exposes.

A gateway may make a preliminary decision, but the MCP Server MUST remain able to deny discovery or execution of protected capabilities based on its own trusted policy context.

## Extension declaration

A server that supports this proposal declares the extension during MCP initialization using the standard MCP extension negotiation mechanism.

Example:

```json
{
  "capabilities": {
    "extensions": {
      "io.modelcontextprotocol/server-policy-enforcement": {
        "version": "1.0",
        "supportsDiscoveryFiltering": true,
        "supportsActorConstraints": true,
        "supportsApprovalConstraints": true,
        "supportsExternalDecision": true,
        "supportsAuditEvents": false
      }
    }
  }
}
```

The exact extension identifier and metadata shape should be aligned with the MCP extension mechanism and maintainers' guidance.

Earlier drafts or non-extension capabilities may appear directly under `capabilities`. This proposal is intended as an MCP extension and therefore uses the extension negotiation mechanism.

## Enforcement points

A conformant MCP Server MUST evaluate policy before executing protected capability methods.

At minimum, this includes protected operations such as:

- tool invocation;
- resource read or subscription;
- prompt access;
- task creation or execution, if supported;
- future MCP capability execution methods.

A conformant MCP Server MAY also evaluate policy during discovery methods, such as list operations for tools, resources or prompts.

Discovery filtering improves least privilege and reduces accidental exposure. However, discovery filtering alone is not sufficient.

If a capability is hidden during discovery, the server MUST still enforce policy if a client attempts to invoke or access it directly.

## Minimal conformance

A minimally conformant server:

1. declares support for this extension using MCP extension negotiation;
2. evaluates policy before executing protected capability methods;
3. fails closed when policy cannot be evaluated;
4. does not accept client-supplied policy as authoritative;
5. does not allow the client to select enforcement mode;
6. validates security-relevant actor, approval and session context using server-trusted evidence;
7. returns a safe denial response that does not leak sensitive policy details.

Discovery filtering, audit events, delegated policy decision services and approval workflow integration are optional profiles.

## Decision model

Policy evaluation SHOULD produce one of the following decisions:

- `allow`
- `deny`
- `require_approval`

The server MAY internally support additional states, such as `indeterminate`, `not_applicable` or `error`, but these MUST be mapped to `deny` or another fail-closed result before execution.

Actor requirements and approval routing are not decision values. They are constraints that must be satisfied before a final decision is enforced.

A policy decision MAY include constraints such as:

```json
{
  "decision": "require_approval",
  "constraints": {
    "actor_required": "human",
    "approval_required": true,
    "require_approval_via": "task"
  },
  "reason_code": "PRODUCTION_CHANGE_REQUIRES_APPROVAL"
}
```

The following constraint values are suggested for interoperability:

```text
actor_required: human | supervised_ai | automation | service_account
require_approval_via: task | elicitation | none
```

`human_required` and `supervised_ai_required` MAY be represented by implementations as shorthand policy rules, but the interoperable decision result SHOULD express them as actor constraints, for example `actor_required: human` or `actor_required: supervised_ai`.

If the server cannot validate that the required actor or approval constraint has been satisfied, the decision MUST be treated as `deny` or remain blocked as `require_approval` until trusted evidence is available.

## Actor constraints

Policy may require that a capability be invoked only by a specific class of actor.

Example actor constraints include:

- `human`
- `supervised_ai`
- `automation`
- `service_account`

This extension does not standardize actor attestation.

The MCP Server MUST determine actor state using server-trusted identity and session context. The client may present actor information, but client-supplied actor claims MUST NOT be trusted by default.

Trusted actor evidence may come from implementation-specific sources such as:

- authenticated identity claims;
- enterprise IdP attributes;
- token-bound client identity;
- local operating system identity;
- PAM or host session context;
- mTLS identity;
- server-side session state;
- a trusted approval service;
- a trusted policy decision service.

If no trusted evidence is available, actor-constrained requests MUST fail closed.

## Approval constraints

Policy may require approval before a protected capability is executed.

Approval requirements are constraints, not standalone authorization decisions.

Example:

```json
{
  "decision": "require_approval",
  "constraints": {
    "approval_required": true,
    "require_approval_via": "task"
  },
  "reason_code": "APPROVAL_REQUIRED"
}
```

The suggested values for `require_approval_via` are:

- `task`: approval should be mediated through a task or change workflow;
- `elicitation`: approval should be mediated through an explicit elicitation step;
- `none`: no additional approval channel is required by policy.

The server MUST NOT treat a client assertion of approval as authoritative unless it can validate the approval using server-trusted evidence.

Approval evidence may be implementation-specific and may include:

- a server-side approval record;
- a signed approval token;
- a trusted task approval state;
- a trusted elicitation result;
- an enterprise change-management record.

If approval is required but cannot be validated, the server MUST deny execution or return a safe `require_approval` response.

## Default-deny fallback

Policy implementations SHOULD provide an explicit default-deny fallback for unknown, future or unclassified MCP methods.

Reference profiles MAY represent this fallback as `_other: deny`.

This fallback is important because MCP may evolve with new methods or capability types. A server-side policy implementation should not accidentally allow an unknown future method simply because no explicit policy entry exists for it.

Example non-normative method policy:

```yaml
_methods:
  tools/list: filter
  tools/call: evaluate
  resources/list: filter
  resources/read: evaluate
  prompts/list: filter
  prompts/get: evaluate
  _other: deny
```

## External policy decision services

A server MAY delegate policy evaluation to an external trusted policy service.

In this model:

- the MCP Server remains the Policy Enforcement Point;
- the external service acts as a Policy Decision Point;
- the MCP Server sends request context to the policy service;
- the policy service returns a decision and constraints;
- the MCP Server enforces the result locally.

The external policy service may use any backend, including RBAC, ABAC, Rego/OPA, Cedar, database-backed policy, enterprise IdP policy, local files or custom logic.

If the external policy service is unavailable, returns an invalid decision, times out or produces an unsupported result, the MCP Server MUST fail closed.

### Non-normative external decision request example

```json
{
  "subject": {
    "principal": "user:alice@example.com",
    "actor_type": "human",
    "groups": ["sre"],
    "tenant": "example-corp"
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
    "deployment": "production",
    "risk": "high"
  },
  "approval": {
    "present": false
  }
}
```

### Non-normative external decision response example

```json
{
  "decision": "require_approval",
  "constraints": {
    "actor_required": "human",
    "approval_required": true,
    "require_approval_via": "task"
  },
  "reason_code": "PRODUCTION_CHANGE_REQUIRES_APPROVAL",
  "safe_message": "This operation requires approval before execution."
}
```

This example is illustrative only. Implementations may use different transport, schema, policy engines or validation backends.

## Fail-closed behavior

A conformant server MUST deny or safely block execution when:

- policy cannot be loaded;
- policy cannot be evaluated;
- an external policy service is unavailable;
- a policy decision is malformed;
- actor constraints cannot be validated;
- approval constraints cannot be validated;
- the requested method is unknown or unsupported;
- the capability is protected but no applicable policy exists;
- the policy backend returns an error or indeterminate result.

The server SHOULD avoid returning sensitive policy internals in error messages.

## Error and response guidance

A denied request SHOULD return a safe error or refusal response.

The response MAY include a stable reason code suitable for client behavior or audit correlation.

Example reason codes:

- `POLICY_DENIED`
- `POLICY_UNAVAILABLE`
- `POLICY_INDETERMINATE`
- `ACTOR_VALIDATION_FAILED`
- `APPROVAL_REQUIRED`
- `APPROVAL_VALIDATION_FAILED`
- `CAPABILITY_NOT_VISIBLE`
- `CAPABILITY_NOT_ALLOWED`
- `UNKNOWN_METHOD_DENIED`

Human-readable messages SHOULD be generic unless the server is configured to expose more detail.

Example:

```json
{
  "error": {
    "code": "POLICY_DENIED",
    "message": "The server policy does not allow this operation."
  }
}
```

## Non-normative YAML reference profile

The following YAML example illustrates one possible static policy profile.

This proposal does not require YAML, RBAC or this schema. The example is included only to clarify how the minimal enforcement contract could be implemented.

```yaml
version: 1

_settings:
  default_decision: deny
  client_claims_authoritative: false
  policy_error: deny
  unknown_capability: deny

_methods:
  tools/list: filter
  tools/call: evaluate
  resources/list: filter
  resources/read: evaluate
  prompts/list: filter
  prompts/get: evaluate
  _other: deny

roles:
  sre:
    description: Site reliability engineers with controlled operational access.
    grants:
      - capability: tool:read_logs
        actions: [execute]
        decision: allow

      - capability: tool:restart_service
        actions: [execute]
        decision: require_approval
        constraints:
          actor_required: human
          approval_required: true
          require_approval_via: task

  automation:
    description: Automation service account with narrow permissions.
    grants:
      - capability: tool:collect_metrics
        actions: [execute]
        decision: allow
        constraints:
          actor_required: service_account

      - capability: tool:restart_service
        actions: [execute]
        decision: deny
```

A fuller reference profile may be maintained outside the core SEP as implementation guidance.

## Optional profiles

The following profiles are non-normative and may be developed separately.

### Discovery filtering profile

Defines how a server filters list responses for tools, resources, prompts or future capabilities based on policy.

Discovery filtering is useful for least privilege, but execution enforcement remains mandatory.

### Actor constraint profile

Defines common actor constraint names and expected validation semantics.

This profile may include actor types such as `human`, `supervised_ai`, `automation` and `service_account`.

### Approval profile

Defines how `require_approval` decisions interact with MCP mechanisms such as elicitation, tasks or future approval primitives.

This profile should not require a single approval transport in the core proposal.

### External decision profile

Defines a reference request/response shape for delegating policy decisions to an external Policy Decision Point.

This profile should remain backend-neutral.

### Static RBAC reference profile

Defines a simple local RBAC mapping for small deployments, local MCP Servers or reference implementations.

This profile is useful for examples, but should not be part of the normative core.

### Structured audit profile

Defines optional audit events for policy evaluation, discovery filtering and execution decisions.

Audit is important for enterprise deployments, but may deserve a separate extension to avoid overloading the core policy proposal.

## Example: protected tool invocation

A client invokes a protected tool:

```json
{
  "method": "tools/call",
  "params": {
    "name": "restart_service",
    "arguments": {
      "service": "database"
    }
  }
}
```

The server evaluates policy using trusted context:

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
    "tenant": "production",
    "risk": "high"
  }
}
```

The policy decision requires approval:

```json
{
  "decision": "require_approval",
  "constraints": {
    "actor_required": "human",
    "approval_required": true,
    "require_approval_via": "task"
  },
  "reason_code": "PRODUCTION_CHANGE_REQUIRES_APPROVAL"
}
```

The server does not execute the tool until trusted approval evidence is available.

## Security considerations

This extension is designed to prevent MCP Clients from becoming the sole authority for policy enforcement.

Important security properties:

- the protected MCP Server remains the final enforcement point;
- discovery filtering is never a substitute for execution enforcement;
- actor and approval claims from the client are not trusted by default;
- unsupported, malformed or unavailable policy decisions fail closed;
- external policy services do not move enforcement out of the protected server;
- unknown or future methods can be covered by an explicit default-deny fallback such as `_other: deny`;
- denial responses should avoid leaking sensitive policy details.

## Why this is needed

Without a server-side enforcement contract, MCP deployments may rely on inconsistent combinations of client confirmation, gateway controls, OAuth scopes, local configuration and implementation-specific access checks.

That may be sufficient for simple demos, but it is weak for enterprise and operational environments where MCP capabilities can affect infrastructure, data, services, compliance posture or production availability.

This proposal provides a minimal common contract for MCP Servers to say:

> I enforce policy before exposing or executing protected capabilities, and I do not trust the client as the policy authority.

## Open questions

1. Should the core decision model include only `allow`, `deny` and `require_approval`, or should `indeterminate` be exposed to clients?
2. Should discovery filtering be required for conformant servers or remain optional?
3. Should approval integration be part of the core extension or a separate profile?
4. Should structured audit events be included in this SEP or split into a separate audit extension?
5. What extension identifier should be used for a formal MCP SEP?
6. Should actor constraint names be standardized in v1 or left to implementation profiles?
7. Should `require_approval_via` values be standardized in the core extension or moved to the approval profile?
8. Should a reference external decision contract be standardized, or should the core only require decision semantics?
9. Should `_other: deny` be a required behavior or a strongly recommended reference-profile convention?

## Recommended SEP framing

The proposal should not be framed as:

> MCP needs a new policy language.

It should be framed as:

> MCP needs a minimal server-side enforcement contract so servers can consistently apply policy at discovery and execution time without trusting the client as the policy authority.

## Recommended title

**Server-side policy enforcement for MCP capabilities**

Alternative:

**Capability-level server-side policy enforcement for MCP**

## Recommended one-line pitch

This extension defines a minimal, backend-neutral contract for MCP Servers to enforce policy at capability discovery and execution time, while preserving the server as the final Policy Enforcement Point.

## AI usage disclosure

Generative AI was used as an editorial and technical sparring partner while developing this draft. It helped review structure, identify possible gaps and refine wording. The core idea, proposal direction and final editorial judgment are mine.

