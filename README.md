# InfoSec risk and KRI reporting: a financial sector case study

A self-initiated learning project, not work experience: a Power BI risk report built on synthetic data for a fictional bank. I took the role of an information security risk analyst, built the report from 18 files, and then read it as an analyst would: what is wrong, why it matters, and what I would recommend. Snapshot date: 22 Oct 2026. KRI thresholds are proposed, not approved.

**Start here:** [findings/management-summary.md](findings/management-summary.md) lists the five risks I would raise first, each with a number, why it matters, a recommended action and a suggested owner.

## Objective

An information security risk analyst keeps the risk and exception register current, follows up on remediation, and reports key risk indicators (KRIs) to management. The evidence for this sits in many places: registers, vulnerability scans, identity logs, thresholds. It is rarely in one view, and the numbers often disagree with each other.

This project shows how I would bring that evidence into one model and answer five questions a risk function is asked:

1. Is the risk register reliable, and are risk acceptances properly approved?
2. Where is remediation stuck, and who owns it?
3. Which KRIs need management's attention, and can the thresholds be trusted?
4. Are vulnerabilities fixed on time, and where is the exposure highest?
5. Are identities protected, and is there sign of attack?

It also shows how I deal with weak data: I report gaps as findings, record every assumption, and do not fill in missing values.

## The data

The 18 files fit four groups:

| Group | Files | Purpose |
|---|---|---|
| Registers | `risk_governance`, `exception_request_details`, `ai_ml_risk_register`, `security_incidents`, `data_breach_register` | The record of risks, exceptions, incidents and breaches |
| Activity logs | `vulnerability_findings`, `authentication_events`, `mfa_coverage`, `backup_jobs`, `recovery_tests`, `phishing_simulations`, `security_training` | Evidence of what actually happens in the environment |
| Lookup lists | `asset_inventory`, `employee_directory`, `illustrative_vulnerability_sla_rules` | Context: which asset, which person, which SLA target |
| Rulebooks | `kri_thresholds_rag`, `KRI_CATALOGUE`, `control_mapping` | How each KRI is defined, its proposed limits, and links to controls |

Nine files feed the report: the risk register and exception file, vulnerabilities, assets and SLA rules, authentication events, MFA coverage, the employee directory, and the KRI thresholds. `KRI_CATALOGUE` gives the KRI definitions I followed. 

## How it is built

One data model with a star schema, so every page uses the same definitions and the numbers agree between pages. Every KRI is recalculated from the raw data. The static values in the threshold file are not used. The report uses a fixed snapshot date and not today's date, so "overdue" gives the same answer whenever someone opens it. The steps are in [method](method).

## The report, page by page

### Page 1: Risk and exception register

**Question:** is the register a reliable record, and are risk acceptances properly governed?

The register is the core record of an information security risk function. An accepted risk is one the organisation chooses to live with. That choice has to be approved, dated and traceable.

- 44 of 64 open risks (68.8%) are past their remediation due date.
- 11 of 12 accepted risks have no approved exception behind them.
- 5 exceptions have already expired, and 3 of those were never decided. 10 more expire within 30 days.
- None of the 34 exceptions records an approver.

**Why it matters:** the register cannot show who decided to accept which risk, or when. In an audit, that is a governance failure, not a data problem.

![Page 1: risk and exception register](screenshots/01-risk-exception-register.png)

### Page 2: Remediation tracking

**Question:** where is remediation stuck, and who owns it?

Page 1 says remediation is late. This page asks how late, and whether the problem sits with a few owners or everywhere.

- The 44 overdue risks are 75 days late on average. 15 are more than 90 days late.
- All 18 risk owners have at least one overdue risk. No owner has more than 4.
- No open risk has reached the Remediated milestone. The milestone column only repeats the status, so it cannot show real progress.

**Why it matters:** lateness spread across every owner points to a process problem, not one weak team. Without dated milestones, nobody can tell whether a plan is moving or has stalled.

