# KPI definitions: Pages 1 to 5

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

## Page 3 (KRI executive summary)

Same snapshot date (22 Oct 2026). Page 3 adds no new calculation. It reads each KRI from the measures of Pages 1, 2, 4 and 5 and compares it with the limits in `kri_thresholds_rag.csv` (proposed, not approved). The static `Current_Value` and `Current_RAG` columns are removed in Power Query and are never used.

| Item | Rule | Power BI measure | Value (checked by hand) |
|---|---|---|---|
| KRI value | Value of the selected KRI, in the same unit as the limits (percentages in points, counts as counts) | KRI Value | KRI-02 16, KRI-04 6.1, KRI-07 90.6, KRI-11 35.8, KRI-12 68.8, KRI-13 5, KRI-14 91.7, KRI-15 10 |
| KRI status | Green, Amber or Red from the direction and the two limits. Not built if the KRI has no page | KRI RAG Status | KRI-04 Amber. KRI-02, 07, 11, 12, 13, 14, 15 Red. 10 KRIs Not built |
| Status counts | Number of KRIs per status | KRIs Red, KRIs Amber, KRIs Green, KRIs Not Built | 7, 1, 0, 10 (total 18) |

### Notes

- DimKRI has one row per KRI (18 rows) and no relationships. Measures read its limits directly with SELECTEDVALUE.
- Direction decides the rule. Lower is better: Green if the value is at or below the green limit, Amber if at or below the amber limit, else Red. Higher is better: Green if at or above the green limit, Amber if at or above the amber limit, else Red.
- The limits in the file are in percentage points (5 means 5%), but the percentage measures are fractions (0.061). The percentage KRIs are multiplied by 100 so both sides use the same unit.
- KRI-07 is automatically Red if a Critical eligible entity has no MFA, whatever the percentage. It is not triggered today (0).
- KRI-02 is wrapped in COALESCE so that zero overdue Critical findings shows as 0 and not as Not built.
- Report Page is a Power Query column that records which page shows each KRI. The 10 KRIs without a page (KRI-01, 03, 05, 06, 08, 09, 10a to 10d) are shown as Not built and are not calculated.
- The total row of the table is switched off. No single KRI is selected there, so the status is blank.
- Seven of eight KRIs are Red under the proposed limits. This may also mean the limits are strict. They need a risk-appetite decision. See [findings/data-issues.md](../findings/data-issues.md), section C.

### DAX: Page 3

```dax
KRI Value =
VAR _id = SELECTEDVALUE ( DimKRI[KRI_ID] )
RETURN
    SWITCH (
        _id,
        "KRI-02", COALESCE ( [Overdue Critical Findings], 0 ),
        "KRI-04", [KRI-04 Suspicious %] * 100,
        "KRI-07", [KRI-07 MFA Coverage %] * 100,
        "KRI-11", [KRI-11 SLA Compliance %] * 100,
        "KRI-12", [Overdue %] * 100,
        "KRI-13", [Expired Exceptions],
        "KRI-14", [Accepted Without Approval %] * 100,
        "KRI-15", [Exceptions Expiring 30d],
        BLANK ()
    )

KRI RAG Status =
VAR _id = SELECTEDVALUE ( DimKRI[KRI_ID] )
VAR _v = [KRI Value]
VAR _dir = SELECTEDVALUE ( DimKRI[Direction] )
VAR _green = SELECTEDVALUE ( DimKRI[Green_Limit] )
VAR _amber = SELECTEDVALUE ( DimKRI[Amber_Limit] )
VAR _override = _id = "KRI-07" && [Critical Eligible Without MFA] > 0
RETURN
    IF (
        ISBLANK ( _id ), BLANK (),
        IF (
            ISBLANK ( _v ), "Not built",
            IF (
                _override, "Red",
                IF (
                    _dir = "Lower is better",
                    IF ( _v <= _green, "Green", IF ( _v <= _amber, "Amber", "Red" ) ),
                    IF ( _v >= _green, "Green", IF ( _v >= _amber, "Amber", "Red" ) )
                )
            )
        )
    )

KRIs Red = COALESCE ( COUNTROWS ( FILTER ( DimKRI, [KRI RAG Status] = "Red" ) ), 0 )

KRIs Amber = COALESCE ( COUNTROWS ( FILTER ( DimKRI, [KRI RAG Status] = "Amber" ) ), 0 )

KRIs Green = COALESCE ( COUNTROWS ( FILTER ( DimKRI, [KRI RAG Status] = "Green" ) ), 0 )

KRIs Not Built = COALESCE ( COUNTROWS ( FILTER ( DimKRI, [KRI RAG Status] = "Not built" ) ), 0 )
```

