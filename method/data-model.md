# Data model

Model view screenshot: [screenshots/00-model-view.png](../screenshots/00-model-view.png)

One Power BI model with a star schema: fact tables hold records that are counted, dimension tables
hold descriptive lists, and each dimension filters facts in one direction (one-to-many).
Fact tables are never connected to each other.

## Tables built

| Table | Type | One row is | Rows | Built from |
|---|---|---|---|---|
| FactRisk | Fact | one risk | 100 | `risk_governance.csv`, joined with `exception_request_details.csv` on `exception_id` |
| FactVulnerability | Fact | one vulnerability finding | 360 (after removing 1 duplicate) | `vulnerability_findings.csv` |
| DimAsset | Dimension | one asset | 90 | `asset_inventory.csv` |
| DimOwner | Dimension | one owner | 18 | owners from `risk_governance.csv` and `asset_inventory.csv` |
| DimSeverity | Dimension | one severity level | 4 | `illustrative_vulnerability_sla_rules.csv` |
| DimDate | Dimension | one calendar day | | generated in DAX, marked as the date table |
| Params | Helper | one row: the snapshot date, 22 Oct 2026 | 1 | typed in |
| FactMFA | Fact | one account or asset at the snapshot of 20 Oct 2026 | 330 | `mfa_coverage.csv`, with two extra key columns (employee_id, asset_id) and a hidden sort column (criticality_order) |
| FactAuth | Fact | one login event | 1,600 | `authentication_events.csv`, with an extra date-only column (event_date) |
| DimEmployee | Dimension | one employee | 240 | `employee_directory.csv` |
| DimKRI | Lookup | one KRI | 18 | `kri_thresholds_rag.csv`, keeping only the name, owner, unit, direction and limits. The static current value and status columns are removed. It has no relationships |

## Design choices

- **Snapshot date from a table, not from today's date.** The data is static, so "overdue" is measured against 22 Oct 2026. Results do not change when the report is opened later.
- **Same grain before merging.** `risk_governance.csv` and `exception_request_details.csv` both have one row per risk-exception pair, so they merge without adding rows. FactRisk must stay at 100 rows. If it grows, the merge duplicated rows.
- **Own date table.** Power BI's automatic date feature is turned off. DimDate is used for all date filtering. Where a fact table has more than one date column, only one relationship to DimDate is active.
- **Measures, not stored columns.** KPIs are calculated on the fly, so they respond to slicers.
- **Dates converted with locale English (UK).** The raw dates are text in DD-MM-YYYY. A default conversion could read 05-10-2026 as 10 May.
- **One ID column with two kinds of key.** In `mfa_coverage.csv`, `entity_id` holds employee IDs (human accounts) and asset IDs (system accounts). One column cannot be the many side of two dimensions. FactMFA therefore has two nullable columns, employee_id and asset_id, and each relates to its own dimension. The trade-off: a slicer from DimEmployee would drop the 90 system accounts, so the MFA visuals use the business unit and criticality columns of FactMFA.
- **Date-only key for the timestamp.** `event_timestamp` in FactAuth has a time of day. A date-only column (event_date) is added next to it, and DimDate relates to that column, so each day matches exactly one DimDate row.
- **Sort helper built in Power Query.** `criticality_order` (Critical 1, High 2, Medium 3, Low 4) sorts criticality in charts. It is hidden and never plotted. It is built in Power Query because a DAX column that reads the column it sorts causes a circular dependency.
- **Lookup table without relationships.** DimKRI and Params stay unconnected. Page 3 measures read their limits and direction directly. This keeps every KRI calculated by the same measures as its own page.
- **Known gap.** The risk register has no asset ID, so FactRisk is not linked to DimAsset (see data issues A3 and A4).

## Tables not built

Pages 6 to 9 are not built, so these tables are not in the model: FactBackup (`backup_jobs.csv`), FactRecovery (`recovery_tests.csv`), FactIncident (`security_incidents.csv`), FactBreach (`data_breach_register.csv`), FactTraining (`security_training.csv`), FactPhishing (`phishing_simulations.csv`), and the AI risk and control mapping tables (`ai_ml_risk_register.csv`, `control_mapping.csv`). Page 3 shows the KRIs that depend on them as Not built.
