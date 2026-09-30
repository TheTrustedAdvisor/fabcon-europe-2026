# ADR-0001: Workspace cut by domain, layer and stage

- **Status:** Accepted (example)
- **Status history:** 2026-10-01 Proposed; 2026-10-01 Accepted (example)
- **Date:** 2026-10-01
- **Owner:** Platform architect, together with the domain owners
- **Confidence:** high for the stage split; medium for the layer split, because it multiplies workspaces and groups
- **Revisit when:** a core data domain has one engineer and one source system, or scripted workspace creation is not in place after three months
- **Related:** [ADR-0002](0002-lakehouse-lakehouse-lakehouse.md) (what goes into the layer workspaces), [ADR-0003](0003-capacity-by-workload.md) (which capacity each workspace uses), [ADR-0004](0004-release-path-fabric-cicd.md) (how content moves between stage workspaces), [ADR-0005](0005-permissions-onelake-security.md) (who gets which role)

## Context

The example organisation runs several teams on one Fabric tenant. Finance owns the revenue and invoice data that every other team needs. Sales and Finance both build reports on that data. Nothing exists yet; the first workspace is about to be created.

Starting with one workspace for everything is fast. After a few months nobody finds the current version, a change in production can't be traced to a person or a commit, and nobody can say who can read what. The structure has to be decided before the first item, because moving items between workspaces later is work, and names, shortcuts and bookmarks point at the old place.

## Decision

