# ADR-0004: Release path: Git, then fabric-cicd to Test and Prod

- **Status:** Accepted (example)
- **Status history:** 2026-10-01 Proposed; 2026-10-01 Accepted (example)
- **Date:** 2026-10-01
- **Owner:** Data platform lead
- **Confidence:** medium-high. The path follows Microsoft's CI/CD guidance; some item types it moves (reports, semantic models) are still in preview in Git integration.
- **Revisit when:** an item type we need has no source control or public create/update API; or a team without Python skills has to run its own releases
- **Related:** [ADR-0001](0001-workspace-cut.md) (one workspace per stage and layer), [ADR-0002](0002-lakehouse-lakehouse-lakehouse.md) (lakehouse schema changes), [ADR-0003](0003-capacity-by-workload.md) (capacities per stage), [ADR-0005](0005-permissions-onelake-security.md) (who may write where)

## Context

The month-six sentence is "Who changed Prod?". Somebody fixed a notebook directly in production, the schedule was changed by hand, and the semantic model in Prod still points at the Test lakehouse. Each stage needs its own connections, its own schedules and its own data, while the logic must be the same. The example organisation has one Git repository per domain and a CI/CD tool (Azure DevOps or GitHub; the decision doesn't depend on which).

## Decision

1. **Dev is connected to Git; work happens in branched-out feature workspaces.** Each Dev workspace (Finance-Dev-Bronze, -Silver, -Gold, Sales-Dev-Reports) is connected to its own folder on the integration branch `main` of its domain repository. Nobody commits to `main` from the Dev workspace: every change starts as a feature branch in a feature workspace created with Fabric's branch-out, on the non-production capacity, and reaches `main` by pull request. The Dev workspace only follows `main`.
2. **Test and Prod are not edited by hand.** A release pipeline runs fabric-cicd with the environment `test`, then, after an approval, `prod`. It deploys each Git folder to its matching workspace, under a service principal.
3. **Stage differences live in configuration.** A variable library with value sets named `dev`, `test` and `prod` is the first choice; a parameter file covers what the variable library can't. Item names are identical in every stage, so a connection that accepts a lakehouse name needs no parameter. References that cross workspace boundaries (a Silver notebook reading Bronze, a shortcut in Sales pointing to Finance Gold) do need one: an item reference variable in the variable library, or a `find_replace` rule in the parameter file.
4. **A post-deploy step runs after every release:** schema notebook for the lakehouses, then the top-level load, then the refresh of import-mode semantic models.

## Options considered

Microsoft compares three release options ([Fabric CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)):

1. **Deployment pipelines.** Low code, built into Fabric. *Rejected because* the page positions them for the same tenant and not for hundreds of items, and the release logic (approvals, post-deploy work) would sit outside the pipeline anyway. A good choice for a small team that releases a handful of reports.
2. **Git sync per stage** (Test and Prod workspaces each connected to their own branch). *Rejected because* every stage then needs its own post-sync work, and promotion becomes a merge between long-lived branches.
3. **API-driven deployment with fabric-cicd (chosen).** The most flexible option in Microsoft's comparison.
4. **Own scripts against the Fabric REST APIs.** *Rejected because* it rebuilds what fabric-cicd already does (dependency order, ID replacement, orphan cleanup).

## Why

- **Microsoft recommends it.** "While fabric-cicd is an open-source project, the Fabric product team officially supports and recommends it as a best practice" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)).
- **Git gives history and a way back.** Git integration works per workspace; Microsoft's multi-workspace example connects each workspace to its own Git folder on the same branch ([Git integration](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration?wt.mc_id=AZ-MVP-5003447), [CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). The integration branch should accept pull requests only.
- **What fabric-cicd does for each stage** ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)):
  - deploys in dependency order (variable library first, then lakehouses, notebooks, pipelines) and re-binds references between items it deploys together;
  - replaces workspace, lakehouse and connection IDs from a parameter file (`parameter.yml`) where references aren't by name;
  - deploys schedules, because they live in the item definition (`.schedules`);
  - runs post-deploy actions against Fabric REST APIs; Microsoft's examples are activating a variable library value set and creating shortcuts;
  - removes items that Git no longer contains (orphan control), so Prod matches the repository.
- **Names instead of IDs, where a name is accepted.** "The lakehouse name stays the same across all environments, eliminating the need for parameterization" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). That is why [naming.md](../naming.md) forbids stage suffixes in item names. It covers connections that accept a lakehouse name, not every reference.
- **Cross-workspace references need a variable.** "Fabric only supports auto-binding behavior between items that exist in the same workspace. When designing multi-workspace solutions, use item reference variables in a variable library to manage item dependencies that span workspace boundaries" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). With one workspace per layer (ADR-0001), this is the normal case, not the exception.
- **Feature workspaces are Microsoft's pattern.** "Use the *branched workspaces* capabilities of Fabric to create and manage feature branches and feature workspaces" and "Delete feature branches after pull requests to keep them short-lived" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)).
- **Variable libraries first.** Deployment rules still exist, but "variable libraries should usually be your first choice for parameterization". The environment name used by fabric-cicd has to match the value set name, which is why the value sets are called `test` and `prod` ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)).

