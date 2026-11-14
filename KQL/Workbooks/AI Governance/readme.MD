# AI Governance

## What this workbook is for

AI Governance provides a practical view of AI services, products, agents and tools observed through Microsoft Defender telemetry.

It is intended to help answer:

- Which AI services and products are being used?
- Which activity is mapped to the selected sanctioned policy?
- Which AI users, devices and products are active?
- Which users are new to AI activity or are using multiple AI products?
- Which local and cloud AI agents or tools are being observed?
- Which sanctioned agents have not been observed during the selected period?
- What additional AI products or agents should be considered for catalogue or policy review?

This is a governance and investigation workbook. It is not proof of user intent, the content submitted to an AI service, or the amount of time a user actively spent using it.

## Current version

The latest version currently identified in the repository is:

`clive_AI_Preview (10 v1.0).workbook`

The file name still describes the workbook as a preview. Treat changes to the public catalogues, local watchlists and telemetry availability as part of the assessment context.

## Before using the workbook

Select the scope in this order:

1. **Customer**
2. **Workspace**
3. **Time Range**
4. **Sanctioned AI Services** where policy comparison is required
5. **Sanctioned AI Agents** where agent governance is required

The Customer parameter selects the accessible subscription scope. Workspace then narrows the Sentinel or Defender workspace population.

The Help parameter provides workbook guidance and the Change Log.

## Workbook tabs

### Summary

Summary shows the breadth of the detection catalogues used by the workbook:

- AI service providers and products;
- AI agent providers and tools;
- AI Deep Search providers and products.

These tiles measure catalogue coverage. They do not measure observed usage during the selected Time Range.

### AI Services

AI Services provides the main service-usage and governance views, including:

- sanctioned and unsanctioned AI product tiles;
- provider and product summaries;
- AI Compliance & Governance Review;
- New AI Users;
- Multi-AI Product Usage;
- AI Usage & Adoption;
- AI Usage by User.

The reports classify recognised activity against the selected sanctioned AI service policy.

### AI Agents

AI Agents provides local and cloud agent views, including:

- Local AI Agent Governance at a Glance;
- Local Sanctioned Agents Not Observed;
- Cloud AI Agent Governance at a Glance;
- Cloud Sanctioned Agents Not Observed.

Local reports primarily use endpoint process evidence. Cloud reports use the cloud audit, identity and agent-related tables available in the environment.

### AI Deep Search

AI Deep Search uses a broader consolidated AI and agent watchlist to identify products that are not already covered by the standard catalogues.

Use these results as an investigation and catalogue-enrichment queue. An item marked `AlreadyCovered=false` is not automatically a governance finding; it is an indication that the item needs review or may need to be added to a local watchlist.

## Governance model

The workbook keeps three ideas separate:

1. **Catalogue coverage** — whether a service, product, agent or tool is recognised by the detection sources.
2. **Observed activity** — whether matching telemetry was seen in the selected scope and time range.
3. **Governance status** — whether the observed item is included in the selected sanctioned policy.

The normal interpretation is:

- 🟢 **Sanctioned** — observed activity maps to a selected approved policy identity.
- 🔴 **Unsanctioned** — recognised activity was observed but is not included in the selected approved policy.
- ⚪ **Sanctioned not observed** — the item is selected as sanctioned but no matching evidence was found in the selected period.
- 🟠 **Mixed** — the same user has been observed using both sanctioned and unsanctioned AI products.

If no sanctioned services are selected, recognised AI activity is classified as unsanctioned. That is a policy-selection result, not a conclusion that every AI service is prohibited.

Risk classification and recommended action come from the catalogues. They are separate from sanctioned or unsanctioned status.

## Catalogue and watchlist inputs

### Public catalogues

The workbook reads public CSV catalogues through `externaldata` for:

- AI services and products;
- AI agents and tools;
- consolidated AI Deep Search indicators.

The public catalogues are the baseline detection coverage. They can change independently of the workbook, so record the catalogue state when retaining assessment evidence.

### `AIGovernance` watchlist

The optional Sentinel watchlist alias `AIGovernance` adds customer-specific AI products and policy defaults.

The expected fields are:

| Field | Purpose |
|---|---|
| `Key` | Local identifier. |
| `PolicyKey` | Stable policy identity used by the sanctioned selector. |
| `Provider` / `Product` | Display and reporting values. |
| `Domain` | Network match value. |
| `Category` | Product category. |
| `Sanctioned` | Default policy selection. |
| `RiskType` | Risk or use classification. |
| `SourceURL` | Reference for the entry. |
| `ProcessName` | Local process match value. |
| `ProcessProductName` / `ProcessPublisher` | Optional process enrichment. |

Local entries are merged with the public catalogue. Local entries take precedence for matching entries and are shown with `[Local]` where the report exposes that label.

The repository includes a starting file:

