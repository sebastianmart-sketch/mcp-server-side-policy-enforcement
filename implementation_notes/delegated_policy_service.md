# Delegated policy service implementation notes

## Purpose

This note describes how an MCP Server can delegate policy evaluation while remaining the final enforcement point.

## Architecture

```text
MCP Client
   |
   | tools/call
   v
Protected MCP Server  ---- policy decision request ----> Policy Decision Service
   |                                                        |
   | <---- decision: allow / deny / require_approval -------|
   v
Execute or deny locally
```

## Key rule

The protected MCP Server owns enforcement.

The external policy service may decide, but it does not execute or expose the protected capability directly.

## Failure behavior

The MCP Server must fail closed when:

- the policy service is unreachable;
- the policy service times out;
- the response is malformed;
- the decision is unsupported;
- the policy service returns an error;
- the server cannot validate required actor or approval constraints.

## Benefits

A delegated policy service allows organizations to use existing policy systems while keeping MCP enforcement consistent.

Possible backends include:

- OPA/Rego;
- Cedar;
- enterprise IdP policy;
- database-backed policy;
- change-management systems;
- custom governance services.

## Risks

Implementations should avoid:

- turning the MCP Client into the policy authority;
- exposing sensitive arguments unnecessarily to the policy service;
- creating recursive policy dependencies without clear trust boundaries;
- allowing fail-open behavior during policy service outages.

