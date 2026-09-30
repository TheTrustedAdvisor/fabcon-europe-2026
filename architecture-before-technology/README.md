# Architecture Before Technology

FabCon Europe 2026, Barcelona · Community Hub · Thursday 1 October 2026 · Matthias Falland

**Decide the structure first; then the technology questions answer themselves.**

## Abstract

Most Fabric platforms that struggle did not pick the wrong engine. They skipped a few structural decisions on day 1. The talk tells the story of a typical platform: five reasonable decisions that become five familiar sentences in month six, from "Which revenue number is right?" to "Why is it slow on Monday?". For each one there is an architecture answer and one Fabric reason why it works. Then the technology question is easy: why I build Bronze, Silver and Gold as lakehouses, and when a Warehouse still wins.

## The five answers

The example organisation: Finance owns the data (a core data domain), Sales builds reports on it (a consumer domain).

| # | Month-six sentence | Architecture answer | Details |
|---|---|---|---|
| 1 | "Where is the current version?" | Workspace per domain and stage. Core data domains split further by layer: Finance-Prod-Bronze, -Silver, -Gold. Consumer domains get one workspace per stage with folders per team: Sales-Prod-Reports. Names follow `{Domain}-{Env}-{Purpose}` | [ADR-0001](adr/0001-workspace-cut.md), [naming.md](naming.md) |
| 2 | "Which revenue number is right?" | The core data domain runs Bronze, Silver and Gold; every consumer reads Gold through a shortcut, never a copy | [ADR-0001](adr/0001-workspace-cut.md) |
| 3 | "Why is it slow on Monday?" | Capacity per workload: scheduled jobs, reports, and Dev and Test on separate capacities | [ADR-0003](adr/0003-capacity-by-workload.md) |
| 4 | "Who changed Prod?" | Git, then fabric-cicd to Test and Prod under a service principal; item names identical in every stage; a post-deploy step runs the load and the refresh; nobody edits Prod | [ADR-0004](adr/0004-release-path-fabric-cicd.md) |
| 5 | "Who can see this report?" | Owner per domain, roles in OneLake: the owner approves groups, groups get workspace roles, readers get Viewer plus a OneLake security role on Gold with table, row and column rules | [ADR-0005](adr/0005-permissions-onelake-security.md) |

Then the technology: Lakehouse, Lakehouse, Lakehouse (change data feed, shortcuts, and the one permission model from answer 5), and a Warehouse where Gold needs T-SQL writes or multi-table transactions ([ADR-0002](adr/0002-lakehouse-lakehouse-lakehouse.md)).

And one more: write an architecture decision record (ADR) per decision.

## Files

| File | What |
|---|---|
| [ArchitectureBeforeTechnology.pdf](ArchitectureBeforeTechnology.pdf) | The slides |
| [handout.pdf](handout.pdf) · [handout.md](handout.md) | The cheatsheet: per answer the decision, the Fabric reason, the typical pitfall, a Monday check and links |
| [adr/0000-template.md](adr/0000-template.md) | ADR template, following Microsoft Well-Architected |
| [adr/0001-workspace-cut.md](adr/0001-workspace-cut.md) | Example: workspace cut by domain, layer and stage |
| [adr/0002-lakehouse-lakehouse-lakehouse.md](adr/0002-lakehouse-lakehouse-lakehouse.md) | Example: Lakehouse for Bronze, Silver and Gold |
| [adr/0003-capacity-by-workload.md](adr/0003-capacity-by-workload.md) | Example: capacity split by workload |
| [adr/0004-release-path-fabric-cicd.md](adr/0004-release-path-fabric-cicd.md) | Example: release path with Git and fabric-cicd |
| [adr/0005-permissions-onelake-security.md](adr/0005-permissions-onelake-security.md) | Example: permission model with OneLake security |
| [naming.md](naming.md) | Naming reference: capacities, domains, workspaces, folders, items, tables, groups, Git, release environments |
| [links.md](links.md) | Microsoft Learn links for every answer |

The ADRs describe a generic example organisation, not a client. Their status "Accepted (example)" means: accepted for the example; your context decides for yours.

Sources were checked on Microsoft Learn on 2026-09-30. Fabric changes fast; if a linked page says something different today, the page wins.
