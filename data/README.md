# Data folder

All data in this folder is synthetic (made up). Email addresses use `example.invalid`.
Snapshot date for all reports: 22 Oct 2026.

These files are raw inputs. They are not edited here. Cleaning happens in Power BI
(Power Query) and is documented in `/method`.

## 1. Registers: lists of recorded risks, exceptions and incidents

| File | One row is | Rows | Used on page |
|---|---|---|---|
| `risk_governance.csv` | one risk in the main risk register | 100 | 1, 2 |
| `exception_request_details.csv` | one exception request (a request to break a rule) | 34 | 1 |
| `ai_ml_risk_register.csv` | one AI/ML risk | 14 | 9 |
| `security_incidents.csv` | one security incident | 120 | 7 |
| `data_breach_register.csv` | one data breach | 8 | 7 |

## 2. Activity logs: things that happened, counted to produce KRIs

| File | One row is | Rows | Used on page |
|---|---|---|---|
| `vulnerability_findings.csv` | one vulnerability finding | 361 (360 unique) | 2, 4 |
| `authentication_events.csv` | one login event | 1,600 | 5 |
| `mfa_coverage.csv` | one account or system, with MFA on or off | 330 | 5 |
| `backup_jobs.csv` | one backup job | 500 | 6 |
| `recovery_tests.csv` | one recovery test | 60 | 6 |
| `phishing_simulations.csv` | one recipient in one phishing campaign | 600 | 8 |
| `security_training.csv` | one training assignment | 455 | 8 |

## 3. Lookup lists: who and what the other files refer to

| File | One row is | Rows | Used for |
|---|---|---|---|
| `asset_inventory.csv` | one system or asset | 90 | asset owner, criticality, internet-facing |
| `employee_directory.csv` | one employee | 240 | business unit, role, MFA eligibility |
| `illustrative_vulnerability_sla_rules.csv` | one severity level | 4 | fix deadline by severity (illustrative only) |

## 4. Rulebooks: definitions and limits

| File | One row is | Rows | Status |
|---|---|---|---|
| `kri_thresholds_rag.csv` | one KRI with green/amber/red limits | 18 | thresholds are PROPOSED, not approved. The static `Current_Value` column is not used; values are recalculated. |
| `KRI_CATALOGUE.csv` | one KRI with definition and owner | 10 | reference only |
| `control_mapping.csv` | one control mapped to DORA, ISO 27001, NIST CSF | 23 | DRAFT: article references need Compliance review |

## Known data issues (verified)

| File | Issue |
|---|---|
| `exception_request_details.csv` | Justification, requester and approver are "TO FILL" in all 34 rows. Approval date is blank in all 34. |
| `risk_governance.csv` | Dates are text in DD-MM-YYYY. Some have a trailing "00:00". |
| `vulnerability_findings.csv` | 361 rows but 360 unique finding IDs (one duplicate). Severity and patch status have mixed case and extra spaces. 10 of the 17 exception IDs used here do not exist in the exception file. |
| `security_incidents.csv` | 5 incidents have blank detected, response or contained times. Severity has mixed case. |
| `backup_jobs.csv` | `job_result` has mixed case and extra spaces. |
| `recovery_tests.csv` | 10 tests are marked Pass although the data integrity check failed. |
| `security_training.csv` | 198 rows show completion before assignment. 14 due dates include a time. |
| `exception_request_details.csv`, `risk_governance.csv` | There is no asset ID in the risk register, so a risk cannot be linked to an asset. |