Display measures (text, for reading only, not used in any calculation):

```dax
KRI Value (display) =
VAR _v = [KRI Value]
VAR _unit = SELECTEDVALUE ( DimKRI[Unit] )
RETURN
    IF (
        ISBLANK ( _v ), BLANK (),
        SWITCH (
            _unit,
            "%", FORMAT ( _v, "0.0" ) & " %",
            "count", FORMAT ( _v, "0" ),
            FORMAT ( _v, "0.0" )
        )
    )

KRI Limits (proposed) =
VAR _dir = SELECTEDVALUE ( DimKRI[Direction] )
VAR _g = SELECTEDVALUE ( DimKRI[Green_Limit] )
VAR _a = SELECTEDVALUE ( DimKRI[Amber_Limit] )
VAR _u = IF ( SELECTEDVALUE ( DimKRI[Unit] ) = "%", " %", "" )
RETURN
    IF (
        ISBLANK ( _dir ), BLANK (),
        IF (
            _dir = "Lower is better",
            "Green ≤ " & FORMAT ( _g, "0" ) & _u & " · Amber ≤ " & FORMAT ( _a, "0" ) & _u,
            "Green ≥ " & FORMAT ( _g, "0" ) & _u & " · Amber ≥ " & FORMAT ( _a, "0" ) & _u
        )
    )

KRI RAG Colour =
SWITCH (
    [KRI RAG Status],
    "Red", "#C62828",
    "Amber", "#F9A825",
    "Green", "#2E7D32",
    "Not built", "#E0E0E0",
    BLANK ()
)

KRI RAG Font Colour =
SWITCH (
    [KRI RAG Status],
    "Red", "#FFFFFF",
    "Amber", "#1A1A1A",
    "Green", "#FFFFFF",
    "Not built", "#666666",
    BLANK ()
)
```

Colour is never the only signal. The status word is always shown in the cell.

## Page 4 (Vulnerability and patch risk)

Same snapshot date (22 Oct 2026). Source: vulnerability_findings.csv (360 findings after removing one exact duplicate, VUL-000017) and asset_inventory.csv. Open means workflow status Open or In progress.

| KPI | Rule | Power BI measure | Value (checked by hand) |
|---|---|---|---|
| Open findings | Status Open or In progress | Open Findings | 234 |
| Overdue findings | Open findings with due date before the snapshot | Overdue Findings | 166 |
| KRI-02 Overdue Critical findings | Overdue findings with severity Critical | Overdue Critical Findings | 16 |
| KRI-11 Vulnerability SLA compliance % | (Closed on or before the due date + open not yet due) divided by (closed + open). Risk-accepted findings are left out. | Compliant Findings, SLA Findings In Scope, KRI-11 SLA Compliance % | 35.8% (120 of 335) |
| Critical/High, internet-facing | Open Critical and High findings on assets with internet_facing = Yes and environment = Production | Critical High Internet Facing Open | 19 |
| Risk-accepted findings | Status Risk accepted | Risk Accepted Findings | 25 |
| Overdue findings by age | Overdue findings in bands 1-30, 31-60, 61-90, over 90 days | Overdue Findings, column Finding Age Bucket | 47, 42, 41, 36 (total 166) |
| Open findings by severity | Open and overdue findings split by severity | Open Findings, Overdue Findings | Critical 18/16, High 64/56, Medium 101/78, Low 51/16 |
| Top assets by open findings | Open findings per asset, top 10 by value, ties included | Open Findings | Analytics-API-033 = 8; 12 assets shown because of ties at 5 |
| Most overdue findings | Days between due date and snapshot, open findings past due | Finding Days Overdue | VUL-000086: 134, VUL-000227: 132, VUL-000248: 129 |

### Notes

- Risk-accepted findings are left out of Open Findings, KRI-02 and KRI-11. This is an assumption, because the rulebook does not say how to treat them. See [findings/data-issues.md](../findings/data-issues.md), section C.
- A finding due exactly on the snapshot date is not overdue (1 finding).
- KRI-11 uses the due date stored in each finding. The SLA targets in illustrative_vulnerability_sla_rules.csv are not applied again, and they are illustrative only.
- The KRI-02 card counts Critical findings only, so it is blank when the Severity slicer is set to another severity.

### DAX: Page 4

