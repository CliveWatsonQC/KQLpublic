# Licence Governance

## What this workbook is for

This workbook brings together two different views of the tenant:

- current licence and directory state from Microsoft Graph;
- successful interactive sign-in activity observed in Microsoft Sentinel.

It is intended to help answer questions such as:

- Which Microsoft licences are subscribed and assigned?
- Which licences are security-related according to the workbook's catalogue?
- Which users currently have a licence assigned?
- Which licensed accounts are disabled or have limited recent interactive activity?
- Which active directory members have no assigned licence?
- Does the Microsoft Graph tenant match the selected Sentinel workspace tenant?

This is a review and reconciliation workbook. It is not a replacement for Microsoft licensing, entitlement, procurement, billing, or access-review processes.

## Important principle

The workbook combines data with different meanings and different time horizons:

- **Licence state** is a current Microsoft Graph view.
- **Directory state** is a current Microsoft Graph view.
- **Activity** is successful interactive sign-in activity observed in the selected Sentinel workspace over the last 90 days.

These datasets are useful together, but they are not the same thing. An account that has not appeared in the activity data is a candidate for review. It is not automatically an unused account or a licence to remove.

## Before using the workbook

Select:

- **Customer**
- **Workspace**
- **Time Range**

The Customer and Workspace parameters use the same Azure Lighthouse and Sentinel workspace conventions as the other workbooks.

The **Time Range** controls the workbook views that use it. The licence and directory data is current Graph state, and the reconciliation activity query uses its own 90-day window.

Use the **Help** parameter to show the workbook guidance or Change Log.

## Workbook layout

### Tenant Context Control

The workbook obtains the tenant ID from Microsoft Graph and the tenant ID associated with the selected Sentinel workspace. It then produces `GraphTenantMatch`.

- **Match** — Graph-based licence and directory reports are shown.
- **Mismatch** — Graph-based reports are hidden to prevent data from one tenant being interpreted as belonging to another.

Selecting a Sentinel workspace does not change the Microsoft Graph directory available to the current Defender session. If the tenant does not match, check the signed-in account and portal context before using the Graph reports.

### Microsoft Graph and Workspace Context

This section shows the tenant and workspace values used by the workbook. It is there to make the data boundary visible before the licence reports are interpreted.

### Microsoft Licensing Reports

The Graph licence reports use the current `subscribedSkus` response and the embedded SKU catalogue.

The workbook provides separate views for:

- all Microsoft licences;
- security-related licences;
- general or other licences.

The detailed licence table can be used to select an individual SKU and show the users currently assigned that SKU.

### Directory Licence Reconciliation

This section combines Microsoft Graph directory members with Sentinel activity. It groups the results by:

- licence state;
- account state;
- recent interactive activity.

The activity categories are:

- **Seen in last 30 days**;
- **Seen in 31–90 days**;
- **Not seen in 90 days**.

The licence categories are:

- **Licensed**;
- **Unlicensed**.

The account categories are:

- **Enabled**;
- **Disabled**.

The result is intended to identify review candidates, such as disabled accounts retaining licence assignments, licensed accounts with limited observed activity, and active unlicensed members.

### Licence and Activity Headline Metrics

The headline metrics summarise the reconciliation result. They should be read with the tenant match status and the activity limitations in mind.

### Help and Change Log

Help is shown using Markdown. The Change Log records workbook changes and the main reporting decisions.

## Data sources

| Source | Purpose |
|--------|---------|
| Microsoft Graph `/v1.0/organization` | Resolve the Graph tenant ID. |
| Microsoft Graph `/v1.0/subscribedSkus` | Read current subscribed SKU and assigned-unit information. |
| Microsoft Graph `/v1.0/users` | Read current member accounts, account state, assigned licences, and directory identifiers. |
| `SigninLogs` | Identify successful interactive sign-ins observed in the selected Sentinel workspace. |
| Azure Resource Graph `resources` | Select subscriptions and workspaces. |
| `ManagedServicesResources` | Group customer subscriptions by Azure Lighthouse directory where available. |
| `ResourceContainers` | Resolve the tenant ID associated with the selected subscription. |
| Embedded SKU catalogue | Provide shorter names and workbook security classifications for SKU part numbers. |

The workbook needs Microsoft Graph access for the licence and directory views. It needs access to the selected Sentinel workspace and `SigninLogs` for the activity reconciliation.

## How to interpret the licence views

The SKU catalogue provides a reporting classification:

- **Security / Identity** — products classified as directly security or identity related.
- **Security-capable suite** — broader suites classified as security-relevant by the workbook.
- **General / Other** — products not currently classified as security-related.

