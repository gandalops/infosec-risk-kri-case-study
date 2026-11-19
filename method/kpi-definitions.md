# KPI definitions: Pages 1 and 2

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

## DAX: Page 1

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

## Page 2 (Remediation tracking)

Same snapshot date (22 Oct 2026). Page 2 drills into KRI-12. It adds no new KRI.

| KPI | Rule | Source | Power BI measure | Value (checked by hand) |
|---|---|---|---|---|
| KRI-12 Overdue risk remediation % | Same as Page 1 | risk_governance.csv | Overdue % | 44 of 64 = 68.8% |
| Average days overdue | Average of (snapshot date minus due date) over open risks past their due date | risk_governance.csv, remediation_due_date, risk_status | Avg Days Overdue | 74.9 |
| Days overdue per risk | Snapshot date minus due date, for open risks past their due date only | risk_governance.csv, remediation_due_date, risk_status | Days Overdue | RSK-00066: 183, RSK-00001: 173, RSK-00059: 96, RSK-00013: 77 |
| Overdue risks by age | Overdue risks in bands 1-30, 31-60, 61-90, over 90 days | risk_governance.csv | Overdue Risks, column Age Bucket | 10, 6, 13, 15 (total 44) |
| Overdue risks by owner | Overdue Risks split by risk owner | risk_governance.csv, risk_owner | Overdue Risks | 18 owners, highest 4, total 44 |
| Open risks by milestone | Open Risks and Overdue Risks split by last_milestone | risk_governance.csv, last_milestone | Open Risks, Overdue Risks | Plan agreed 34 open, 25 overdue; Evidence collected 30 open, 19 overdue |

### Notes

- A risk due exactly on the snapshot date has 0 days overdue and is not counted as overdue.
- Age Bucket is a calculated column, because a chart axis cannot use a measure. It is based on the due date only. The Overdue Risks measure keeps closed risks out of the chart.
- No open risk has reached the Remediated milestone. All 64 open risks are at Plan agreed (34) or Evidence collected (30).
- All 64 open risks have a remediation due date.

### DAX: Page 2

```dax
Days Overdue =
VAR _snap = [Snapshot Date]
VAR _due = MAX ( FactRisk[remediation_due_date] )
RETURN
    IF (
        [Open Risks] > 0 && NOT ISBLANK ( _due ) && _due < _snap,
        DATEDIFF ( _due, _snap, DAY )
    )

Avg Days Overdue =
VAR _snap = [Snapshot Date]
RETURN
    AVERAGEX (
        FILTER (
            FactRisk,
            FactRisk[risk_status] IN { "Open", "In progress" }
                && NOT ISBLANK ( FactRisk[remediation_due_date] )
                && FactRisk[remediation_due_date] < _snap
        ),
        DATEDIFF ( FactRisk[remediation_due_date], _snap, DAY )
    )
```

Calculated column in FactRisk:

```dax
Age Bucket =
VAR _snap = MAX ( Params[SnapshotDate] )
VAR _days = DATEDIFF ( FactRisk[remediation_due_date], _snap, DAY )
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( FactRisk[remediation_due_date] ), BLANK (),
        _days <= 0, "Not overdue",
        _days <= 30, "1-30 days",
        _days <= 60, "31-60 days",
        _days <= 90, "61-90 days",
        "Over 90 days"
    )
```
