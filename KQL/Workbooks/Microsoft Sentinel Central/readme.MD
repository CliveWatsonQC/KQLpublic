# Sentinel Central

## What this workbook is for

Sentinel Central provides one operational view across multiple Microsoft Sentinel workspaces.

It is intended to help answer:

- Which customers and workspaces are in scope?
- What incidents are being created and how are they distributed?
- Where are alert processing or ingestion delays occurring?
- Which workspaces and data sources are driving ingestion?
- Which tables are active and how are they configured for retention?
- Which analytics rules, query packs, saved searches, and Lighthouse relationships are present?
- How is the SOC performing against its SLA targets?

This is an estate-management and operational-reporting workbook. It is not a replacement for the Sentinel portal, incident investigation, billing reports, or customer-specific service-management processes.

## Important principle

Sentinel Central works across workspace estates in batches.

The workbook does not try to run one unrestricted cross-workspace query over every workspace at once. It divides the selected estate into deterministic batches and applies the active batch to the cross-workspace reports.

The production settings are:

- **Batch Size:** 30 workspaces;
- **Hard Limit:** 100 workspaces for the cross-workspace safety guard.

No workspaces should be silently dropped by batching. To cover a large estate, move through each **Analysis Batch**.

## Before using the workbook

Select the scope in this order:

1. **Region**
2. **Customer Group**
3. **Customer**
4. **Analysis Batch**
5. **Workspace**
6. **Time Range**

The other filters, such as **Product**, **Owner**, and **Resource Group**, apply where the selected report supports them.

Leave **Workspace = All** to analyse the complete active batch. Select individual workspaces when a report needs narrower investigation.

The **Help** parameter provides the workbook guidance, Change Log, configuration values, and Tenant Search.

## Workbook pages

### Incident Overview

Shows incidents across the selected workspaces. It supports filtering by severity, product, owner, and time range.

Incident reporting first reduces `SecurityIncident` to the latest state of each incident. This prevents historical versions of the same incident from inflating the results.

The main incident population is based on incidents created during the selected Time Range.

### Latency Investigation

Separates the main timing stages around an incident and its alerts:

- alert duration;
- product processing;
- Sentinel ingestion;
- incident elapsed time.

For incidents with multiple alerts, the incident-level view surfaces the worst associated alert latency. Select an incident to investigate the alert-level timestamps.

The 10-minute marker is an investigation aid. It is not an SLA threshold.

### Ingestion

Shows workspace configuration, billable ingestion, top billable data sources, and workspace/data-source contributors.

Use this page to move from:

**Where is the volume? → What is causing it? → Which workspace and table are involved?**

### Tables

Shows active tables observed in Log Analytics `Usage` during the selected period, including billable status, volume, last seen time, and activity.

### Hunting

Provides hunting views over the active workspace scope. For a large estate, change the Analysis Batch to work through the complete selected customer population.

### Query Packs

Provides discovery and execution of reusable Log Analytics queries stored in Azure.

The normal flow is:

**Select subscriptions → select a Query Pack → select a query → view the results.**

Stored Query Pack KQL may contain its own time filters, so the workbook Time Range is not guaranteed to control the query body.

### Saved Searches

Provides discovery and execution of saved Log Analytics queries within the active workspace scope.

### Retention

Combines observed Log Analytics table activity with Azure Resource Manager retention configuration.

The report starts with tables actually observed during the selected period rather than returning every possible table definition.

### Analytics Rules

Provides an inventory of Sentinel analytics rules for the selected workspace scope.

The views include rule type, status, severity, product, last modified time, schedules, MITRE mappings, entity mappings, and incident configuration where those fields are available.

### Azure Lighthouse

Shows customer and subscription context for delegated environments and identifies Sentinel-enabled workspaces visible through Azure Lighthouse.

Customer grouping uses the Azure Lighthouse directory name where available, with subscription information used as fallback context.

### SOC Reporting

Provides operational SLA reporting using a fixed rolling 90-day window.

It is intentionally separate from the general workbook Time Range. The reports include:

- SLA performance by severity;
- median and P90 first-action times;
- median and P90 resolution times;
- first-action and resolution SLA percentages;
- breach counts and time over SLA;
- incident-level breach investigation.

Median or P50 shows typical performance. P90 shows tail risk and slower cases that averages can hide.