This classification is maintained inside the workbook. It is not a Microsoft entitlement decision, and it should not be used as proof that a particular SKU provides or does not provide a specific security control.

The subscribed SKU data shows the current Graph view of the directory. It does not by itself establish whether the organisation's contract, purchase agreement, licence terms, or internal assignment policy is being followed.

## How to interpret the reconciliation

The reconciliation uses the Entra user ID to match current Graph directory members with Sentinel activity.

The activity query counts successful interactive sign-ins over 90 days. It does not represent every form of account use.

For example:

- **Licensed + Disabled** is a review signal, not an automatic removal decision.
- **Licensed + Not seen in 90 days** may be a dormant user, a user active outside the selected workspace, or a user using non-interactive access.
- **Unlicensed + Seen in last 30 days** is a review signal. It may reflect a required licence outside the workbook's catalogue, a service or exception process, or an incomplete licence assignment.
- **Unlicensed + Not seen in 90 days** is normally lower priority, but it is still a directory record rather than proof of irrelevance.

Use the assigned-user drill-down and the directory identifiers to validate the result before taking action.

## Activity limitations

The activity view is deliberately narrow:

- it uses `SigninLogs`;
- it looks at successful interactive sign-ins;
- it uses the selected Sentinel workspace;
- it looks back 90 days;
- it matches users through the Entra user ID.

It does not provide a complete usage report. It may not show activity performed through:

- non-interactive authentication;
- service principals or managed identities;
- workloads that do not write to the selected workspace;
- activity outside the retention period;
- accounts excluded by the workbook's filtering rules.

The current activity query also excludes conventionally named service accounts beginning with `svc`. Review this filter before using the workbook for a customer with a different service-account naming convention.

The current version also contains a workspace-specific branch for `qc-prod` and `@quorumcyber.com` identities. This should be removed or parameterised before distributing the workbook to another customer.

## Customer-specific checks before sharing

The workbook is intended to be reusable, but check the following before distributing it:

- remove saved subscription, workspace, and tenant values from exported JSON;
- confirm that the Graph tenant matches the selected Sentinel workspace tenant;
- check the Graph permissions available to the signed-in user;
- confirm that `SigninLogs` is present and retained for the selected workspace;
- review the embedded SKU catalogue and security classifications;
- review any account exclusions or naming conventions in the activity query;
- remove or parameterise the current `qc-prod` and `@quorumcyber.com` activity exception;
- confirm that the user population and licence rules match the customer's process.

The workbook should not contain a customer-specific Graph tenant, workspace, or subscription as its portable default.

## Troubleshooting

| Symptom | What to check |
|---------|---------------|
| Graph reports are hidden | Check `GraphTenantMatch`, the signed-in Defender account, and the selected workspace tenant. |
| Graph tenant ID is empty | Check Microsoft Graph access and the Defender portal context. |
| No licence data is shown | Confirm Graph permissions and that the Graph reports are running in the Defender portal. |
| No activity is shown | Check the selected workspace, `SigninLogs`, retention, the 90-day window, and the success filter. |
| Counts look lower than expected | Check whether the account is a guest, service account, non-interactive identity, or outside the selected workspace. |
| A licence is classified incorrectly | Review the embedded SKU catalogue and update the reporting classification. |
| A licensed account looks unused | Treat it as a review candidate. Check non-interactive use, other workspaces, business ownership, and the customer's licence policy. |
| A user appears unlicensed | Confirm the current Graph assignment, the expected licence rule, and whether the account is covered by an exception or separate process. |

## Technical notes

The workbook is designed for the Microsoft Defender portal because its Microsoft Graph workbook queries use the Defender Graph endpoint capability. Sentinel remains the activity source for the reconciliation view.

The main data boundary is controlled by `GraphTenantMatch`. Graph-based report groups are conditionally hidden when the Graph directory and selected workspace tenant do not match.

The workbook uses an embedded SKU catalogue rather than relying on an external CSV at run time. That makes the workbook easier to deploy, but it also means the catalogue must be reviewed as Microsoft licensing changes.

The workbook is an example and a starting point for governance discussion. It is not production-ready entitlement tooling without validation of tenant context, Graph permissions, SKU classification, account scope, activity coverage, and the customer's licensing process.

## Source workbook

The workbook this README describes is maintained in the KQLpublic repository:

[LicensingGovernance.workbook](https://raw.githubusercontent.com/CliveWatsonQC/KQLpublic/refs/heads/master/KQL/Workbooks/MSFT%20License/LicensingGovernance.workbook)
