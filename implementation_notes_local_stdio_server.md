# Local stdio MCP Server implementation notes

## Purpose

This note explains why server-side policy enforcement is important for local MCP Servers, especially those using stdio transport.

## Problem

A local MCP Server may run on the same machine as the MCP Client. It may expose powerful local capabilities such as:

- file access;
- shell execution;
- package management;
- service restart;
- system inspection;
- local credentials or configuration;
- administrative scripts.

In this model, a network gateway may not exist.

The server cannot assume that the client is the correct policy authority.

## Recommended implementation pattern

A local stdio MCP Server should:

- load policy from a local trusted path;
- validate the local user or process context where possible;
- deny protected capabilities by default;
- filter discovery based on local policy where practical;
- enforce policy again at execution time;
- avoid trusting client-provided actor or approval claims;
- fail closed if policy cannot be loaded or evaluated.

## Example

A local MCP Server exposes this tool:

```json
{
  "name": "restart_service",
  "description": "Restart a local system service"
}
```

Policy may require:

- local administrator identity;
- human actor validation;
- explicit approval;
- denial when invoked by automation.

Even if the tool is hidden during discovery, the server must still deny direct invocation unless policy allows execution.

