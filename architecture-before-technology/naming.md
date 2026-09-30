# Naming convention: quick reference

**Pattern:** `{Domain}-{Env}-{Purpose}`

| Part | Values | Why |
|---|---|---|
| Domain | Business area: Finance, Sales, Customer, ... | Tells you who owns it; lets you assign the workspace to a Fabric domain by name |
| Env | Dev, Test, Prod | Tells you whether you are in production |
| Purpose | Data (core data domain: Bronze, Silver, Gold), Reports (consumer domain) | Tells you what is inside |

## Examples

| Workspace | Kind |
|---|---|
| Finance-Dev-Data | Core data domain, development |
| Finance-Prod-Data | Core data domain, production (Gold lives here) |
| Finance-Prod-Reports | Consumer domain: Finance's own reports, reading Gold by shortcut |
| Sales-Dev-Reports | Consumer domain, development |
| Sales-Prod-Reports | Consumer domain, production |

## Rules

1. Agree on the pattern before the first workspace exists; renaming later leaves bookmarks, scripts and habits behind.
2. One pattern for everything. Microsoft suggests leaving production without a suffix ([Workspace naming conventions](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning?wt.mc_id=AZ-MVP-5003447)); that works too, as long as it is the only pattern.
3. Record the pattern in an ADR (see [adr/](adr/)).
4. Consistent names make domain assignment easy: Microsoft lists "by workspace name" as a way to assign workspaces to domains ([Best practices for domains](https://learn.microsoft.com/fabric/governance/domains-best-practices?wt.mc_id=AZ-MVP-5003447)).
