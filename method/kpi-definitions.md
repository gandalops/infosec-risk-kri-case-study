# KPI definitions: Page 1 (Risk and Exception Register)

Snapshot date: 22 Oct 2026. All values are recalculated from the raw data. The static Current_Value column in kri_thresholds_rag.csv is not used. Thresholds in that file are proposed, not approved.

| KPI | Rule | Source | Power BI measure | Value (checked by hand) |
|---|---|---|---|---|
| Total risks | Count of risks | risk_governance.csv, risk_id | Total Risks | 100 |
| Open risks | Risks with status Open or In progress | risk_governance.csv, risk_status | Open Risks | 64 |
| Exceptions | Risks that have an exception ID | risk_governance.csv, exception_id | Exceptions | 34 |
| KRI-12 Overdue risk remediation % | Open risks with due date before the snapshot, divided by open risks | risk_governance.csv, remediation_due_date, risk_status | Overdue Risks, Overdue % | 44 of 64 = 68.8% |
| KRI-13 Expired security exceptions | Exceptions with expiry date before the snapshot | risk_governance.csv, exception_expiry_date | Expired Exceptions | 5 |
| KRI-14 Accepted risks without approved exception % | Accepted risks whose exception is not Approved (or absent), divided by accepted risks | risk_governance.csv, risk_status, exception_approval_status | Accepted Without Approval, Accepted Without Approval % | 11 of 12 = 91.7% |
| KRI-15 Exceptions expiring within 30 days | Exceptions with expiry date from 22 Oct to 21 Nov 2026 | risk_governance.csv, exception_expiry_date | Exceptions Expiring 30d | 10 |

How to check by hand: filter the CSV in Excel on the rule above and count the rows.

## Notes

- KRI-15 counts every exception in the window, whatever its approval status. Of the 10, 3 are Rejected, 4 are Pending and 3 are Approved. Whether Rejected exceptions should count is an open question: see [findings/data-issues.md](../findings/data-issues.md), section C.
- The 30-day window is implemented in DAX. It is not defined in the rulebook.

## DAX

```dax
Snapshot Date = MAX ( Params[SnapshotDate] )

Total Risks = COUNTROWS ( FactRisk )

Open Risks =
CALCULATE ( COUNTROWS ( FactRisk ), FactRisk[risk_status] IN { "Open", "In progress" } )

Exceptions =
CALCULATE ( COUNTROWS ( FactRisk ), FactRisk[exception_id] <> BLANK () )

Overdue Risks =
VAR _snap = [Snapshot Date]
RETURN
    CALCULATE ( [Open Risks], FactRisk[remediation_due_date] < _snap )

Overdue % = DIVIDE ( [Overdue Risks], [Open Risks] )

Expired Exceptions =
VAR SnapDate = [Snapshot Date]
RETURN
    COALESCE (
        CALCULATE (
            COUNTROWS ( FactRisk ),
            FactRisk[exception_id] <> BLANK (),
            FactRisk[exception_expiry_date] < SnapDate
        ),
        0
    )

Accepted Risks =
COALESCE ( CALCULATE ( COUNTROWS ( FactRisk ), FactRisk[risk_status] = "Accepted" ), 0 )

Accepted Without Approval =
COALESCE (
    CALCULATE ( [Accepted Risks], FactRisk[exception_approval_status] <> "Approved" ),
    0
)

Accepted Without Approval % =
DIVIDE ( [Accepted Without Approval], [Accepted Risks] )

Exceptions Expiring 30d =
VAR SnapDate = [Snapshot Date]
RETURN
    COALESCE (
        CALCULATE (
            COUNTROWS ( FactRisk ),
            FactRisk[exception_id] <> BLANK (),
            FactRisk[exception_expiry_date] >= SnapDate,
            FactRisk[exception_expiry_date] <= SnapDate + 30
        ),
        0
    )

Days to Expiry =
DATEDIFF ( [Snapshot Date], MAX ( FactRisk[exception_expiry_date] ), DAY )
```

`Days to Expiry` is only valid in a table with one row per exception. Do not use it on a card.