![Page 2: remediation tracking](screenshots/02-remediation-tracking.png)

### Page 3: KRI executive summary

**Question:** which KRIs need management's attention, and can the thresholds be trusted?

This is the page for management: one table with each KRI's value, its status, its owner and the limits behind the status. It reads the same measures as the other pages, so it cannot contradict them.

- 8 of the 18 KRIs have a page behind them. Under the proposed limits, 7 are Red and 1 is Amber.
- The other 10 are shown as Not built. They are not hidden and not rated.

**Why it matters:** seven Reds can mean the controls are weak, or that the limits are strict. The limits are proposed and nobody has approved them. So the colours show direction, and a risk-appetite decision has to settle what they mean before anyone escalates.

![Page 3: KRI executive summary](screenshots/03-kri-executive-summary.png)

### Page 4: Vulnerability and patch risk

**Question:** are vulnerabilities fixed on time, and where is the exposure highest?

- 166 of 234 open findings (70.9%) are past their due date. 16 of the 18 open Critical findings are overdue.
- SLA compliance (KRI-11) is 35.8%, against a proposed green limit of 95%.
- 19 open Critical and High findings sit on internet-facing production assets.
- Only 3 of the 25 risk-accepted findings have an approved exception that has not expired.

**Why it matters:** late fixes on systems exposed to the internet are the most direct route for an attacker. Accepting a finding without a valid approval repeats the governance gap from Page 1, this time on technical risk.

![Page 4: vulnerability and patch risk](screenshots/04-vulnerability-patch-risk.png)

### Page 5: Identity and MFA

**Question:** are accounts protected, and is there sign of attack?

- MFA coverage (KRI-07) is 90.6% (261 of 288 eligible entities), against a proposed green limit of 98%. 27 eligible entities have none, 18 of them High criticality.
- No Critical entity lacks MFA, but only because three Critical system accounts are marked not eligible, and the file gives no reason.
- Suspicious authentication events (KRI-04) were 6.1% in September 2026, after 8.6% in July and 10.4% in August.

**Why it matters:** missing MFA on high-criticality accounts raises the risk of account takeover. The zero for Critical depends on an exemption nobody explains. The suspicious-event definition is my assumption, and the directory has no termination dates, so I cannot judge activity from departed staff.

![Page 5: identity and MFA](screenshots/05-identity-mfa.png)

## Judgment calls and limits

- The risk register holds only a severity. The likelihood, impact, threats and treatments in the risk assessment are my assumptions, and they are labelled.
- KRI-11 leaves risk-accepted findings out, because the rulebook does not say how to treat them. The SLA targets are illustrative.
- Several fields in the exception file are empty ("TO FILL"). I report this as a finding and did not fill them in.
- Regulation references (DORA, ISO 27001, NIST CSF) in `control_mapping.csv` are drafts and need Compliance review.
- Most KRIs are a single snapshot, so the report shows the position on 22 Oct 2026, not a trend.

## What this does not cover

Incidents and breaches, backup and recovery, security training and phishing, and AI risk with control mapping have data in the repo, but no report page. I left them out to check the five pages above against the raw data. Page 3 shows the KRIs that depend on them as Not built.

## Where to look

| Folder | What you will find |
|---|---|
| [findings](findings) | The management summary, a risk assessment of 6 risks, and a log of data problems with counts |
| [method](method) | How the data model is built and how each KPI is calculated, with the DAX |
| [data](data) | The 18 source files, grouped, with row counts and known issues |
| [screenshots](screenshots) | The data model and each report page |
| [report](report) | A PDF of the five-page report |

For the judgment behind the summary, read [findings/risk-assessment.md](findings/risk-assessment.md). For what is wrong with the data, read [findings/data-issues.md](findings/data-issues.md).

## Tools

Power BI (data model, DAX measures, report), Power Query (cleaning), Excel and CSV for checking numbers.