## Consequences and trade-offs

- **Data never travels.** "Each option creates the lakehouse as an empty data container that requires additional work" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). Table creation and changes run as a notebook in the post-deploy step; the first load fills the tables.
- **Refresh is a post-deploy step, not a fabric-cicd feature.** Microsoft recommends a job after deployment that runs the top-level pipeline or notebook, and: "If your Fabric solution includes a semantic model that uses import mode, you should also automate the process of running a refresh operation" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)).
- **Role memberships don't travel.** Entra users and groups as members of OneLake security roles are, in Microsoft's words, "not supported in lakehouse item definition APIs, and lakehouse CICD and will be ignored if specified"; the data access roles part of the lakehouse definition is disabled by default ([Lakehouse definition](https://learn.microsoft.com/rest/api/fabric/articles/item-management/definitions/lakehouse-definition?wt.mc_id=AZ-MVP-5003447)). Group assignments per stage are a separate, scripted step (ADR-0005).
- **Full deployment every time.** fabric-cicd deploys the whole folder, not the diff of the last commit, and supports only item types that have source control and public create and update APIs ([fabric-cicd documentation](https://microsoft.github.io/fabric-cicd/latest/)). Check the supported list before choosing a new item type.
- **Some items are preview in Git.** Git integration lists reports and semantic models as preview ([Git integration](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration?wt.mc_id=AZ-MVP-5003447)). Include them, and know that behaviour can change.
- **A service principal owns the release; humans don't write in Test and Prod.** The release service principal is at least Contributor on each target workspace, as Microsoft's fabric-cicd walkthrough asks for "at least the Contributor role on target Fabric workspaces", and service principals must be allowed to use Fabric APIs in the tenant settings ([Deploy PBIP using fabric-cicd](https://learn.microsoft.com/power-bi/developer/projects/projects-deploy-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). Where the release also manages OneLake security roles or switches the SQL analytics endpoint mode, it needs Member (or a separate security service principal does that part, ADR-0005). The service principal that creates shortcuts in a consumer workspace needs Read on the Gold target ([OneLake shortcut security](https://learn.microsoft.com/fabric/onelake/onelake-shortcut-security?wt.mc_id=AZ-MVP-5003447)). No secrets in the repository; they live in the CI/CD tool's secret store. No builder or Contributor role for humans in Test and Prod. The admins group (two named people, ideally just-in-time via PIM) is break-glass only. Every human change shows up in the audit log, which Fabric writes to the Microsoft Purview audit log ([Track user activities in Microsoft Fabric](https://learn.microsoft.com/fabric/admin/track-user-activities?wt.mc_id=AZ-MVP-5003447)). A direct edit in Prod is therefore an exception you can see.
- **A change across layers means one feature workspace per layer.** A new Gold column that needs a Silver change touches two workspaces, so the developer branches out two feature workspaces and opens one pull request. That is the price of the layer split in ADR-0001, paid in developer time and non-production capacity.
- **Workspace folders in the release are an open point.** See ADR-0001: Microsoft Learn is contradictory on folders in Git, and the fabric-cicd documentation says nothing about workspace folders. Test it before relying on it.
- **The team needs Python and a CI/CD tool.** Deployment pipelines need neither. This is the main cost of the decision.

## Sources

Checked on Microsoft Learn on 2026-09-30; fabric-cicd documentation on GitHub on the same day.

- [Fabric CI/CD concepts and best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)
- [What is Microsoft Fabric Git integration?](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration?wt.mc_id=AZ-MVP-5003447)
- [Lakehouse definition](https://learn.microsoft.com/rest/api/fabric/articles/item-management/definitions/lakehouse-definition?wt.mc_id=AZ-MVP-5003447)
- [Track user activities in Microsoft Fabric](https://learn.microsoft.com/fabric/admin/track-user-activities?wt.mc_id=AZ-MVP-5003447)
- [OneLake shortcut security](https://learn.microsoft.com/fabric/onelake/onelake-shortcut-security?wt.mc_id=AZ-MVP-5003447)
- [Deploy Power BI projects (PBIP) using fabric-cicd](https://learn.microsoft.com/power-bi/developer/projects/projects-deploy-fabric-cicd?wt.mc_id=AZ-MVP-5003447)
- [fabric-cicd documentation](https://microsoft.github.io/fabric-cicd/latest/)
