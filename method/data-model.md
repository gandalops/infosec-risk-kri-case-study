# Data model

One Power BI model with a star schema: fact tables hold records that are counted, dimension tables
hold descriptive lists, and each dimension filters facts in one direction (one-to-many).
Fact tables are never connected to each other.

## Tables built so far

| Table | Type | One row is | Rows | Built from |
|---|---|---|---|---|
| FactRisk | Fact | one risk | 100 | `risk_governance.csv`, joined with `exception_request_details.csv` on `exception_id` |
| FactVulnerability | Fact | one vulnerability finding | 360 (after removing 1 duplicate) | `vulnerability_findings.csv` |
| DimAsset | Dimension | one asset | 90 | `asset_inventory.csv` |
| DimOwner | Dimension | one owner | 18 | owners from `risk_governance.csv` and `asset_inventory.csv` |
| DimSeverity | Dimension | one severity level | 4 | `illustrative_vulnerability_sla_rules.csv` |
| DimDate | Dimension | one calendar day | | generated in DAX, marked as the date table |
| Params | Helper | one row: the snapshot date, 22 Oct 2026 | 1 | typed in |

## Design choices

- **Snapshot date from a table, not from today's date.** The data is static, so "overdue" is measured against 22 Oct 2026. Results do not change when the report is opened later.
- **Same grain before merging.** `risk_governance.csv` and `exception_request_details.csv` both have one row per risk-exception pair, so they merge without adding rows. FactRisk must stay at 100 rows. If it grows, the merge duplicated rows.
- **Own date table.** Power BI's automatic date feature is turned off. DimDate is used for all date filtering. Where a fact table has more than one date column, only one relationship to DimDate is active.
- **Measures, not stored columns.** KPIs are calculated on the fly, so they respond to slicers.
- **Dates converted with locale English (UK).** The raw dates are text in DD-MM-YYYY. A default conversion could read 05-10-2026 as 10 May.
- **Known gap.** The risk register has no asset ID, so FactRisk is not linked to DimAsset (see data issues A3 and A4).
