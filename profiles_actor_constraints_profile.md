# Actor constraints profile

## Status

Non-normative implementation profile.

## Purpose

This profile describes possible actor constraint names and validation expectations for MCP server-side policy enforcement.

Actor constraints are policy requirements. They are not final authorization decisions.

## Example actor constraints

- `human`
- `supervised_ai`
- `automation`
- `service_account`

## Semantics

### human

The server has trusted evidence that the operation is being performed by or explicitly approved by a human actor.

### supervised_ai

The server has trusted evidence that an AI-assisted action is operating under a human-supervised session, workflow or approval model.

### automation

The server has trusted evidence that the action comes from an automation workflow.

### service_account

The server has trusted evidence that the action comes from a service account or machine identity.

## Trust model

The MCP Server must not trust a client-supplied actor claim by default.

Trusted actor evidence may come from:

- authenticated identity claims;
- enterprise IdP attributes;
- mTLS identity;
- local OS user identity;
- PAM or host session context;
- token-bound client identity;
- server-side session state;
- a trusted approval service;
- a trusted policy service.

If no trusted evidence is available, actor-constrained requests must fail closed.

## Open question

The core SEP should decide whether actor constraint names are standardized in v1 or left to implementation profiles.