1. **Core data domains** (the domains that produce and own data, here Finance) get **one workspace per medallion layer and per stage**: `Finance-Dev-Bronze`, `Finance-Dev-Silver`, `Finance-Dev-Gold`, and the same for Test and Prod. Nine workspaces per core data domain.
2. **Consumer domains** (the domains that build reports on top, here Sales, and Finance's own reporting) get **one workspace per stage**: `Sales-Dev-Reports`, `Sales-Test-Reports`, `Sales-Prod-Reports`. Inside, content is organised with workspace folders, one per team that builds reports there. Consumers read Gold by shortcut, never by copy.
3. Every workspace, Dev and Test included, is assigned to its Fabric domain. Names follow [naming.md](../naming.md).

## Options considered

The first options follow Microsoft's deployment patterns ([Choose a Microsoft Fabric deployment pattern](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447)); the last ones add the layer question.

1. **One workspace for everything** (Microsoft's monolithic pattern). *Rejected because* there is no stage separation: "Features such as deployment pipelines and life cycle management aren't available in single-workspace patterns because they require separate workspaces."
2. **One workspace per team.** *Rejected because* a team is not an owner of data. Revenue data would end up in whichever team built it first, reports and data would share one set of roles, and a reorganisation would change the structure.
3. **One workspace per domain and stage, all three layers in it** (`Finance-Prod-Data`). *Rejected because* Microsoft's medallion guidance says: "While you can create all lakehouses in a single Fabric workspace, we recommend that you create each lakehouse in its own, separate workspace. This approach provides you with more control and better governance at the layer level." In one workspace, whoever may write Silver may also write Gold, and the only separation left is OneLake security roles, which workspace Contributors bypass. Still a reasonable choice for a very small team.
4. **Bronze and Silver together, Gold separate.** Microsoft's deployment-pattern page gives this as an example ("host the bronze and silver layers ... in one workspace and host the gold layer in a separate workspace"). *Rejected because* it saves one workspace per stage but mixes raw and cleaned data under one set of roles. It is the fallback if the workspace count becomes a burden.
5. **Separate tenants per business unit** (Microsoft's multi-tenant pattern). *Rejected because* there is no regulatory or sovereignty requirement for it, and it would put a tenant boundary between the Finance data and its main consumers.
6. **One workspace per layer and stage for core data domains, one per stage for consumer domains (chosen).**

## Why

- **The workspace is the unit for roles, Git and deployment.** Microsoft describes "Workspaces as boundaries for scale, governance, and security": access is granted through workspace roles, Git integration connects one workspace to one Git folder, and deployment pipelines need separate workspaces ([deployment patterns](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447)). Whatever must differ between Dev and Prod, or between Bronze and Gold, needs its own workspace.
- **Layer-level control.** The medallion recommendation quoted above. The practical effect: engineers who load Bronze don't need write access to Gold, and readers only ever see the Gold workspace ([Medallion architecture in Fabric](https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture?wt.mc_id=AZ-MVP-5003447)).
- **Stages.** Microsoft's checklist for a governed workspace includes a specific naming convention, an assigned domain, security groups for roles, separate development, test and production workspaces, and Git ([Workspaces at the tenant level](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning?wt.mc_id=AZ-MVP-5003447)).
- **Core and consumer domains.** Microsoft's domain-oriented pattern: "Have consumer domains use shortcuts to reference producer data products rather than copying them" ([OneLake patterns](https://learn.microsoft.com/fabric/onelake/architecture-patterns?wt.mc_id=AZ-MVP-5003447)). Microsoft says producer; I say core data domain.
- **Layers in separate workspaces still work together.** Materialized lake views can reference tables in other lakehouses: "These lakehouses can be in the same workspace or different workspaces" ([Manage materialized lake views lineage](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/view-lineage?wt.mc_id=AZ-MVP-5003447)). For Git, Microsoft shows a multi-workspace solution with one Git folder per workspace on the same branch ([Fabric CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)).
- **Folders in consumer workspaces organise, they don't secure.** "Currently, folders inherit the permissions of the workspace where they're located" ([Create folders in workspaces](https://learn.microsoft.com/fabric/fundamentals/workspaces-folders?wt.mc_id=AZ-MVP-5003447)). That is fine for a consumer workspace with one audience per stage.

What is my choice and not Microsoft's: the exact cut (layers only in core data domains, one reports workspace per consumer domain and stage) and the names. Microsoft recommends separate stage workspaces, domains and the layer split; it doesn't prescribe this combination.

## Consequences and trade-offs

- **More workspaces.** Nine per core data domain plus three per consumer domain. Creation must be scripted; Microsoft's CI/CD guidance says "Use Terraform to create the workspace or workspaces and a separate capacity that each environment requires" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). This example scripts the workspaces but shares one non-production capacity between Dev and Test; the reason is in [ADR-0003](0003-capacity-by-workload.md). A workspace request process is needed; Microsoft names "2 to 4 hours" as an ideal turnaround ([Workspaces at the tenant level](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning?wt.mc_id=AZ-MVP-5003447)).
- **More groups, unless they are shared.** Microsoft warns that groups mapped to workspace roles "can quickly become unmanageable" and that splitting data and reporting workspaces, and Dev, Test and Prod, multiplies them. It also says "often it's possible to manage a collection of workspaces with one set of groups" ([Workspaces at the workspace level](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-workspace-level-planning?wt.mc_id=AZ-MVP-5003447)). We use one set of groups per domain and stage across the three layer workspaces; only Gold gets a readers group ([ADR-0005](0005-permissions-onelake-security.md)).
- **Cross-workspace reads need access upstream.** The identity that loads Silver needs read access to Bronze. Viewing extended lineage of materialized lake views needs read access on the upstream workspaces, and scheduling upstream views needs Contributor on the upstream lakehouse ([MLV lineage](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/view-lineage?wt.mc_id=AZ-MVP-5003447)).
- **The release deploys per workspace.** One Git folder per layer workspace, one deployment per target workspace ([ADR-0004](0004-release-path-fabric-cicd.md)).
- **Domains don't grant access.** "Domain assignment doesn't affect item visibility or accessibility" ([Fabric domains](https://learn.microsoft.com/fabric/governance/domains?wt.mc_id=AZ-MVP-5003447)). Assigning a workspace to the Finance domain organises the catalog; access still comes from workspace roles and OneLake roles.
- **Open point: do folders survive Git and the release?** Microsoft Learn contradicts itself. The folders page says "Git doesn't currently support workspace folders" ([Folders in workspaces](https://learn.microsoft.com/fabric/fundamentals/workspaces-folders?wt.mc_id=AZ-MVP-5003447)); the Git integration overview says "The workspace structure, including subfolders, is preserved in the Git repository" ([What is Fabric Git integration?](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration?wt.mc_id=AZ-MVP-5003447)). The fabric-cicd documentation doesn't mention workspace folders. Until a test in Dev and Test shows the folders arriving in Prod, treat the team folders in consumer workspaces as convenience, and don't let anything depend on them.
- **Pick the region first.** Moving a workspace to a capacity in another region means re-creating most items; Microsoft advises to "prefer same-region migration" ([deployment patterns](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447)).

## Sources

Checked on Microsoft Learn on 2026-09-30.

- [Choose a Microsoft Fabric deployment pattern](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447)
- [Understand medallion architecture for Fabric with OneLake](https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture?wt.mc_id=AZ-MVP-5003447)
- [OneLake patterns and foundational capabilities](https://learn.microsoft.com/fabric/onelake/architecture-patterns?wt.mc_id=AZ-MVP-5003447)
- [Power BI implementation planning: Workspaces at the tenant level](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning?wt.mc_id=AZ-MVP-5003447)
- [Power BI implementation planning: Workspaces at the workspace level](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-workspace-level-planning?wt.mc_id=AZ-MVP-5003447)
- [Fabric domains](https://learn.microsoft.com/fabric/governance/domains?wt.mc_id=AZ-MVP-5003447)
- [Create folders in workspaces](https://learn.microsoft.com/fabric/fundamentals/workspaces-folders?wt.mc_id=AZ-MVP-5003447)
- [Manage Fabric materialized lake views lineage](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/view-lineage?wt.mc_id=AZ-MVP-5003447)
- [Fabric CI/CD concepts and best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)
