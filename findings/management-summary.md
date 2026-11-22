# Management summary

The MFA figures use the snapshot of 20 Oct 2026. Everything else uses 22 Oct 2026.

This page lists the five risks I would raise first, based on Pages 1, 2, 4 and 5 of the report. Every number can be recounted from the raw files: [method/kpi-definitions.md](../method/kpi-definitions.md). 

**KRI status (Page 3, proposed limits):** 7 of the 8 KRIs with a report page are Red, 1 is Amber (KRI-04) and none is Green. The other 10 KRIs have no page and are not rated. Seven Reds may also mean the limits are strict; see [data-issues.md](data-issues.md), C10. The page is [here](../screenshots/03-kri-executive-summary.png).

The ranking, the recommended actions and the suggested owners are my judgment, not facts from the data. The data has no owner for these actions, so the owners are roles I suggest.

| # | Risk | Evidence | Why it matters | Recommended action | Suggested owner |
|---|---|---|---|---|---|
| 1 | Vulnerabilities are not fixed on time | 166 of 234 open findings are overdue (70.9%). 16 of the 18 open Critical findings are overdue. 36 findings are more than 90 days late. SLA compliance (KRI-11) is 35.8%, proposed green limit 95%. 19 open Critical and High findings are on internet-facing production assets | Late fixes on exposed systems are the most direct route for an attacker. The gap to the proposed limit is very large | Fix the 19 internet-facing Critical and High findings first, with dates agreed with asset owners. Report overdue Critical findings weekly until the number falls | Vulnerability management lead, with asset owners |
| 2 | Exceptions and risk acceptances are not properly approved | None of the 34 exceptions records an approver. 5 exceptions have expired, 3 of them were never decided. 11 of 12 accepted risks have no approved exception. Only 3 of 25 risk-accepted findings have an approved exception that has not expired | A risk accepted without a valid approval is not a decision anyone can defend in an audit | Stop accepting risks without an approved, dated exception. Review the 5 expired and the 10 expiring within 30 days. Fill approver and justification for every open exception | Information security risk manager, with the risk owners |
| 3 | Risk remediation is late across the whole organisation | 44 of 64 open risks are overdue (68.8%), 75 days late on average, 15 more than 90 days late. All 18 owners have at least one overdue risk. No open risk has reached the Remediated milestone | Lateness is spread across all owners, so it points to a process problem, not one weak team | Agree new dates for the 15 risks that are over 90 days late, or formally accept them. Review the overdue list monthly | Risk manager, with risk owners |
| 4 | MFA is missing on 27 eligible accounts, and Critical exemptions are unexplained | MFA coverage (KRI-07) is 90.6% (261 of 288), proposed green limit 98%. 27 eligible entities have no MFA: 18 High, 6 Medium, 3 Low. Three Critical system accounts are marked not eligible, with no reason recorded | Missing MFA on privileged or high-criticality accounts raises the chance of account takeover. The Critical figure of 0 depends on an unexplained exemption | Enforce MFA on the 18 High-criticality gaps first, starting with Finance, Operations and People (six each). Ask the account owners why AST-0021, AST-0043 and AST-0064 are exempt | Identity and access management lead |
| 5 | Suspicious authentication activity needs a confirmed definition | KRI-04 was 6.1% in September 2026 (26 of 424 events), after 8.6% in July and 10.4% in August. 140 of 1,600 events are classed as suspicious. Events exist for Terminated (51) and Leave (88) accounts, but the directory has no termination date | The KRI swings between Amber and Red depending on the month. The suspicious definition is not confirmed with policy, and the activity of departed staff cannot be judged | Confirm with policy which classifications count as suspicious. Add termination dates to the directory, then check activity after departure | Security operations lead, with HR for the dates |

## What I could not conclude

- Risk-accepted findings are left out of KRI-11. The rulebook does not say how to treat them. See [findings/data-issues.md](data-issues.md), section C.
- I did not analyse `security_incidents.csv`, so I cannot say whether any of these risks has already led to an incident.
- Regulation links in `control_mapping.csv` (DORA, ISO 27001, NIST CSF) are drafts and are not used here. They need Compliance review.
- The Red and Amber statuses depend on proposed limits that nobody has approved. Seven Reds may also mean the limits are strict. See [data-issues.md](data-issues.md), C10.
