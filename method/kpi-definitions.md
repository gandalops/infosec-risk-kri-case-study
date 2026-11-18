# KPI definitions: Page 1 (Risk and Exception Register)

Snapshot date: 22 Oct 2026. All values are recalculated from the raw data. The static
`Current_Value` column in `kri_thresholds_rag.csv` is not used. Thresholds in that file are
proposed, not approved.

| KPI | Rule | Source | Value (checked by hand) |
|---|---|---|---|
| Total risks | Count of risks | `risk_governance.csv`, `risk_id` | 100 |
| Open risks | Risks with status Open or In progress | `risk_governance.csv`, `risk_status` | 64 |
| Exceptions | Risks that have an exception ID | `risk_governance.csv`, `exception_id` | 34 |
| KRI-12 Overdue risk remediation % | Open risks with due date before the snapshot, divided by open risks | `risk_governance.csv`, `remediation_due_date`, `risk_status` | 44 of 64 = 68.8% |
| KRI-13 Expired security exceptions | Exceptions with expiry date before the snapshot | `risk_governance.csv`, `exception_expiry_date` | 5 |
| KRI-14 Accepted risks without approved exception % | Accepted risks whose exception is not Approved (or absent), divided by accepted risks | `risk_governance.csv`, `risk_status`, `exception_approval_status` | 11 of 12 = 91.7% |
| KRI-15 Exceptions expiring within 30 days | Exceptions with expiry date from 22 Oct to 21 Nov 2026 | `risk_governance.csv`, `exception_expiry_date` | 10 |

How to check by hand: filter the CSV in Excel on the rule above and count the rows.
The measure names in the Power BI file will be added when the report file is uploaded.
