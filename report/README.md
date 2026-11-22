# Report

PDF export of the Power BI report. Four pages are built.

[IS_Risk_KRI_Report.pdf](IS_Risk_KRI_Report.pdf)

| Page | Content |
|---|---|
| 1 Risk and exception register | KRI-12 to KRI-15 |
| 2 Remediation tracking | KRI-12 drill-down |
| 4 Vulnerability and patch risk | KRI-02, KRI-11 |
| 5 Identity and MFA | KRI-07, KRI-04 |

Pages 3 and 6 to 9 are not built. See the progress table in the [main README](../README.md).

## Notes for reading the PDF

- The data is synthetic. KRI thresholds are proposed, not approved. Snapshot date: 22 Oct 2026 (MFA snapshot 20 Oct 2026).
- Power BI shows decimals with a comma (for example 35,8 %). The documents in this repo use a point (35.8%). The values are the same.
- All slicers are set to All in the export.
- Owner addresses (example.invalid) are synthetic placeholders.
- The Power BI file itself is not in this repo. The numbers can be recounted from the files in [data](../data), using [method/kpi-definitions.md](../method/kpi-definitions.md).