```dax
Open Findings =
CALCULATE (
    COUNTROWS ( FactVulnerability ),
    FactVulnerability[workflow_status] IN { "Open", "In progress" }
)

Overdue Findings =
VAR _snap = [Snapshot Date]
RETURN
    CALCULATE ( [Open Findings], FactVulnerability[remediation_due_date] < _snap )

Overdue Critical Findings =
CALCULATE ( [Overdue Findings], FactVulnerability[severity] = "Critical" )

Compliant Findings =
VAR _snap = [Snapshot Date]
RETURN
    CALCULATE (
        COUNTROWS ( FactVulnerability ),
        FILTER (
            FactVulnerability,
            ( FactVulnerability[workflow_status] = "Closed"
                && FactVulnerability[closed_date] <= FactVulnerability[remediation_due_date] )
                || ( FactVulnerability[workflow_status] IN { "Open", "In progress" }
                    && FactVulnerability[remediation_due_date] >= _snap )
        )
    )

SLA Findings In Scope =
CALCULATE (
    COUNTROWS ( FactVulnerability ),
    FactVulnerability[workflow_status] IN { "Closed", "Open", "In progress" }
)

KRI-11 SLA Compliance % =
DIVIDE ( [Compliant Findings], [SLA Findings In Scope] )

Critical High Internet Facing Open =
CALCULATE (
    [Open Findings],
    FactVulnerability[severity] IN { "Critical", "High" },
    DimAsset[internet_facing] = "Yes",
    DimAsset[environment] = "Production"
)

Risk Accepted Findings =
CALCULATE (
    COUNTROWS ( FactVulnerability ),
    FactVulnerability[workflow_status] = "Risk accepted"
)

Finding Days Overdue =
VAR _snap = [Snapshot Date]
VAR _due = MAX ( FactVulnerability[remediation_due_date] )
RETURN
    IF (
        [Open Findings] > 0 && NOT ISBLANK ( _due ) && _due < _snap,
        DATEDIFF ( _due, _snap, DAY )
    )
```

Calculated column in FactVulnerability:

```dax
Finding Age Bucket =
VAR _snap = MAX ( Params[SnapshotDate] )
VAR _days = DATEDIFF ( FactVulnerability[remediation_due_date], _snap, DAY )
RETURN
    SWITCH (
        TRUE (),
        ISBLANK ( FactVulnerability[remediation_due_date] ), BLANK (),
        _days <= 0, "Not overdue",
        _days <= 30, "1-30 days",
        _days <= 60, "31-60 days",
        _days <= 90, "61-90 days",
        "Over 90 days"
    )
```


## Page 5 (Identity and MFA)

Sources: mfa_coverage.csv (330 rows, one row per account or asset), authentication_events.csv (1,600 rows, one row per event) and employee_directory.csv (240 rows). KRI-07 uses the MFA snapshot of 20 Oct 2026. KRI-04 uses the latest full month before the report snapshot of 22 Oct 2026, which is September 2026.

| KPI | Rule | Power BI measure | Value (checked by hand) |
|---|---|---|---|
| Eligible entities | Rows with eligible_for_mfa = Yes | Eligible Entities | 288 |
| KRI-07 MFA coverage % | Eligible entities with MFA enabled, divided by eligible entities | Eligible With MFA, KRI-07 MFA Coverage % | 261 of 288 = 90.6% |
| Critical eligible without MFA | Eligible, mfa_enabled = No, criticality = Critical. Automatic Red if above 0 | Critical Eligible Without MFA | 0 |
| Eligible without MFA | Eligible entities with mfa_enabled = No | Eligible Without MFA | 27 (High 18, Medium 6, Low 3, Critical 0) |
| MFA coverage by criticality | KRI-07 split by criticality | KRI-07 MFA Coverage % | Critical 100.0%, High 86.5%, Medium 93.2%, Low 95.2% |
| MFA coverage by business unit | KRI-07 split by business_unit | KRI-07 MFA Coverage % | Customer 95.6%, Retail 93.5%, Finance 89.7%, Technology 89.2%, People 88.9%, Operations 87.5% |
| KRI-04 Suspicious authentication events % | Suspicious events divided by all events, latest full month | Suspicious Events (Latest Full Month), Auth Events (Latest Full Month), KRI-04 Suspicious % | Sep 2026: 26 of 424 = 6.1% |
| Suspicious events by type | Events per suspicious classification, all months | Total Auth Events with a visual-level filter | Password spray 41, impossible travel 35, disabled account attempt 34, MFA fatigue 30 (total 140) |
| Suspicious % by month | Suspicious events divided by all events, per month | Suspicious Auth % | Jul 8.6%, Aug 10.4%, Sep 6.1%, Oct 10.3% (partial, data ends 21 Oct) |

