# ADR-0002: Lakehouse for Bronze, Silver and Gold

- **Status:** Accepted (example)
- **Status history:** 2026-10-01 Proposed; 2026-10-01 Accepted (example)
- **Date:** 2026-10-01
- **Owner:** Data platform lead
- **Confidence:** medium. The lakehouse path fits today's Gold consumers (reports and ad-hoc SQL reads); a T-SQL write requirement would change it.
- **Revisit when:** Gold needs T-SQL writes, stored procedures or multi-table transactions; or a consumer needs dynamic or multi-table row-level security
- **Related:** [ADR-0001](0001-workspace-cut.md) (one workspace per layer), [ADR-0004](0004-release-path-fabric-cicd.md) (lakehouse schema changes in the release), [ADR-0005](0005-permissions-onelake-security.md) (the permission model this decision relies on)

## Context

Each layer workspace from ADR-0001 holds one storage item: Bronze, Silver and Gold. Microsoft documents two patterns: "Create each layer as a lakehouse. In this case, business users access data by using the SQL analytics endpoint", or Bronze and Silver as lakehouses with Gold as a warehouse ([Medallion architecture in Fabric](https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture?wt.mc_id=AZ-MVP-5003447)). Gold consumers in the example organisation are semantic models, reports and analysts who query with SQL. Nobody writes to Gold with T-SQL today.

## Decision

All three layers are lakehouses: `finance_bronze` in Finance-{Env}-Bronze, `finance_silver` in Finance-{Env}-Silver, `finance_gold` in Finance-{Env}-Gold. Silver and Gold tables are Delta tables with change data feed switched on. A warehouse is added next to Gold only where Gold needs T-SQL writes, stored procedures or multi-table transactions, and it gets its own ADR.

## Options considered

1. **Lakehouse, Lakehouse, Lakehouse (chosen).**
2. **Lakehouse, Lakehouse, Warehouse.** *Rejected because* Gold would carry a second permission model (T-SQL security) that doesn't travel through shortcuts, and today's consumers don't need T-SQL writes.
3. **Warehouse for all layers.** *Rejected because* Bronze lands files and raw formats; Microsoft recommends keeping Bronze data "in its original format" and using shortcuts in Bronze instead of copies, which is lakehouse territory ([Medallion architecture](https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture?wt.mc_id=AZ-MVP-5003447)).

## Why

