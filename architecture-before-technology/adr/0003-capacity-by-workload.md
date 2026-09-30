# ADR-0003: Capacity split by workload

- **Status:** Accepted (example)
- **Status history:** 2026-10-01 Proposed; 2026-10-01 Accepted (example)
- **Date:** 2026-10-01
- **Owner:** Platform architect; one named capacity admin per capacity
- **Confidence:** medium. The split follows Microsoft's guidance; the sizes are estimates until a month of Capacity Metrics data exists.
- **Revisit when:** the Throttling chart of any production capacity shows interactive delay in two consecutive weeks; one domain uses more than half of a shared capacity; or capacity overage charges exceed the agreed threshold twice in a quarter
- **Related:** [ADR-0001](0001-workspace-cut.md) (the workspaces assigned here), [ADR-0004](0004-release-path-fabric-cicd.md) (capacities created with the environments)

## Context

The month-six sentence is "Why is it slow on Monday?". Scheduled loads for Bronze, Silver and Gold, the reports people open in the morning, and development experiments all run on one capacity. Fabric smooths background jobs over 24 hours, so a heavy night of loads is still being paid off when people open reports at nine. When the capacity is overloaded, Fabric delays interactive requests first, then rejects them, and only later rejects background work ([Fabric throttling policy](https://learn.microsoft.com/fabric/enterprise/throttling?wt.mc_id=AZ-MVP-5003447)). The people who notice are report readers, not the engineers whose jobs caused the load.

## Decision

Three capacities, split by workload and stage, all in the same region:

| Capacity | Workspaces | Load |
|---|---|---|
| `fcprodjobseu01` | Finance-Prod-Bronze, -Silver, -Gold (every core data domain in Prod) | Scheduled loads, background |
| `fcprodreportseu01` | Sales-Prod-Reports, Finance-Prod-Reports (every consumer domain in Prod) | Reports and semantic models people open, interactive |
| `fcnonprodeu01` | Every Dev and Test workspace | Development, tests; paused outside working hours |

Consumers read Gold through shortcuts that live in their own workspace, so their reads run on the reports capacity. Capacity overage is decided per capacity, not left at its default. Capacity-level surge protection is on for the reports capacity.

## Options considered

1. **One capacity for everything.** *Rejected because* scheduled jobs and interactive reports compete for the same capacity units. Microsoft: "Heavy background processing (for example, Spark ETL jobs or AI training) should use different capacities than interactive report queries, since overlapping can cause performance issues" ([Capacity planning part 3](https://learn.microsoft.com/fabric/enterprise/capacity-planning-enterprise-managed-self-service-solutions?wt.mc_id=AZ-MVP-5003447)). For a small team with light loads, one capacity plus surge protection is a fair start.
2. **One capacity per domain.** Microsoft lists "Dedicated capacity per department/domain" as a pattern ([Capacity planning part 2](https://learn.microsoft.com/fabric/enterprise/capacity-planning-scale-self-service-analytics?wt.mc_id=AZ-MVP-5003447)). *Rejected for now because* the Monday conflict is inside Finance: Finance's own loads slow Finance's own reports. It becomes a second axis when one domain dominates a capacity.
3. **One capacity, protected by workspace-level surge protection.** *Rejected because* workspace-level surge protection is in preview, and Microsoft itself says: "To fully protect critical solutions, isolate them in a designated capacity" ([Surge protection](https://learn.microsoft.com/fabric/enterprise/surge-protection?wt.mc_id=AZ-MVP-5003447)).
4. **Autoscale Billing for Spark instead of a jobs capacity.** With it, "Spark jobs no longer consume CU from the Fabric capacity and instead use serverless resources", with a max CU limit ([Apache Spark billing](https://learn.microsoft.com/fabric/data-engineering/billing-capacity-management-for-spark?wt.mc_id=AZ-MVP-5003447)). *Not chosen as the answer because* it moves Spark jobs only; pipelines, SQL and semantic model work stay on the capacity. Worth evaluating as a complement for the jobs capacity.
5. **Split by workload and stage (chosen).**

## Why

- **Throttling is per capacity.** "Fabric applies throttling at the capacity level ... other capacities might continue running normally" ([Throttling](https://learn.microsoft.com/fabric/enterprise/throttling?wt.mc_id=AZ-MVP-5003447)).
- **Shortcut reads follow the consumer.** "When one capacity produces features such as OneLake items and another capacity consumes them, the throttling state of the consuming capacity determines whether it throttles calls to the item" ([Throttling](https://learn.microsoft.com/fabric/enterprise/throttling?wt.mc_id=AZ-MVP-5003447)). And "the transaction usage counts against the capacity tied to the workspace where the shortcut is created" ([OneLake consumption](https://learn.microsoft.com/fabric/onelake/onelake-consumption?wt.mc_id=AZ-MVP-5003447)). A throttled jobs capacity therefore doesn't throttle reports that read Gold through a shortcut in a reports workspace.
- **Dev and Test off the production capacities.** Microsoft: "Central IT should have Dev and QA workspaces on a nonproduction capacity categorized as noncritical" ([Capacity planning part 3](https://learn.microsoft.com/fabric/enterprise/capacity-planning-enterprise-managed-self-service-solutions?wt.mc_id=AZ-MVP-5003447)); the deployment-pattern page describes a "shared capacity for dev/test with a separate production capacity" and pausing dev and test outside working hours ([Deployment patterns](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447)).
- **Two smaller beat one larger when the demands differ.** Microsoft's example: "run tier 1 on an F64 and tier 2 on another F64 rather than both on a single F128" ([Capacity planning part 3](https://learn.microsoft.com/fabric/enterprise/capacity-planning-enterprise-managed-self-service-solutions?wt.mc_id=AZ-MVP-5003447)).

## Consequences and trade-offs

- **Dev and Test share a capacity, against Microsoft's CI/CD recommendation.** Microsoft: "While you can assign workspaces from all environments to a single shared capacity, the recommended practice is to isolate environments by creating a separate Fabric capacity for each one" ([CI/CD best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)). This example deviates for cost, and because Test here checks correctness and releases, not performance: no load tests run on Test. If Test is used for performance tests, or Dev experiments disturb Test runs, give Test its own capacity.
- **Headroom is no longer shared.** Idle units on the jobs capacity can't help the reports capacity. Size each from its own peak; Microsoft's sizing unit is the 30-second window, and an F64 provides 1,920 CU seconds per 30 seconds ([Capacity planning part 3](https://learn.microsoft.com/fabric/enterprise/capacity-planning-enterprise-managed-self-service-solutions?wt.mc_id=AZ-MVP-5003447)). Avoid frequent resizing.
- **Capacity overage is on by default.** For new capacities it is enabled by default with a default threshold of 25% ([Enable capacity overage](https://learn.microsoft.com/fabric/enterprise/enable-capacity-overage?wt.mc_id=AZ-MVP-5003447)). Overage is billed at three times the pay-as-you-go rate, is "not a hard spending cap", and gives "No performance boost" ([Capacity overage](https://learn.microsoft.com/fabric/enterprise/capacity-overage-overview?wt.mc_id=AZ-MVP-5003447)). Here: on for the two production capacities with a threshold the budget owner signs; off for non-production. Email notifications for overage are in preview.
- **Surge protection covers background work only.** Capacity-level surge protection rejects new background work above a threshold so that interactive work keeps room; it doesn't stop jobs already running ([Surge protection](https://learn.microsoft.com/fabric/enterprise/surge-protection?wt.mc_id=AZ-MVP-5003447)). On the reports capacity that protects report users from semantic model refreshes.
- **Cost per domain needs the Chargeback app.** With capacities split by workload, a domain's cost is spread over three capacities. The Chargeback app breaks usage down "across workspaces, items, and domains", and usage of workspaces without a domain appears under "No domain" ([Chargeback app](https://learn.microsoft.com/fabric/enterprise/chargeback-app?wt.mc_id=AZ-MVP-5003447)). Hard cost separation per domain needs capacities per domain (option 2).
- **A paused capacity stops everything on it.** While `fcnonprodeu01` is paused, its Dev and Test workspaces can't run schedules, can't receive a release and can't sync with Git. Releases to Test and test loads run inside the working window, or the release pipeline resumes the capacity first and pauses it afterwards.
- **Direct reads on Gold run on the jobs capacity.** Only reads through a shortcut in a reports workspace move to the reports capacity. An analyst who queries the SQL analytics endpoint of Finance-Prod-Gold directly, or a semantic model that lives in the Gold workspace, uses `fcprodjobseu01`, and interactive work there competes with the loads. Two choices: analysts query through a consumer workspace with shortcuts (my recommendation; Finance's analysts get their own consumer workspace like everyone else), or accept direct reads on Gold and watch the jobs capacity for interactive delay.
- **Names are forever.** "You can't change the Fabric capacity name once it's created" ([Capacity planning part 3](https://learn.microsoft.com/fabric/enterprise/capacity-planning-enterprise-managed-self-service-solutions?wt.mc_id=AZ-MVP-5003447)); hence no team or SKU in the name. See [naming.md](../naming.md).
- **Not every slow Monday is the capacity.** "Slow performance is often due to the design of an item. Only sometimes is slow performance due to capacity throttling" ([Throttling](https://learn.microsoft.com/fabric/enterprise/throttling?wt.mc_id=AZ-MVP-5003447)). Check the Throttling chart before buying a larger SKU.

## Sources

Checked on Microsoft Learn on 2026-09-30.

- [Understand the Fabric capacity throttling policy](https://learn.microsoft.com/fabric/enterprise/throttling?wt.mc_id=AZ-MVP-5003447)
- [Capacity planning guide part 3: Scale for centralized analytics](https://learn.microsoft.com/fabric/enterprise/capacity-planning-enterprise-managed-self-service-solutions?wt.mc_id=AZ-MVP-5003447)
- [Capacity planning guide part 2: Scale for decentralized analytics](https://learn.microsoft.com/fabric/enterprise/capacity-planning-scale-self-service-analytics?wt.mc_id=AZ-MVP-5003447)
- [Manage surge protection for Fabric capacities](https://learn.microsoft.com/fabric/enterprise/surge-protection?wt.mc_id=AZ-MVP-5003447)
- [Capacity overage in Microsoft Fabric](https://learn.microsoft.com/fabric/enterprise/capacity-overage-overview?wt.mc_id=AZ-MVP-5003447)
- [Enable capacity overage](https://learn.microsoft.com/fabric/enterprise/enable-capacity-overage?wt.mc_id=AZ-MVP-5003447)
- [OneLake compute and storage consumption](https://learn.microsoft.com/fabric/onelake/onelake-consumption?wt.mc_id=AZ-MVP-5003447)
- [Microsoft Fabric Chargeback app](https://learn.microsoft.com/fabric/enterprise/chargeback-app?wt.mc_id=AZ-MVP-5003447)
- [Apache Spark billing and utilization](https://learn.microsoft.com/fabric/data-engineering/billing-capacity-management-for-spark?wt.mc_id=AZ-MVP-5003447)
- [Fabric CI/CD concepts and best practices](https://learn.microsoft.com/fabric/fundamentals/understand-best-practices-fabric-cicd?wt.mc_id=AZ-MVP-5003447)
- [Choose a Microsoft Fabric deployment pattern](https://learn.microsoft.com/azure/architecture/data-guide/technology-choices/fabric-deployment-patterns?wt.mc_id=AZ-MVP-5003447)
