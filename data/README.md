# Data folder

The 18 files stand in for the data an information security risk function receives from other teams: registers kept by risk management, logs from security tooling, lookup lists from IT and HR, and the definitions of the KRIs. They are raw inputs and are not edited here. Cleaning happens in Power Query, and how each issue is handled is in [findings/data-issues.md](../findings/data-issues.md). Email addresses use `example.invalid`.

In the tables below, "In the report" shows the pages that use a file. A dash means the file is not used.

## 1. Registers: lists of recorded risks, exceptions and incidents

The record of what the organisation has identified and decided. The risk and exception registers are what a risk analyst maintains and reports on.

| File | One row is | Rows | In the report |
|---|---|---|---|
| `risk_governance.csv` | one risk in the main risk register | 100 | Pages 1, 2, 3 |
| `exception_request_details.csv` | one exception request (a request to break a rule) | 34 | Page 1 |
| `ai_ml_risk_register.csv` | one AI/ML risk | 14 | – |
| `security_incidents.csv` | one security incident | 120 | – |
| `data_breach_register.csv` | one data breach | 8 | – |

## 2. Activity logs: things that happened, counted to produce KRIs

Evidence of what actually happens in the environment. The KRIs are counted from these.

| File | One row is | Rows | In the report |
|---|---|---|---|
| `vulnerability_findings.csv` | one vulnerability finding | 361 (360 unique) | Pages 3, 4 |
| `authentication_events.csv` | one login event | 1,600 | Pages 3, 5 |
| `mfa_coverage.csv` | one account or system, with MFA on or off | 330 | Pages 3, 5 |
| `backup_jobs.csv` | one backup job | 500 | – |
| `recovery_tests.csv` | one recovery test | 60 | – |
| `phishing_simulations.csv` | one recipient in one phishing campaign | 600 | – |
| `security_training.csv` | one training assignment | 455 | – |

## 3. Lookup lists: who and what the other files refer to

Context that turns an ID into something readable: which asset, which person, which deadline.

| File | One row is | Rows | Used for |
|---|---|---|---|
| `asset_inventory.csv` | one system or asset | 90 | asset owner, criticality, internet-facing (Pages 4, 5) |
| `employee_directory.csv` | one employee | 240 | business unit, role, MFA eligibility (Page 5) |
| `illustrative_vulnerability_sla_rules.csv` | one severity level | 4 | fix deadline by severity, illustrative only (Page 4) |

## 4. Rulebooks: definitions and limits

How each KRI is defined, and when it turns amber or red.

| File | One row is | Rows | What it adds |
|---|---|---|---|
| `kri_thresholds_rag.csv` | one KRI with green/amber/red limits | 18 | Direction and green and amber limits for each KRI. Adds KRI-11 to KRI-15 and splits KRI-10 into four. The `Current_Value` column is not used; every value is recalculated. Page 3. |
| `KRI_CATALOGUE.csv` | one KRI with definition and owner | 10 | The KRI definitions and escalation actions I followed. Its thresholds are text, so the numeric limits come from `kri_thresholds_rag.csv`. |
| `control_mapping.csv` | one control mapped to DORA, ISO 27001, NIST CSF | 23 | Links each control to KRIs and evidence fields. |

## Known data issues

Counts, effects and handling for every issue found in these files are in [findings/data-issues.md](../findings/data-issues.md).