## Customer and workspace selection

### Region

Filters the Sentinel-enabled estate by Azure region.

### Customer Group

Groups customers alphabetically using the Azure Lighthouse customer or directory name:

- A-G
- H-M
- N-S
- T-Z
- All

### Customer

Selects the subscription scopes associated with the selected customer group.

### Analysis Batch

Splits the selected workspace estate into deterministic ranges, for example:

`1–30 of 219`

`31–60 of 219`

`211–219 of 219`

Change this value to work through the complete estate.

### Workspace

Selects all workspaces in the active batch or narrows the report to individual workspaces.

The section-specific workspace selectors used by drill-downs inherit the global workspace scope.

## Incident reporting

Incident counts can differ from the number of raw `SecurityIncident` records because the workbook reduces each incident to its latest known state.

The incident reports also use incident creation time for the main reporting population. An incident modified during the selected period may not appear if it was created outside that period.

This is deliberate. It keeps the main report aligned to incidents created in the reporting period rather than mixing creation and modification populations.

## Data sources

| Source | Main use |
|--------|----------|
| `SecurityIncident` | Incident state, severity, ownership, products, status, and alert references. |
| Log Analytics `Usage` | Billable ingestion, table activity, and volume. |
| Azure Resource Graph `resources` | Workspaces, Sentinel solutions, and Azure resource inventory. |
| `ResourceContainers` | Subscription names, tenant context, and management information. |
| `ManagedServicesResources` | Azure Lighthouse registration and delegated-customer context. |
| Azure Resource Manager table configuration | Interactive and archive retention settings. |
| Analytics rule resources | Rule inventory and configuration. |
| Query Pack resources | Saved Query Pack and query definitions. |
| Saved Search resources | Saved Log Analytics searches. |

The results depend on the permissions, retention, resource visibility, and table ingestion available to the signed-in user.

## Ingestion and table reporting

The ingestion reports are intended to show where data volume is coming from, not to label high volume as automatically bad.

Use the reports together:

- **Billable Ingestion by Workspace** shows the workspaces with the largest billable volumes.
- **Top Billable Data Sources** shows the data types driving the volume.
- **Workspace & Data Source Contributors** shows the workspace and table combination behind the volume.
- **Table Usage & Activity** shows whether the table was active and when it was last seen.

Retention reporting starts from observed table activity and then enriches it with Azure configuration. A table that is not observed in the selected period may not appear even if a retention definition exists.

## Latency reporting

Latency values are calculated from the timestamps available in the incident and alert data. Missing or inconsistent timestamps can affect the result.

The report separates:

- the alert event window;
- product processing time;
- Sentinel ingestion time;
- incident elapsed time.

The incident view uses the slowest associated alert where an incident has multiple alerts. Use the alert-level view to identify the actual timestamps behind that result.

## SOC SLA reporting

SOC Reporting uses a fixed rolling 90-day window rather than the general Time Range.

Use:

- **Median / P50** for typical performance;
- **P90** for tail performance;
- **SLA breach impact** for the number and scale of breaches;
- **SLA breach investigation** to identify the incidents contributing to the result.

Severity-specific targets can be configured for High, Medium, Low, and Informational incidents. Confirm those targets match the customer's agreed service definition before using the results externally.

## Troubleshooting

| Symptom | What to check |
|---------|---------------|
| No customers or workspaces appear | Check Region, Customer Group, Customer, permissions, and whether the subscriptions contain Sentinel workspaces. |
| Only part of the estate appears | Move through the Analysis Batch values. Workspace selection is limited to the active batch. |
| The cross-workspace report is blocked | Check Workspaces in Batch and the Hard Limit. Reduce the scope or use another Analysis Batch. |
| A workspace is not available in a drill-down | Confirm that it is included in the active batch and selected by the global Workspace parameter. |
| Incident counts look too low | Check the incident creation period, latest-state reduction, product filter, owner filter, and selected workspaces. |
| Incident counts look too high | Confirm that the latest-state reduction is present and that the report is not counting historical incident versions. |
| Ingestion is empty | Check the selected Time Range, `Usage` availability, workspace permissions, and the active batch. |
| Retention is missing | Check that the table was observed during the selected period and that Azure Resource Manager table configuration is readable. |
| SOC results do not follow Time Range | SOC Reporting intentionally uses a fixed rolling 90-day window. |
| Query Pack results do not follow Time Range | The stored Query Pack query may contain its own explicit time filter. |
| Analytics rules are missing | Check workspace selection, Sentinel resource visibility, and permissions. |

