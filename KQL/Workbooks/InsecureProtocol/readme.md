# Insecure Protocols

## What this workbook is for

This workbook helps us find and assess legacy authentication and insecure-protocol use across a customer's Defender, Entra, and Sentinel data.

It is intended to answer:

- What insecure-protocol evidence have we actually seen?
- Which scenarios can the available telemetry support?
- Which tables are missing, stale, or only providing context?
- Which findings should be investigated or remediated first?
- Are findings new, persistent, or no longer being observed?

The workbook is Defender-first, but the queries remain usable for customers who are still running Sentinel. The current transition assumption is that both experiences may be used through March 2027.

## Important principle

An absence of data is not the same as evidence that a protocol is not being used.

The workbook therefore reports coverage separately from findings. If a table is missing or has not reported in the selected period, that is shown as a telemetry gap. It is not converted into a zero count or a low-risk result.

## Before using the findings

Select:

- **Customer**
- **Workspace**
- **Time Range**

Start with **Coverage**. Check that the sources needed for the scenarios of interest are reporting before interpreting the Findings page.

The workbook uses the selected `TimeRange` by default. The Remediation page uses a fixed 30-day lookback because it needs earlier evidence to identify lifecycle status.

## Workbook pages

### Executive Summary

The short view of the current position:

- tables observed;
- tables unavailable;
- scenario coverage;
- most recent observation.

Use this page to decide whether the rest of the workbook can be interpreted with confidence.

### Coverage

This page reports the expected source tables, their requirement level, recent records, freshness, and group status.

It also shows which scenarios are potentially covered:

- **Potentially covered** — direct evidence tables reported data.
- **Context only** — supporting telemetry reported data, but direct evidence did not.
- **Not observed** — no relevant telemetry was observed in the selected period.

For example, `DeviceNetworkEvents` activity on port 389 may indicate LDAP-related traffic. It does not prove that an insecure LDAP bind took place.

### Findings

This is the main evidence table. It remains at finding detail rather than reducing everything to one row per protocol.

The table supports filtering, search, grouping, export, and up to 1,000 displayed rows.

Select an **Account** cell to display the conditional **Selected Account Context** table. That table shows supporting events for the selected account over the selected Time Range. It is investigation context and does not change the finding's risk score.

### Evidence

This page shows the underlying protocol-related signals from Defender and identity sources. Use it when a finding needs to be checked against the raw signal that produced it.

### Prioritization

This page adds available customer-impact context to the protocol evidence.

The base protocol score is kept separate from the priority score. Where the source data exposes useful indicators, the page can identify items such as:

- domain controllers;
- privileged identities;
- critical or internet-exposed assets;
- service identities;
- classified machine groups.

If those indicators are not available, the item is **Unclassified**. That does not reduce the protocol risk.

The page also groups evidence by the best available device, account, or target entity to show where one remediation action may remove the most observed exposure.

### Remediation

This page compares the selected period with the preceding 30 days.

- **New** — observed in the selected period but not in the lookback.
- **Persistent** — observed both before and during the selected period.
- **Resolved** — observed in the lookback but not in the selected period.

It also provides a recommended action based on the protocol scenario.

Owner, due-date, and exception fields are visible placeholders at this stage. They are not persisted by the workbook and should not be treated as a completed remediation workflow.

### Trends

This page shows daily movement in observed findings, affected devices, affected accounts, risk level, and protocol.

Use it to look for changes after a remediation action, while checking the Coverage page to make sure a change in telemetry has not been mistaken for a reduction in exposure.

## Source tables

The queries use `union isfuzzy=true` so that customers can have different table sets without causing the workbook to fail.

