# ADR-0002: Lakehouse for Bronze, Silver and Gold

- **Status:** Accepted (example)
- **Date:** 2026-10-01
- **Owner:** Data platform lead
- **Confidence:** medium (review when Gold needs T-SQL writes)

## Context

The medallion layers Bronze, Silver and Gold need a storage item each. Microsoft documents two patterns: all three layers as lakehouses, or Bronze and Silver as lakehouses with Gold as a warehouse ([Medallion architecture in Fabric](https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture?wt.mc_id=AZ-MVP-5003447)).

## Options considered

1. Lakehouse, Lakehouse, Lakehouse.
2. Lakehouse, Lakehouse, Warehouse.

## Decision

Option 1: all three layers as lakehouses. A Gold warehouse is added only where Gold needs T-SQL writes, stored procedures or multi-table transactions.

## Why

- **Change data feed.** On Delta tables in a lakehouse, change data feed lets the next layer read only the rows that changed instead of reprocessing the whole table ([Change data feed with Delta tables](https://learn.microsoft.com/fabric/data-engineering/delta-lake-change-data-feed?wt.mc_id=AZ-MVP-5003447)). Set `delta.enableChangeDataFeed = true` when you create the table; it doesn't backfill. Materialized lake views need it for incremental refresh ([Optimal refresh for materialized lake views](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/refresh-materialized-lake-view?wt.mc_id=AZ-MVP-5003447)).
- **Shortcuts.** Shortcuts land in a lakehouse, so every layer and every consumer shares without copies ([Shortcuts in a lakehouse](https://learn.microsoft.com/fabric/data-engineering/lakehouse-shortcuts?wt.mc_id=AZ-MVP-5003447)).
- **One security model.** OneLake security roles define table, row and column permissions once, enforced across Fabric engines ([Data security in OneLake](https://learn.microsoft.com/fabric/onelake/security/get-started-security?wt.mc_id=AZ-MVP-5003447)). Warehouse T-SQL row, column and object security is "enforced only within the SQL execution context of the warehouse", and through a shortcut users "might see the full warehouse data" ([OneLake security for SQL analytics endpoints](https://learn.microsoft.com/fabric/onelake/security/sql-analytics-endpoint-onelake-security?wt.mc_id=AZ-MVP-5003447)).

## Consequences and trade-offs

- The SQL analytics endpoint of a lakehouse is read-only; T-SQL writes need a warehouse ([Decision guide: Warehouse or Lakehouse](https://learn.microsoft.com/fabric/fundamentals/decision-guide-lakehouse-warehouse?wt.mc_id=AZ-MVP-5003447)).
- Both store Delta in OneLake and share the same SQL engine, so a warehouse can be added later without moving data.
- OneLake roles don't restrict workspace Admins, Members and Contributors, except row-level security on the SQL analytics endpoint in user identity mode, which applies to everyone ([Troubleshoot OneLake security for SQL analytics endpoints](https://learn.microsoft.com/fabric/onelake/security/troubleshoot-onelake-security-for-sql-analytics-endpoints?wt.mc_id=AZ-MVP-5003447)); readers get the Viewer role plus a OneLake role ([Table, column, and row-level security in OneLake](https://learn.microsoft.com/fabric/onelake/security/table-column-row-security?wt.mc_id=AZ-MVP-5003447)).
- To read OneLake-secured data through the SQL analytics endpoint, the endpoint runs in user identity mode ([Read data secured with OneLake security](https://learn.microsoft.com/fabric/onelake/security/read-secured-data?wt.mc_id=AZ-MVP-5003447)).

## Links

- [ADR-0001](0001-workspace-cut.md)
