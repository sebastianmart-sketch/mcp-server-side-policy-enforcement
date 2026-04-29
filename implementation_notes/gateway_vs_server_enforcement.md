# Gateway enforcement vs server-side enforcement

## Purpose

This note clarifies the relationship between gateway enforcement and server-side policy enforcement.

## Gateways are useful

Gateways and proxies can enforce important controls, including:

- authentication;
- coarse-grained authorization;
- rate limiting;
- tenant routing;
- network segmentation;
- request logging;
- centralized policy checks.

## Gateways are not sufficient by themselves

Gateways may not exist in every MCP deployment.

Examples:

- local stdio MCP Servers;
- same-host client/server deployments;
- embedded systems;
- air-gapped environments;
- administrative scripts;
- direct internal service calls;
- development tools;
- edge environments.

Even when a gateway exists, it may not understand the full capability semantics, local system state, runtime context or safety requirements of the protected MCP Server.

## Server-side enforcement remains necessary

The MCP Server that owns a protected capability is the last responsible authority before that capability is exposed or executed.

Therefore:

- a gateway may allow a request to reach the server;
- the server may still deny the capability;
- discovery filtering at the gateway is not enough;
- discovery filtering at the server is not enough;
- execution-time enforcement remains mandatory for protected capabilities.

## Recommended framing

This proposal does not compete with gateway enforcement.

It complements gateway enforcement by defining the final capability-level enforcement point inside the MCP Server.