## Technical notes

Sentinel Central uses Azure Resource Graph for estate discovery and Log Analytics cross-workspace queries for operational reporting.

The global workspace scope is controlled by the active Analysis Batch and Workspace parameters. Several hidden helper parameters carry workspace IDs, workspace counts, batch values, and query guards between workbook items.

The cross-workspace Hard Limit is a safety control. It is not a statement that the customer has fewer workspaces or that the remaining workspaces are unimportant.

Rows and chart series can export parameters to downstream reports. Avoid renaming internal parameters or exported fields unless all dependent workbook items are updated together.

The workbook may display inventory from Azure Resource Graph even when a Log Analytics report cannot run for the same scope. Treat inventory and telemetry as related but different views.

## Scope and limitations

Sentinel Central reports what is visible through the selected Azure subscriptions, Azure Lighthouse registrations, workspaces, permissions, retention, and ingested tables.

It cannot recover data that was not retained or was never ingested. It also cannot determine whether a configuration is acceptable for a particular customer without the customer's policy and service context.

Operational metrics should be reviewed with the service owner before being used as formal SLA or commercial evidence.

## Source workbook

The workbook this README describes is maintained in the KQLpublic repository:

[Sentinel Central v2.6.0.workbook](https://raw.githubusercontent.com/CliveWatsonQC/KQLpublic/refs/heads/master/KQL/Workbooks/Microsoft%20Sentinel%20Central/QC%20Sentinel%20Central%20v2.6.0.workbook)


# Sentinel Central — Recent Version Summary

This is a short summary of the three latest numbered Sentinel Central releases. It is intended as a quick reference alongside the workbook, not as a replacement for the full Change Log in Help.

## v2.6.0 — Customer-first workspace batching

- Replaced workspace-name grouping with customer grouping based on Azure Lighthouse customer or directory name.
- Updated the selection flow to **Region → Customer Group → Customer → Analysis Batch → Workspace**.
- Added deterministic workspace batches, with a production batch size of 30 and a 100-workspace cross-workspace safety limit.
- Made the active batch explicit, so larger estates can be covered without silently dropping workspaces.
- Made section-specific workspace selectors inherit the global scope.
- Kept Latency Investigation as a dedicated area and removed the obsolete Incident Ingestion Status navigation entry.
- Moved Help and configuration into their own parameter block, including batch and safety-limit guidance.

## v2.5.2 — Operational reporting and SOC SLA reporting

- Improved incident accuracy by using the latest state of each incident and reporting the main population by incident creation time.
- Improved incident context, filtering, alert counts, classification, MITRE tactics, and techniques.
- Reworked Ingestion around workspace configuration, billable volume, top data sources, and workspace/data-source contributors.
- Reworked Tables and Retention around observed `Usage` data, billable status, volume, last seen time, and ARM retention configuration.
- Improved Query Packs, Analytics Rules, Azure Lighthouse reporting, and cross-workspace reliability.
- Added a dedicated SOC Reporting area with a fixed rolling 90-day window.
- Added severity-specific SLA targets, median and P90 measures, breach impact, and incident-level breach investigation.

## v2.5.1 — P90 and tail-risk analysis

- Added P90 resolution-time analysis to expose the slower cases hidden by median and average measures.
- Added reporting for the worst-performing 10% of closed incidents.
- Added configurable severity-specific SLA targets, with defaults for High, Medium, and Low incidents.
- Added time-to-resolve, P90 threshold, minutes above P90, SLA breach status, and minutes over SLA.
- Added alert counts, alert-product context, direct Sentinel links, and incident drill-down from the tail-risk reports.
- Added guidance explaining the difference between median, average, P90, and long-tail performance.

## Reading the version history

The release numbering has not always followed one clean sequence. Older workbook files and Microsoft repository versions remain in the repository, and some filenames have duplicate suffixes from earlier uploads. This summary follows the latest numbered release line used by the current v2.6.0 workbook.

For the complete history, use the workbook's **Help → Change Log** option.