### Notes

- Proposed thresholds, not approved. KRI-07: Green at 98% or above, Amber at 95% or above, Red below 95%, and automatic Red if a Critical eligible entity has no MFA. KRI-04: Green at 5% or below, Amber at 10% or below, Red above 10%. At 90.6%, KRI-07 would be Red. At 6.1%, KRI-04 would be Amber.
- Suspicious means one of four event classifications: Password spray suspected, Impossible travel, Disabled account attempt, MFA fatigue suspected. This definition is not confirmed with policy. See [findings/data-issues.md](../findings/data-issues.md), section C.
- Three Critical system accounts (AST-0021, AST-0043, AST-0064) are marked not eligible in the source file, so they are outside KRI-07. The file gives no reason. This is why Critical eligible without MFA is 0.
- entity_id in mfa_coverage.csv holds both employee keys (EMP) and asset keys (AST). Two key columns, employee_id and asset_id, were created in Power Query so each can relate to its own dimension. The 90 system accounts have no employee_id.
- The business unit and criticality visuals on the MFA side use columns of FactMFA, not of DimEmployee. A DimEmployee slicer would drop the 90 system accounts. This is a deliberate exception to the rule that slicers come from dimensions.
- criticality_order is a Power Query column used only to sort criticality (Critical, High, Medium, Low). It is hidden and not used on any chart.
- FactAuth relates to DimDate through event_date (date only), not through the timestamp.
- Events from accounts whose employee status is Terminated (51) or Leave (88) exist. The directory has no termination date, so these cannot be judged as post-termination activity.
- October 2026 is partial, so it is not used for KRI-04.

### DAX: Page 5

```dax
Eligible Entities =
CALCULATE ( COUNTROWS ( FactMFA ), FactMFA[eligible_for_mfa] = "Yes" )

Eligible With MFA =
CALCULATE (
    COUNTROWS ( FactMFA ),
    FactMFA[eligible_for_mfa] = "Yes",
    FactMFA[mfa_enabled] = "Yes"
)

KRI-07 MFA Coverage % = DIVIDE ( [Eligible With MFA], [Eligible Entities] )

KRI-07 Target = 0.98

Eligible Without MFA =
COALESCE (
    CALCULATE (
        COUNTROWS ( FactMFA ),
        FactMFA[eligible_for_mfa] = "Yes",
        FactMFA[mfa_enabled] = "No"
    ),
    0
)

Critical Eligible Without MFA =
COALESCE (
    CALCULATE (
        COUNTROWS ( FactMFA ),
        FactMFA[eligible_for_mfa] = "Yes",
        FactMFA[mfa_enabled] = "No",
        FactMFA[criticality] = "Critical"
    ),
    0
)

Total Auth Events = COUNTROWS ( FactAuth )

Suspicious Auth Events =
CALCULATE (
    COUNTROWS ( FactAuth ),
    FactAuth[event_classification] IN {
        "Password spray suspected",
        "Impossible travel",
        "Disabled account attempt",
        "MFA fatigue suspected"
    }
)

Suspicious Auth % = DIVIDE ( [Suspicious Auth Events], [Total Auth Events] )

Auth Events (Latest Full Month) =
VAR SnapDate = [Snapshot Date]
VAR MonthStart = EOMONTH ( SnapDate, -2 ) + 1
VAR MonthEnd = EOMONTH ( SnapDate, -1 )
RETURN
    CALCULATE (
        [Total Auth Events],
        DimDate[Date] >= MonthStart,
        DimDate[Date] <= MonthEnd
    )

Suspicious Events (Latest Full Month) =
VAR SnapDate = [Snapshot Date]
VAR MonthStart = EOMONTH ( SnapDate, -2 ) + 1
VAR MonthEnd = EOMONTH ( SnapDate, -1 )
RETURN
    CALCULATE (
        [Suspicious Auth Events],
        DimDate[Date] >= MonthStart,
        DimDate[Date] <= MonthEnd
    )

KRI-04 Suspicious % =
DIVIDE ( [Suspicious Events (Latest Full Month)], [Auth Events (Latest Full Month)] )
```

`Suspicious Auth Events` must not be split by event_classification. CALCULATE overrides the filter on that column, so every type would show 140. The treemap uses `Total Auth Events` with a visual-level filter on the four classifications instead.