- **Change data feed.** On Delta tables, change data feed records inserts, updates and deletes, so the next layer can read only the rows that changed instead of reprocessing the table ([Change data feed with Delta tables](https://learn.microsoft.com/fabric/data-engineering/delta-lake-change-data-feed?wt.mc_id=AZ-MVP-5003447)). Materialized lake views use it for incremental refresh; without change data feed on all sources they fall back to a full refresh ([Optimal refresh for materialized lake views](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/refresh-materialized-lake-view?wt.mc_id=AZ-MVP-5003447)).
- **Shortcuts.** A shortcut lands in a lakehouse and makes a table in another lakehouse appear as a local table, without a copy ([Shortcuts in a lakehouse](https://learn.microsoft.com/fabric/data-engineering/lakehouse-shortcuts?wt.mc_id=AZ-MVP-5003447)). Consumer workspaces read Gold this way (ADR-0001).
- **One permission model.** OneLake security roles define table, row and column permissions on the lakehouse, enforced across Spark, the SQL analytics endpoint in user's identity mode and Direct Lake on OneLake ([Read data secured with OneLake security](https://learn.microsoft.com/fabric/onelake/security/read-secured-data?wt.mc_id=AZ-MVP-5003447)). Warehouse T-SQL row, column and object security is enforced only inside the warehouse's SQL context; through a shortcut, "users accessing the data through a shortcut might see the full warehouse data" ([OneLake security for SQL analytics endpoints](https://learn.microsoft.com/fabric/onelake/security/sql-analytics-endpoint-onelake-security?wt.mc_id=AZ-MVP-5003447)).
- **It matches Microsoft's CI/CD checklist.** "Design a medallion architecture by using lakehouse items as storage containers for bronze, silver, and gold layers" ([Fabric CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)).
- **The warehouse stays open.** Lakehouse and warehouse "store data in Delta Lake format in OneLake and share the same SQL engine" ([Decision guide: Warehouse or Lakehouse](https://learn.microsoft.com/fabric/fundamentals/decision-guide-lakehouse-warehouse?wt.mc_id=AZ-MVP-5003447)), so a warehouse can be added later without moving the lake.

## Consequences and trade-offs

- **The SQL analytics endpoint is read-only.** The decision guide lists "Full DQL, no DML, and limited DDL" for it. Anything that must write with T-SQL goes to a warehouse ([Decision guide](https://learn.microsoft.com/fabric/fundamentals/decision-guide-lakehouse-warehouse?wt.mc_id=AZ-MVP-5003447)).
- **Change data feed has to be switched on, per table.** It "doesn't backfill": it records changes from the moment it's on. It can be set when a table is created or later with ALTER TABLE; creating the table with it is the rule here. It adds files under `_change_data`, which VACUUM cleans up ([Change data feed](https://learn.microsoft.com/fabric/data-engineering/delta-lake-change-data-feed?wt.mc_id=AZ-MVP-5003447)).
- **Materialized lake views refresh incrementally only under conditions** ([Optimal refresh](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/refresh-materialized-lake-view?wt.mc_id=AZ-MVP-5003447)):
  - change data feed must be on for every source table, and the view must be written in Spark SQL; "PySpark-defined MLVs always use full refresh" (PySpark authoring is in preview, [MLV overview](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/overview-materialized-lake-view?wt.mc_id=AZ-MVP-5003447));
  - appends are processed incrementally; for updates and deletes, "incremental refresh is supported only when the materialized lake view uses a refresh hint that identifies row identity. Otherwise, the engine falls back to full refresh";
  - inner joins, `UNION ALL` and `SUM`, `MIN`, `MAX`, `COUNT` qualify; `DISTINCT`, window functions and non-deterministic functions don't, and left outer joins only if the right-hand table is unchanged in that cycle;
  - they require schema-enabled lakehouses ([Lakehouse schemas](https://learn.microsoft.com/fabric/data-engineering/lakehouse-schemas?wt.mc_id=AZ-MVP-5003447)); schemas are on by default for lakehouses created in the portal, but a lakehouse created by REST API needs them requested explicitly.

  Silver tables fed by updates (customer master data, invoice corrections) need a refresh hint, or they refresh fully every time.
- **Schema changes are code.** Lakehouse table definitions aren't part of the item definition that a release deploys; the release runs a notebook that creates or alters tables. A warehouse would use SqlPackage instead ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). See ADR-0004.
- **Row-level security costs capacity.** OneLake security row-level security is metered at 1 CU second per million rows in the table ([OneLake compute and storage consumption](https://learn.microsoft.com/fabric/onelake/onelake-consumption?wt.mc_id=AZ-MVP-5003447)).
- **Workspace writers are not restricted by OneLake roles.** Admin, Member and Contributor bypass them; the exception is row-level security on the SQL analytics endpoint in user's identity mode, where "the defined security rules are enforced for all users, including those in Admin, Member, and Contributor roles" ([OneLake security for SQL analytics endpoints](https://learn.microsoft.com/fabric/onelake/security/sql-analytics-endpoint-onelake-security?wt.mc_id=AZ-MVP-5003447)). Readers therefore get Viewer plus a OneLake role (ADR-0005).

## Sources

Checked on Microsoft Learn on 2026-09-30.

- [Understand medallion architecture for Fabric with OneLake](https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture?wt.mc_id=AZ-MVP-5003447)
- [Use change data feed with Delta tables](https://learn.microsoft.com/fabric/data-engineering/delta-lake-change-data-feed?wt.mc_id=AZ-MVP-5003447)
- [Optimal refresh for materialized lake views](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/refresh-materialized-lake-view?wt.mc_id=AZ-MVP-5003447)
- [What are materialized lake views?](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/overview-materialized-lake-view?wt.mc_id=AZ-MVP-5003447)
- [What are lakehouse schemas?](https://learn.microsoft.com/fabric/data-engineering/lakehouse-schemas?wt.mc_id=AZ-MVP-5003447)
- [Shortcuts in a lakehouse](https://learn.microsoft.com/fabric/data-engineering/lakehouse-shortcuts?wt.mc_id=AZ-MVP-5003447)
- [Decision guide: Warehouse or Lakehouse](https://learn.microsoft.com/fabric/fundamentals/decision-guide-lakehouse-warehouse?wt.mc_id=AZ-MVP-5003447)
- [Read data secured with OneLake security](https://learn.microsoft.com/fabric/onelake/security/read-secured-data?wt.mc_id=AZ-MVP-5003447)
- [OneLake security for SQL analytics endpoints](https://learn.microsoft.com/fabric/onelake/security/sql-analytics-endpoint-onelake-security?wt.mc_id=AZ-MVP-5003447)
- [OneLake compute and storage consumption](https://learn.microsoft.com/fabric/onelake/onelake-consumption?wt.mc_id=AZ-MVP-5003447)
- [Fabric CI/CD concepts and best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)