| Area | Tables | Main use |
|------|--------|----------|
| Windows security events | `SecurityEvent` | NTLM, Kerberos, and authentication evidence. |
| Windows non-security events | `Event`, `WindowsEvent` | LDAP, SMB, and Netlogon event paths. |
| Entra authentication | `SigninLogs`, `AADNonInteractiveUserSignInLogs`, `AADServicePrincipalSignInLogs`, `AADManagedIdentitySignInLogs` | Legacy-client and authentication activity. |
| Defender for Identity | `IdentityLogonEvents`, `IdentityDirectoryEvents` | Identity and directory context. |
| Defender for Endpoint | `DeviceInfo`, `DeviceLogonEvents`, `DeviceNetworkEvents`, `DeviceRegistryEvents` | Device, network, logon, and configuration evidence. |

The presence of a table is not enough to prove coverage. The table must also contain relevant data in the selected period and expose the fields needed by the detection logic.

## Risk model

The initial model is evidence-based and deliberately visible in the Findings output.

| Field | Meaning |
|-------|---------|
| `BaseRiskScore` | Severity assigned to the protocol and evidence type. |
| `RiskScore` | Base score after the defined outcome adjustment. |
| `Risk` | Colour-coded risk label. |
| `Confidence` | How directly the evidence supports the finding. |
| `Status` | Success, failure, context-only, or direct observation. |
| `RiskReason` | Why the finding was classified. |
| `Activity` | Best available human-readable activity or event description. |
| `EventLog` | Windows event-log context when available. |

The current scoring is:

- 🔴 **Critical** — score 5, including WDigest enabled and Kerberos DES.
- 🟠 **High** — score 4, including NTLMv1, insecure LDAP, SMBv1, vulnerable Netlogon, and Kerberos RC4.
- 🟡 **Medium** — score 3, including failed Entra legacy authentication and generic NTLM credential validation.
- 🟢 **Low** — scores 1–2, normally supporting network context rather than direct proof.

A successful Entra legacy-authentication event receives a one-level increase. Network-only LDAP port 389 evidence receives a one-level reduction because it does not prove an insecure bind.

Customer impact is not silently mixed into this base score. The Prioritization page shows it separately where the data supports it.

## Protocol-specific limitations

### NTLMv1

An event 4624 is treated as NTLMv1 only when:

- the authentication package is NTLM;
- `LmPackageName` indicates NTLMv1; and
- the account is not an anonymous logon.

Event 4776 is labelled **NTLM credential validation**. It is not automatically treated as NTLMv1 because the event alone does not prove the NTLM version.

### LDAP

Port 389 activity is context. It is not proof of a cleartext or unsigned LDAP bind. Direct evidence depends on the relevant event or identity table being available and populated.

### WDigest

The finding requires `UseLogonCredential` to be enabled. Registry writes that set the value to `0` are not findings.

### Kerberos

The query looks for legacy ticket-encryption values in the available event data. The result depends on the encryption type being exposed in the source record.

## How the workbook should be used

1. Select the customer, workspace, and Time Range.
2. Review Required Tables / Coverage.
3. Review Scenario Coverage before treating a missing finding as meaningful.
4. Use Findings to inspect the normalized evidence.
5. Select an Account cell when supporting context is needed.
6. Use Evidence to check the underlying signal.
7. Use Prioritization to identify important or concentrated exposure.
8. Use Remediation to separate new, persistent, and resolved exposure.
9. Use Trends to check whether observed activity is changing.

## What this workbook does not yet know

The workbook does not automatically know:

- which accounts are privileged under the customer's policy;
- which servers are business-critical;
- which devices are domain controllers or Tier 0 assets unless the source data identifies them;
- which findings have an approved exception;
- who owns the remediation;
- when a remediation action is due;
- whether a legacy dependency is acceptable for the business.

Those details need customer-approved enrichment or a connected remediation register. Until then, the workbook should be used to support the discussion, not to make the final change decision.

## Technical notes

The queries:

- use the selected workspace and Time Range;
- fail gracefully when source tables are unavailable;
- normalise event time and common fields across different schemas;
- preserve row-level evidence in Findings;
- use the same protocol and risk logic across the dashboard views;
- treat coverage gaps separately from protocol findings.

The workbook reports observed telemetry. It cannot prove that an unobserved protocol is absent, and it cannot recover history that was not retained in Defender or Sentinel.
