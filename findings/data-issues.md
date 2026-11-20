# Data issues log

Scenario: a fictional bank. All data is synthetic. Snapshot date: 22 Oct 2026.

Purpose: a list of what is wrong, missing or unclear in the data, how often, and what the report does about it.
Nothing was silently fixed. Where a value is missing, the report shows it as missing.

Note: pages 5 to 9 are not built yet. For tables used only there, "handling" is the plan.

## A. Governance records: gaps a risk analyst should report

| # | File | Issue | Count | Why it matters |
|---|---|---|---|---|
| A1 | `exception_request_details.csv` | `approver`, `business_justification`, `requester`, `technical_reviewer` are "TO FILL" | 34 of 34 rows | No exception shows who approved it or why |
| A2 | `exception_request_details.csv` | `approval_date` is blank, including for approved exceptions | 34 of 34 rows (10 are Approved) | An approval without a date or approver cannot be audited |
| A3 | `exception_request_details.csv` | `affected_asset_id` is "TO FILL" | 34 of 34 rows | Exceptions cannot be tied to a system |
| A4 | `risk_governance.csv` | No `asset_id` column | all 100 risks | Risks cannot be tied to `asset_inventory.csv` |
| A5 | `exception_request_details.csv` | `compensating_control_effectiveness` is "TO FILL" | 34 of 34 rows | Nobody records whether the compensating control works |
| A6 | `exception_request_details.csv` | `request_date` is blank for EXC-0031 | 1 row | Age of the request cannot be calculated |
| A7 | `vulnerability_findings.csv` vs `exception_request_details.csv` | Exception IDs used by findings that do not exist in the exception file | 10 of 17 IDs | Findings point to exceptions that cannot be checked |
| A8 | `risk_governance.csv` | Closed risks that still carry an exception record | 8 of 24 closed risks (RSK-00014, 21, 22, 30, 39, 56, 64, 88). 3 of them are still Pending (RSK-00021, 30, 56) | The register disagrees with itself: a closed risk should not have a pending decision |
| A9 | `risk_governance.csv` | Risk accepted although its exception was rejected | RSK-00077 | Acceptance has no valid approval |
| A10 | `risk_governance.csv` | Exception expiry date before the snapshot | 5 exceptions: EXC-0022 and EXC-0059 (Approved), EXC-0035, EXC-0069, EXC-0095 (Pending) | 3 of the 5 were never decided before they expired |
| A11 | risk_governance.csv | last_milestone has no date | all 100 risks | We cannot tell how long a risk has been at its stage, or whether a milestone was missed |
| A12 | risk_governance.csv | last_milestone repeats risk_status: Open = Plan agreed (34), In progress = Evidence collected (30), Closed = Remediated (24), Accepted = Plan agreed (12) | 100 of 100 risks | The milestone chart on Page 2 is the same split as Open against In progress. It is not an independent progress measure |
| A13 | vulnerability_findings.csv vs exception_request_details.csv | All 25 risk-accepted findings carry an exception ID, and no other finding does. 17 of the 25 point to IDs that do not exist in the exception file (10 distinct IDs, the same gap as A7). Of the 8 that exist: 3 Approved and not expired, 1 Approved but expired, 3 Pending, 1 Rejected | 25 findings | Only 3 of 25 risk-accepted findings have a valid approved exception. The other 22 are accepted without valid approval, the same governance gap as KRI-14 for risks |

Positive check: 
- for all 34 exceptions, category, severity and approval status are the same in `risk_governance.csv` and `exception_request_details.csv`.
- all 64 open risks have a remediation_due_date, so no open risk is counted as overdue because of a blank date.
- for all 360 findings, the due date equals the discovery date plus the illustrative SLA target for its severity (Critical 7, High 14, Medium 30, Low 90 days).

## B. Cleaning issues: technical problems in the raw files

| # | File | Issue | Count | Handling in the report |
|---|---|---|---|---|
| B1 | `vulnerability_findings.csv` | Duplicate `finding_id` | 361 rows, 360 unique | Inspect, then remove the duplicate |
| B2 | `vulnerability_findings.csv` | `severity` and `patch_status` in mixed case, some with extra spaces | e.g. "HIGH", " high ", "High" | Trim, clean, capitalise each word |
| B3 | `security_incidents.csv` | `severity` in mixed case, some with extra spaces | e.g. "MEDIUM", " low " | Same as B2 |
| B4 | `backup_jobs.csv` | `job_result` in mixed case, some with extra spaces | e.g. "SUCCESS", " skipped " | Same as B2 |
| B5 | `security_incidents.csv` | Blank detected, response or contained time | 5 incidents | Keep blank; exclude from time-to-detect and time-to-respond; show the sample size |
| B6 | `security_training.csv` | Completion date before assignment date | 198 rows | Flag it; do not silently change it |
| B7 | `security_training.csv` | `due_date` with and without a time | 14 rows with a time | Convert to date-time, then to date |
| B8 | `recovery_tests.csv` | Result "Pass" but integrity check "Fail" | 10 of 60 tests | Report as its own count; do not call them clean passes |
| B9 | `risk_governance.csv` | Dates with a trailing "00:00" | 6 rows | Convert with locale English (UK), then to date |
| B10 | All CSVs | Dates are text in DD-MM-YYYY | all date columns | Convert with locale English (UK) so 05-10 is not read as 10 May |
| B11 | `vulnerability_findings.csv` | `Flag` column (values Ok, Review) with no documented meaning | 6 Review, 355 Ok (all 6 Review are on Open findings) | Not used until its meaning is known |


## C. Definition questions still open

| # | Topic | Question |
|---|---|---|
| C1 | Vulnerability SLA compliance | How to treat risk-accepted findings. The report leaves them out (assumption): 35.8% (120 of 335). Counted as compliant: 40.3%. Counted as non-compliant: 33.3%. All three are Red. The static value in kri_thresholds_rag.csv (34.3) does not match any of them. |
| C2 | Overdue risk remediation (KRI-12) | Recalculated 68.8%; static value 67.2. |
| C3 | Training compliance (KRI-05) | Recalculated 85.4%; static value 85.0. |
| C4 | Suspicious authentication (KRI-04) | Which event classifications count as suspicious. |
| C5 | KRI thresholds | Proposed, not approved. |
| C6 | Control mapping | Regulation references are draft and need Compliance review. |
| C7 | Vulnerability SLA targets | Illustrative only (Critical 7, High 14, Medium 30, Low 90 days). |
| C8 | KRI-15 scope | Should "Exceptions expiring within 30 days" count only Approved exceptions? Of the 10 in the window, 3 are Rejected (EXC-0053, EXC-0024, EXC-0034), 4 are Pending and 3 are Approved. A Rejected exception is not in force, so it is unclear what "expiring" means for it. The report currently counts all 10. |
| C9 | KRI-15 window | The window (22 Oct to 21 Nov 2026, snapshot date plus 30 days) is set in the DAX measure. The rulebook does not define it. |

Static values are not used. All KRI values are recalculated from the raw data.
