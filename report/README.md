# Report

[IS_Risk_KRI_Report.pdf](IS_Risk_KRI_Report.pdf) is the PDF export of the Power BI report. It has five pages, and each one looks at a different area of risk for the bank's information security risk function. This file describes what each page sets out to find out and what it found, so you can follow the result without opening the PDF.

## Page 1: Risk and exception register

This page looks at the risk register and asks whether risks are being dealt with on time and whether exceptions are properly governed. Of the 100 risks, 64 are open, and 44 of them (68.8%) have passed their remediation date (KRI-12). 34 risks carry an exception, and 5 of those have already expired (KRI-13). The weaker point is acceptance: of 12 accepted risks, 11 (91.7%) have no approved exception behind them (KRI-14). Another 10 exceptions expire within 30 days (KRI-15), and only 3 of those are approved. Taken together, risks are being accepted without the approval that makes the decision defensible, and most open risks are already late.

## Page 2: Remediation tracking

This page follows up on the overdue figure from Page 1: how late are these risks, and where does the work get stuck? Overdue risks are on average 74.9 days late, and 28 of the 44 are more than 60 days late. The longest are RSK-00066 at 183 days and RSK-00001 at 173 days. The milestones show where the work stops. No open risk has reached Remediated: 34 are at Plan agreed, 25 of them overdue, and 30 are at Evidence collected, 19 of them overdue. The lateness is not concentrated. No single owner holds more than 4 of the 44, so it points to the process as a whole and not to one team.

## Page 3: KRI executive summary

This is the management view. Of the 18 KRIs in the catalogue, 8 can be calculated from the pages in this report, and the page gives each one a status. Seven are Red (KRI-02, 07, 11, 12, 13, 14, 15), one is Amber (KRI-04), and none is Green. The other 10 show as Not built, because they depend on data areas this report does not cover. Seven Reds out of eight can mean weak control, or limits that are too strict. Only a risk-appetite decision can tell which; see [data-issues.md](../findings/data-issues.md), C10.

## Page 4: Vulnerability and patch risk

This page looks at how exposed the bank is through unpatched vulnerabilities, and whether fixes arrive on time. 234 of the 360 findings are open (after removing one duplicate), and 166 of those are overdue, 16 of them Critical (KRI-02). Only 35.8% of the findings in scope meet their deadline, 120 of 335 (KRI-11). The most urgent group is the 19 open Critical or High findings on internet-facing production assets, because they are the most exposed. The backlog is also old: 36 overdue findings are more than 90 days late, and the most overdue, VUL-000086, is 134 days late. Analytics-API-033 has the most open findings, 8. The 25 risk-accepted findings are left out of these figures.

## Page 5: Identity and MFA

This page asks whether MFA covers the accounts that should have it, and whether suspicious login activity is under control. 261 of 288 eligible accounts have MFA (90.6%, KRI-07, using the snapshot of 20 Oct). The 27 without it include 18 High-criticality accounts and no Critical ones, though that zero depends on three Critical system accounts that the source file marks as not eligible without giving a reason. Coverage is lowest in Operations (87.5%), People (88.9%) and Technology (89.2%). On logins, 26 of 424 events in September (6.1%) were suspicious (KRI-04), compared with 8.6% in July and 10.4% in August. October is partial, so it is not used. Across all months there are 140 suspicious events, most of them password spray (41). The gaps in MFA sit in High-criticality accounts and in three business units, which is where to look first.

## Reading the PDF

- Power BI prints decimals with a comma (35,8 %). The documents in this repo use a point (35.8%). The values are the same.
- All slicers are set to All in the export.
- Owner addresses (example.invalid) are placeholders.
- The Power BI file is not in this repo. Every number can be recounted from the files in [data](../data) using [method/kpi-definitions.md](../method/kpi-definitions.md).
