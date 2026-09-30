# ADR-0001: Workspace cut by domain and stage

- **Status:** Accepted (example)
- **Date:** 2026-10-01
- **Owner:** Platform architect
- **Confidence:** high

## Context

Several teams build on the same Fabric tenant. Starting with one workspace for everything is fast, but after a few months nobody finds the current version, a change in production can't be traced, and access can't be reviewed.

## Options considered

1. One workspace for everything.
2. One workspace per team.
3. One workspace per domain and per stage (Dev, Test, Prod), with core data domains and consumer domains.

## Decision

Option 3. Every domain gets one workspace per stage, named `{Domain}-{Env}-{Purpose}`, for example Finance-Dev-Data, Finance-Prod-Data, Sales-Prod-Reports. Core data domains hold Bronze, Silver and Gold; consumer domains hold reports and read Gold by shortcut.

## Why

- The workspace is the unit for roles, Git integration and deployment: workspaces define access through four roles, support Git integration and "serve as the scope for deployment pipelines" ([Choose a Fabric deployment pattern](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447)).
- Consumer domains "use shortcuts to reference producer data products rather than copying them" ([OneLake patterns: domain-oriented data mesh](https://learn.microsoft.com/fabric/onelake/architecture-patterns?wt.mc_id=AZ-MVP-5003447)).
- Consistent names help people find content ([Workspace naming conventions](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning?wt.mc_id=AZ-MVP-5003447)).

## Consequences and trade-offs

- More workspaces to create; creation is scripted, names follow [naming.md](../naming.md).
- Each stage needs its own data, connections and schedules; releases run from Git.
- Access is reviewed per domain: the domain owner approves membership of a builders group and a readers group.

## Links

- [naming.md](../naming.md)
- [ADR-0002](0002-lakehouse-lakehouse-lakehouse.md)
