# InfoSec risk and KRI reporting: financial sector case study

A self-initiated learning project, not work experience. I built a Power BI risk report for a
fictional bank, then read it like a risk analyst: what is wrong, why it matters, and what I would
recommend. All data is synthetic. Snapshot date: 22 Oct 2026. KRI thresholds are proposed, not approved.

## What I found (Page 1: risk and exception register)

- 44 of 64 open risks (68.8%) are past their remediation due date.
- 11 of 12 accepted risks have no approved exception.
- 5 exceptions have already expired, and 10 more expire within 30 days. 3 of the 5 were never decided.
- None of the 34 exceptions records an approver, so no approval can be audited.

Each number can be recounted from the raw files. The steps are in [method/kpi-definitions.md](method/kpi-definitions.md).

## Where to look

| Folder | What you will find |
|---|---|
| [findings](findings) | A risk assessment of 6 risks, and a log of data problems with counts |
| [method](method) | How the data model is built and how each KPI is calculated |
| [data](data) | The 18 source files, grouped, with row counts and known issues |
| [screenshots](screenshots) | Report pages (coming) |
| [report](report) | The Power BI report file (coming) |

If you have two minutes: read the findings above, then [findings/risk-assessment.md](findings/risk-assessment.md).

## Progress

| Page | Status |
|---|---|
| 1 Risk and exception register | built |
| 2 Remediation tracking | in progress |
| 3 to 9: KRI summary, vulnerability, identity and MFA, resilience, incidents and breaches, human risk, AI risk and controls | planned |

## Limits

- The data is synthetic and made up. Findings describe this dataset, not a real company.
- The main risk register has only a severity. Likelihood, impact, threats and treatments in the risk assessment are my assumptions, and they are labelled.
- Regulation references (DORA, ISO 27001, NIST CSF) in `control_mapping.csv` are drafts and need Compliance review.
- Several fields in the exception file are empty ("TO FILL"). I report this as a finding and did not fill them in.

## Tools

Power BI (data model, DAX measures, report), Power Query (cleaning), Excel and CSV for checking numbers.
