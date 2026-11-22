# Report

PDF export of the Power BI report. Five pages are built.

[IS_Risk_KRI_Report.pdf](IS_Risk_KRI_Report.pdf)

| Page | Content |
|---|---|
| 1 Risk and exception register | KRI-12 to KRI-15 |
| 2 Remediation tracking | KRI-12 drill-down |
| 3 KRI executive summary | Status of the 8 KRIs that have a page; the other 10 show as Not built |
| 4 Vulnerability and patch risk | KRI-02, KRI-11 |
| 5 Identity and MFA | KRI-07, KRI-04 |

Pages 6 to 9 are not built. See the progress table in the [main README](../README.md).

## Notes for reading the PDF

- The data is synthetic. KRI thresholds are proposed, not approved. Snapshot date: 22 Oct 2026 (MFA snapshot 20 Oct 2026).
- Power BI shows decimals with a comma (for example 35,8 %). The documents in this repo use a point (35.8%). The values are the same.
- All slicers are set to All in the export.
- Owner addresses (example.invalid) are synthetic placeholders.
- The Power BI file itself is not in this repo. The numbers can be recounted from the files in [data](../data), using [method/kpi-definitions.md](../method/kpi-definitions.md).
- On Page 3, Red, Amber and Green come from proposed limits that are not approved. The status word is always printed in the cell, so the page also reads in grayscale.
