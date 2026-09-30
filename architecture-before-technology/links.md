# Microsoft Learn links, per answer

Every page below was checked on 2026-09-30; all but the Direct Lake security page were read in full, and that page's quoted passages were read verbatim. Where a feature is in preview, the link says so. If a page says something different today, the page wins.

## 1. Where does it live? Workspace per domain and stage

- [Choose a Microsoft Fabric deployment pattern](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447): the four patterns, workspaces as boundaries
- [Understand medallion architecture for Fabric with OneLake](https://learn.microsoft.com/fabric/onelake/onelake-medallion-lakehouse-architecture?wt.mc_id=AZ-MVP-5003447): one workspace per layer
- [Fabric domains](https://learn.microsoft.com/fabric/governance/domains?wt.mc_id=AZ-MVP-5003447)
- [Best practices for planning and creating domains](https://learn.microsoft.com/fabric/governance/domains-best-practices?wt.mc_id=AZ-MVP-5003447)
- [Create folders in workspaces](https://learn.microsoft.com/fabric/fundamentals/workspaces-folders?wt.mc_id=AZ-MVP-5003447): folders inherit workspace permissions
- [Power BI implementation planning: Workspaces at the tenant level](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-tenant-level-planning?wt.mc_id=AZ-MVP-5003447): naming, governed workspaces, request process

### Naming (see [naming.md](naming.md))

- [CAF: Define your naming convention](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming?wt.mc_id=AZ-MVP-5003447)
- [CAF: Abbreviation recommendations for Azure resources](https://learn.microsoft.com/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations?wt.mc_id=AZ-MVP-5003447)
- [Microsoft.Fabric/capacities template reference](https://learn.microsoft.com/azure/templates/microsoft.fabric/capacities?wt.mc_id=AZ-MVP-5003447): capacity name rule
- [Manage your capacities in the admin portal](https://learn.microsoft.com/fabric/admin/capacity-settings?wt.mc_id=AZ-MVP-5003447): capacity names can't be changed
- [Create a lakehouse](https://learn.microsoft.com/fabric/data-engineering/create-lakehouse?wt.mc_id=AZ-MVP-5003447): lakehouse name rule
- [What are lakehouse schemas?](https://learn.microsoft.com/fabric/data-engineering/lakehouse-schemas?wt.mc_id=AZ-MVP-5003447)
- [Delta Lake table format interoperability](https://learn.microsoft.com/fabric/fundamentals/delta-lake-interoperability?wt.mc_id=AZ-MVP-5003447): characters to avoid in table names
- [Git integration source code format](https://learn.microsoft.com/fabric/cicd/git-integration/source-code-format?wt.mc_id=AZ-MVP-5003447): item folder names in Git

## 2. How do others read it? Shortcut to Gold, never a copy

- [OneLake patterns and foundational capabilities](https://learn.microsoft.com/fabric/onelake/architecture-patterns?wt.mc_id=AZ-MVP-5003447): domain-oriented data mesh, consumers use shortcuts
- [Shortcuts in a lakehouse](https://learn.microsoft.com/fabric/data-engineering/lakehouse-shortcuts?wt.mc_id=AZ-MVP-5003447)
- [OneLake shortcuts](https://learn.microsoft.com/fabric/onelake/onelake-shortcuts?wt.mc_id=AZ-MVP-5003447): limits and naming
- [OneLake shortcut security](https://learn.microsoft.com/fabric/onelake/onelake-shortcut-security?wt.mc_id=AZ-MVP-5003447): passthrough identity
- [OneLake compute and storage consumption](https://learn.microsoft.com/fabric/onelake/onelake-consumption?wt.mc_id=AZ-MVP-5003447): the reader's capacity pays for shortcut reads

## 3. What competes for compute? Capacity per workload

- [Understand the Fabric capacity throttling policy](https://learn.microsoft.com/fabric/enterprise/throttling?wt.mc_id=AZ-MVP-5003447)
- [Capacity planning guide part 3: Scale for centralized analytics](https://learn.microsoft.com/fabric/enterprise/capacity-planning-enterprise-managed-self-service-solutions?wt.mc_id=AZ-MVP-5003447): separate ETL from reports, Dev and QA off production, capacity names
- [Capacity planning guide part 2: Scale for decentralized analytics](https://learn.microsoft.com/fabric/enterprise/capacity-planning-scale-self-service-analytics?wt.mc_id=AZ-MVP-5003447): shared versus dedicated capacities
- [Capacity planning guide part 4: Manage growth and governance](https://learn.microsoft.com/fabric/enterprise/capacity-planning-manage-capacity-growth-governance?wt.mc_id=AZ-MVP-5003447)
- [Manage surge protection for Fabric capacities](https://learn.microsoft.com/fabric/enterprise/surge-protection?wt.mc_id=AZ-MVP-5003447): capacity level; the workspace level is preview
- [Capacity overage in Microsoft Fabric](https://learn.microsoft.com/fabric/enterprise/capacity-overage-overview?wt.mc_id=AZ-MVP-5003447) and [Enable capacity overage](https://learn.microsoft.com/fabric/enterprise/enable-capacity-overage?wt.mc_id=AZ-MVP-5003447): on by default for new capacities; email notifications preview
- [Microsoft Fabric Chargeback app](https://learn.microsoft.com/fabric/enterprise/chargeback-app?wt.mc_id=AZ-MVP-5003447)
- [Apache Spark billing and utilization](https://learn.microsoft.com/fabric/data-engineering/billing-capacity-management-for-spark?wt.mc_id=AZ-MVP-5003447): Autoscale Billing for Spark

## 4. How does it reach Prod? Git, then fabric-cicd to Prod

- [Fabric CI/CD concepts and best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447): fabric-cicd, variable libraries and item references, branched workspaces, release options, post-deploy steps
- [Track user activities in Microsoft Fabric](https://learn.microsoft.com/fabric/admin/track-user-activities?wt.mc_id=AZ-MVP-5003447): the audit log in Microsoft Purview
- [What is Microsoft Fabric Git integration?](https://learn.microsoft.com/fabric/cicd/git-integration/intro-to-git-integration?wt.mc_id=AZ-MVP-5003447): supported items; reports and semantic models are preview
- [The deployment pipelines process](https://learn.microsoft.com/fabric/cicd/deployment-pipelines/understand-the-deployment-process?wt.mc_id=AZ-MVP-5003447): what is and isn't copied, stage names are permanent
- [Lakehouse definition](https://learn.microsoft.com/rest/api/fabric/articles/item-management/definitions/lakehouse-definition?wt.mc_id=AZ-MVP-5003447): which OneLake security parts travel with the lakehouse
- [Deploy Power BI projects (PBIP) using fabric-cicd](https://learn.microsoft.com/power-bi/developer/projects/projects-deploy-fabric-cicd?wt.mc_id=AZ-MVP-5003447)
- [fabric-cicd documentation](https://microsoft.github.io/fabric-cicd/latest/) (GitHub, not Microsoft Learn)

## 5. Who says yes to access? Owner per domain, roles in OneLake

- [How OneLake security controls data access](https://learn.microsoft.com/fabric/onelake/security/data-access-control-model?wt.mc_id=AZ-MVP-5003447)
- [Best practices for OneLake security](https://learn.microsoft.com/fabric/onelake/security/best-practices-secure-data-in-onelake?wt.mc_id=AZ-MVP-5003447)
- [Data security in OneLake](https://learn.microsoft.com/fabric/onelake/security/get-started-security?wt.mc_id=AZ-MVP-5003447): workspace roles and OneLake access
- [Create and manage OneLake security roles](https://learn.microsoft.com/fabric/onelake/security/create-manage-roles?wt.mc_id=AZ-MVP-5003447): DefaultReader
- [Table, column, and row-level security in OneLake](https://learn.microsoft.com/fabric/onelake/security/table-column-row-security?wt.mc_id=AZ-MVP-5003447)
- [OneLake security for SQL analytics endpoints](https://learn.microsoft.com/fabric/onelake/security/sql-analytics-endpoint-onelake-security?wt.mc_id=AZ-MVP-5003447): user's identity mode, limitations
- [Read data secured with OneLake security](https://learn.microsoft.com/fabric/onelake/security/read-secured-data?wt.mc_id=AZ-MVP-5003447): which engines enforce the roles
- [Integrate Direct Lake security](https://learn.microsoft.com/fabric/fundamentals/direct-lake-security-integration?wt.mc_id=AZ-MVP-5003447): Direct Lake on SQL versus Direct Lake on OneLake with OneLake security
- [How Direct Lake works](https://learn.microsoft.com/fabric/fundamentals/direct-lake-how-it-works?wt.mc_id=AZ-MVP-5003447): DirectQuery fallback
- [Power BI implementation planning: Tenant-level security planning](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-security-tenant-level-planning?wt.mc_id=AZ-MVP-5003447): group types and naming
- [Power BI implementation planning: Workspaces at the workspace level](https://learn.microsoft.com/power-bi/guidance/powerbi-implementation-planning-workspaces-workspace-level-planning?wt.mc_id=AZ-MVP-5003447): groups for workspace roles

## Technology: Lakehouse all the way

- [Use change data feed with Delta tables](https://learn.microsoft.com/fabric/data-engineering/delta-lake-change-data-feed?wt.mc_id=AZ-MVP-5003447)
- [What are materialized lake views?](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/overview-materialized-lake-view?wt.mc_id=AZ-MVP-5003447): PySpark authoring is preview
- [Optimal refresh for materialized lake views](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/refresh-materialized-lake-view?wt.mc_id=AZ-MVP-5003447)
- [Manage materialized lake views lineage](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/view-lineage?wt.mc_id=AZ-MVP-5003447): views across lakehouses and workspaces
- [Spark SQL reference for materialized lake views](https://learn.microsoft.com/fabric/data-engineering/materialized-lake-views/create-materialized-lake-view?wt.mc_id=AZ-MVP-5003447)
- [Decision guide: Warehouse or Lakehouse](https://learn.microsoft.com/fabric/fundamentals/decision-guide-lakehouse-warehouse?wt.mc_id=AZ-MVP-5003447)

## ADRs

- [Maintain an architecture decision record (ADR)](https://learn.microsoft.com/azure/well-architected/architect-role/architecture-decision-record?wt.mc_id=AZ-MVP-5003447)
- [What's new in Microsoft Fabric (archive)](https://learn.microsoft.com/fabric/fundamentals/whats-new-archive?wt.mc_id=AZ-MVP-5003447): release dates cited in the ADRs