[`AIGovernance_PolicyKey.csv`](https://github.com/CliveWatsonQC/KQLpublic/blob/master/KQL/Workbooks/AI%20Governance/AIGovernance_PolicyKey.csv)

### `AIAgentGovernance` watchlist

The optional Sentinel watchlist alias `AIAgentGovernance` provides customer-specific agent governance and sanctioned defaults.

The expected fields are:

| Field | Purpose |
|---|---|
| `Key` | Stable local identifier. |
| `AgentName` | Display name. |
| `PolicyKey` | Stable policy identity. |
| `Vendor` | Agent or tool vendor. |
| `Category` | Agent category. |
| `Sanctioned` | Whether the agent is selected as approved by default. |
| `SourceURL` | Reference for the entry. |

Rows marked `Sanctioned=TRUE` are selected automatically by the workbook.

The repository includes a starting file:

[`ai_agent_governance_watchlist.csv`](https://github.com/CliveWatsonQC/KQLpublic/blob/master/KQL/Workbooks/AI%20Governance/ai_agent_governance_watchlist.csv)

## Evidence sources

| Source | Main use |
|---|---|
| `DeviceNetworkEvents` | Connections to recognised AI services and products. |
| `DeviceProcessEvents` | Execution of recognised AI clients, agents, model hosts and tools. |
| `OfficeActivity` | Microsoft 365 audit evidence for cloud agent or service activity where available. |
| `AuditLogs` | Microsoft Entra audit evidence where available. |
| `AADServicePrincipalSignInLogs` | Workload identity and service-principal sign-in context. |
| Agent-related tables | Additional cloud or agent telemetry when present in the tenant. |
| `AIGovernance` | Customer-specific AI products and policy defaults. |
| `AIAgentGovernance` | Customer-specific AI agents and sanctioned defaults. |

Cloud and agent sources are queried opportunistically. Results depend on the available Microsoft Defender or Sentinel tables, permissions, endpoint onboarding, licensing, retention and connector configuration.

## How to interpret the reports

### Observed activity

Network evidence shows that a device or account was associated with a matching connection. Process evidence shows that a matching process was observed. Audit and identity evidence shows that a matching service or workload event was recorded.

These signals do not establish:

- what the user intended;
- what content was submitted;
- whether the activity was authorised outside the selected workbook policy;
- how long the user actively used the product;
- whether a connection represented meaningful use rather than background traffic.

### New AI Users

New AI Users identifies users whose first observed AI activity occurred during the most recent seven days.

The report uses a fixed 30-day observation window to provide historical context. A user may be new within that observation window rather than new to the organisation. Treat this as an observed-telemetry measure, not a definitive employment or onboarding measure.

### Multi-AI Product Usage

Multi-AI Product Usage identifies users observed accessing multiple AI products or providers during the selected Time Range. Review these results alongside the product, device and evidence fields before deciding whether any action is required.

### Sanctioned not observed

This result means that the item is selected in the sanctioned policy but no matching evidence was found in the selected scope and time range. It does not prove that the product is not being used. Possible explanations include:

- the product is not covered by the current match fields;
- the relevant device or cloud source is not available;
- the activity occurred outside the selected period;
- telemetry was not retained;
- the product was used from an unmanaged device or another environment.

## Practical operating model

Use the workbook in this order:

1. Review **Summary** to understand catalogue coverage.
2. Confirm Customer, Workspace and Time Range.
3. Select the customer's sanctioned AI service and agent policies.
4. Review **AI Compliance & Governance Review** for the main service picture.
5. Use **AI Usage & Adoption** and **AI Usage by User** to understand adoption and concentration.
6. Review **New AI Users** and **Multi-AI Product Usage** as investigation queues.
7. Review local and cloud agent governance.
8. Use **AI Deep Search** to identify catalogue gaps and candidate watchlist additions.
9. Update the local watchlists only after validating the product, match value, source and customer policy.

## Troubleshooting

| Symptom | What to check |
|---|---|
| No services or products appear | Check Customer, Workspace, Time Range, catalogue availability and Defender permissions. |
| Everything is unsanctioned | Check whether sanctioned services were selected and whether local `AIGovernance` rows loaded correctly. |
| A public product is missing | Check the public catalogue, then add a validated local entry to `AIGovernance` if required. |
| A local product is missing | Check the watchlist alias, column names, `PolicyKey`, match field and Sentinel watchlist upload. |
| Agent results are empty | Check `DeviceProcessEvents` and the available cloud or agent telemetry in the selected workspace. |
| Cloud agent results are incomplete | Confirm that `OfficeActivity`, `AuditLogs`, service-principal sign-ins and any agent-related sources are available and retained. |
| Sanctioned not observed is unexpectedly high | Check the selected Time Range, match coverage, endpoint onboarding and whether the product is used outside the monitored estate. |
| Deep Search returns candidates already known to the customer | Check `AlreadyCovered`, standard catalogue entries and local watchlists before treating the row as a gap. |
| Catalogue-driven queries fail or return no data | Check access to the external CSV sources used by `externaldata`; use local watchlists for customer-specific additions and validate the public source separately. |

## Scope and limitations

AI Governance reports what is visible through the selected Defender or Sentinel workspaces and the telemetry retained there. It cannot recover activity that was never ingested, was not retained, occurred on an unmanaged device, or is not covered by the current catalogue match fields.

The workbook does not replace an AI policy, acceptable-use process, data-protection review or customer-specific risk assessment. Use the results to support those conversations, not to make unsupported conclusions about individual users.

## Source workbook

This is the latest version currently identified in the repository:

- [GitHub file — `clive_AI_Preview (10 v1.0).workbook`](https://github.com/CliveWatsonQC/KQLpublic/blob/master/KQL/Workbooks/AI%20Governance/clive_AI_Preview%20%2810%20v1.0%29.workbook)
- [Raw workbook — `clive_AI_Preview (10 v1.0).workbook`](https://raw.githubusercontent.com/CliveWatsonQC/KQLpublic/refs/heads/master/KQL/Workbooks/AI%20Governance/clive_AI_Preview%20%2810%20v1.0%29.workbook)

