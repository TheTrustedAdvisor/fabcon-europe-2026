# Architecture Before Technology: the checklist

Matthias Falland · FabCon Europe 2026 · Community Hub · linkedin.com/in/matthias-falland

**Decide the structure first; then the technology questions answer themselves.** Answer these five questions before the first item is created, in this order: each answer makes the next one easier. Sources: Microsoft Learn, checked on 2026-09-29.

| # | Decide first | Architecture answer | Why it works in Fabric | Month-six sentence it prevents |
|---|---|---|---|---|
| 1 | Where does it live? | One workspace per domain and stage; core data domains (Finance data) and consumer domains (Sales reports) | The workspace is the unit for roles, Git and deployment pipelines; domains group a team's workspaces for the OneLake catalog | "Where is the current version?" |
| 2 | How do others read it? | One path Bronze, Silver, Gold in the core data domain; consumers get a shortcut to Gold, never a copy | A shortcut points to the data instead of copying it; changes are visible at once, and the reader's capacity pays for its reads | "Which revenue number is right?" |
| 3 | What competes for compute? | Separate capacity by workload: scheduled jobs, reports, Dev and Test | Throttling is per capacity: when one is overloaded, the others aren't slowed by its load. On an overloaded capacity, the reports people open are delayed, then rejected first | "Why is it slow on Monday?" |
| 4 | How does it reach Prod? | Git, then fabric-cicd to Test and Prod; nobody edits Prod | Git integration works per workspace. Data never travels, connections do: fabric-cicd swaps IDs per stage with a parameter file, schedules live in the item definition, post-deploy actions run the refresh. Microsoft recommends fabric-cicd as a best practice | "Who changed Prod?" |
| 5 | Who says yes to access? | Owner per domain, roles in OneLake: the owner approves groups; readers get Viewer plus a OneLake role on Gold (tables, rows, columns) | Access comes from workspace roles, not the domain; OneLake rules are authored once and enforced across Fabric engines | "Who can see this report?" |

**Also:** name workspaces `{Domain}-{Env}-{Purpose}` (Finance-Prod-Data, Sales-Dev-Reports, Finance-Prod-Reports), agreed before the first workspace. Write an ADR per architecture decision (decision, why, alternatives, date, owner, status), superseded, never edited, versioned in Git.

## Then the technology question: Lakehouse or Warehouse?

**My recommendation: Lakehouse, Lakehouse, Lakehouse** for Bronze, Silver and Gold.

- **Change data feed:** switch it on for the Delta tables in a lakehouse, and the next layer can read only the rows that changed. Materialized lake views use the same change feed for incremental refresh. Set `delta.enableChangeDataFeed = true` when you create the table; it doesn't backfill.
- **Shortcuts:** they land in a lakehouse, so every layer shares without copies.
- **One permission model:** the OneLake roles from answer 5 cover every layer. A Gold Warehouse adds a second model: its T-SQL security applies to SQL queries only.
- **Take a Warehouse** where Gold needs T-SQL writes, stored procedures or multi-table transactions. Same Delta in OneLake, same SQL engine: add it later without moving data.

**On Monday:** ask the five questions about your own platform; the first one without a clear answer is where to start.

## Links

- Everything from the session: https://github.com/TheTrustedAdvisor/fabcon-europe-2026

- Fabric deployment patterns: https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447
- fabric-cicd and CI/CD best practices: https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447
- OneLake security, row and column level: https://learn.microsoft.com/fabric/onelake/security/table-column-row-security?wt.mc_id=AZ-MVP-5003447
- Change data feed with Delta tables: https://learn.microsoft.com/fabric/data-engineering/delta-lake-change-data-feed?wt.mc_id=AZ-MVP-5003447
- Send me your month-six sentence: https://www.linkedin.com/in/matthias-falland/
