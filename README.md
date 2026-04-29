# MCP Server-Side Policy Enforcement

Draft proposal for an Model Context protocol (MCP) extension that defines a minimal, backend-neutral contract for server-side policy enforcement.

## Purpose

MCP enables clients and agents to discover and invoke capabilities across many systems. As MCP is adopted for operational, administrative and enterprise workflows, access control cannot rely only on the MCP Client, a gateway or user confirmation UI.

This repository proposes a server-side policy enforcement extension for MCP.

The core idea is simple:

> The MCP Server that owns a protected capability must remain the final Policy Enforcement Point before that capability is exposed or executed.

## What this proposal defines

The core SEP defines a minimal contract for MCP Servers to:

- advertise support for server-side policy enforcement;
- enforce policy before protected capability execution;
- optionally filter capability discovery based on policy;
- fail closed when policy cannot be evaluated;
- avoid trusting the MCP Client as the policy authority;
- validate actor, approval and session context using server-trusted evidence;
- support local or external policy decision backends without prescribing a policy language.

## What this proposal does not define

This proposal does not define:

- a required policy language;
- a required RBAC or ABAC schema;
- a required audit format;
- a required policy engine;
- a required identity provider;
- a replacement for MCP authorization, OAuth, gateways or enterprise IdPs;
- a universal mechanism for proving whether the actor is human, AI-assisted or automated.

## Why this matters

MCP authorization and OAuth can answer:

> Can this client obtain and present credentials to access this MCP Server?

Server-side policy enforcement answers:

> Given this authenticated context, should this specific capability be listed, read or executed now, with these parameters, for this actor, tenant, environment and approval state?

Both are needed.

## Repository contents

- [`SEP-XXXX-Server-Side-Policy-Enforcement.md`](./SEP-XXXX-Server-Side-Policy-Enforcement.md) contains the minimal normative proposal.
- [`examples/`](./examples/) contains non-normative examples for policy files, policy decisions, protected tool calls and discovery filtering.
- [`profiles/`](./profiles/) contains optional implementation profiles that may later become separate extensions or implementation guides.
- [`implementation_notes/`](./implementation_notes/) contains deployment guidance for local servers, delegated policy services and gateway/server enforcement boundaries.

## Design principle

This proposal does not attempt to standardize all policy enforcement.

It standardizes the minimal MCP contract that allows a server to say:

> I enforce policy before exposing or executing protected capabilities, and I do not trust the client as the policy authority.

## Status

Early draft for community feedback.

This is not an official MCP SEP.

## AI usage disclosure

Generative AI was used as an editorial and technical sparring partner while developing this draft. The core idea, proposal direction and final editorial judgment are mine.

