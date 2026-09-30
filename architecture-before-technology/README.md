# Architecture Before Technology

FabCon Europe 2026, Barcelona · Community Hub · Thursday 1 October 2026 · Matthias Falland

**Decide the structure first; then the technology questions answer themselves.**

## Abstract

Most Fabric platforms that struggle did not pick the wrong engine. They skipped a few structural decisions on day 1. The talk tells the story of a typical platform: five reasonable decisions that become five familiar sentences in month six, from "Which revenue number is right?" to "Why is it slow on Monday?". For each one there is an architecture answer and one Fabric reason why it works. Then the technology question is easy: why I build Bronze, Silver and Gold as lakehouses, and when a Warehouse still wins.

## The five answers

| # | Month-six sentence | Architecture answer |
|---|---|---|
| 1 | "Where is the current version?" | One workspace per domain and stage, named `{Domain}-{Env}-{Purpose}` (see [naming.md](naming.md)) |
| 2 | "Which revenue number is right?" | Core data domains produce Gold; consumer domains read it by shortcut, never a copy |
| 3 | "Why is it slow on Monday?" | Separate capacity by workload: scheduled jobs, reports, Dev and Test |
| 4 | "Who changed Prod?" | Build in Dev with Git; release to Test and Prod with fabric-cicd and a parameter file per stage |
| 5 | "Who can see this report?" | One owner per domain; groups, not people, get workspace roles |

Then the technology: Lakehouse, Lakehouse, Lakehouse (change data feed, shortcuts, one OneLake security model), and a Warehouse where Gold needs T-SQL writes or multi-table transactions.

And one more: write an architecture decision record (ADR) per decision. Template and two examples in [adr/](adr/).

## Files

| File | What |
|---|---|
| [ArchitectureBeforeTechnology.pdf](ArchitectureBeforeTechnology.pdf) | The slides |
| [handout.pdf](handout.pdf) · [handout.md](handout.md) | The one-page architecture checklist |
| [adr/0000-template.md](adr/0000-template.md) | ADR template |
| [adr/0001-workspace-cut.md](adr/0001-workspace-cut.md) | Example: the workspace cut |
| [adr/0002-lakehouse-lakehouse-lakehouse.md](adr/0002-lakehouse-lakehouse-lakehouse.md) | Example: Lakehouse for Bronze, Silver and Gold |
| [naming.md](naming.md) | Naming convention quick reference |
| [links.md](links.md) | Microsoft Learn links for every answer |

Sources were checked on Microsoft Learn in September 2026. Fabric changes fast; if a linked page says something different today, the page wins.
