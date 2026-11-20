# Screenshots

Screenshots of the Power BI report and its data model. All data is synthetic. KRI thresholds are proposed, not approved. Snapshot date: 22 Oct 2026.

| File | Page | What it shows | Status |
|---|---|---|---|
| [00-model-view.png](00-model-view.png) | Model | Star schema, phase 1 | Done |
| [01-risk-exception-register.png](01-risk-exception-register.png) | 1 Risk and exception register | Risks, exceptions, KRI-12 to KRI-15 | Done |
| [02-remediation-tracking.png](02-remediation-tracking.png) | 2 Remediation tracking | Overdue risks by owner, age and milestone stage; KRI-12 drill-down | Done |
| 03 | 3 KRI executive summary | | Planned |
| [04-vulnerability-patch-risk.png](04-vulnerability-patch-risk.png) | 4 Vulnerability and patch risk | KRI-02, KRI-11, open and overdue findings | Done |
| 05 to 09 | Pages 5 to 9 | | Planned |

## 00 Model view

![Model view](00-model-view.png)

- Facts in the centre: FactRisk, FactVulnerability.
- Dimensions around them: DimAsset, DimOwner, DimSeverity, DimDate.
- Params is a one-row helper table holding the snapshot date. It has no relationships.
- Relationships are one-to-many, single direction, from dimension to fact. Fact tables are not connected to each other.
- Dotted lines are inactive relationships (DimDate links to several date columns, only one is active per fact).
- Known gap: the risk register has no asset ID, so FactRisk is not linked to DimAsset.
- More facts are added in later pages. See [method/data-model.md](../method/data-model.md).

## 01 Risk and exception register

![Page 1](01-risk-exception-register.png)

- 100 risks, 64 open (Open + In progress), 34 with an exception.
- KRI-12 Overdue risk remediation: 68.8% (44 of 64).
- KRI-13 Expired security exceptions: 5.
- KRI-14 Accepted risks without approved exception: 91.7% (11 of 12).
- KRI-15 Exceptions expiring within 30 days: 10.
- Definitions and formulas: [method/kpi-definitions.md](../method/kpi-definitions.md).
- Owner addresses are synthetic placeholders (example.invalid).

## 02 Remediation tracking

![Page 2](02-remediation-tracking.png)

- KRI-12 Overdue risk remediation: 68.8% (44 of 64 open risks), the same value as Page 1.
- Average days overdue: 74.9.
- By age: 10 risks at 1 to 30 days, 6 at 31 to 60, 13 at 61 to 90, 15 over 90.
- By owner: all 18 owners have at least one overdue risk. The most is 4 (owner06, owner08, owner11).
- By milestone: 34 open risks are at Plan agreed (25 overdue) and 30 at Evidence collected (19 overdue). None has reached Remediated.
- Definitions and formulas: [method/kpi-definitions.md](../method/kpi-definitions.md).

## 04 Vulnerability and patch risk

![Page 4](04-vulnerability-patch-risk.png)

- Open findings (Open + In progress): 234. Overdue: 166.
- KRI-02 Overdue Critical findings: 16.
- KRI-11 Vulnerability SLA compliance: 35.8% (120 of 335). Risk-accepted findings are left out (assumption).
- 19 open Critical and High findings are on internet-facing production assets.
- 25 findings are risk-accepted and reported separately.
- By age: 47 findings at 1 to 30 days, 42 at 31 to 60, 41 at 61 to 90, 36 over 90.
- Definitions and formulas: [method/kpi-definitions.md](../method/kpi-definitions.md).
