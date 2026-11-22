# Risk assessment: 6 worked examples

This shows how I move from a register entry to a judgment: what could go wrong, how bad it would be, which controls exist, and what I would do. The risk register holds only a severity, so the likelihood and impact scores for those risks are my assumptions, and each one is tagged.

Method: each risk is scored on a 1 to 5 scale for likelihood and impact (score = likelihood × impact),
the scale used in `ai_ml_risk_register.csv`. `risk_governance.csv` has only a severity, so scores for
those risks are assumptions, chosen to match that severity. Rating bands are also assumed.

Tags show where each fact comes from:
- `[filename, row ID]` = copied from that file
- `[gap: filename, column]` = the column exists but is empty or "TO FILL"
- `[assumption]` = my judgment, not in any file

Source files: `ai_ml_risk_register.csv`, `risk_governance.csv`,
`exception_request_details.csv`, `recovery_tests.csv`.

---

## 1. AIR-001: Internal GenAI assistant, data leakage

- Threat and weakness: confidential or personal data typed into prompts may be kept or exposed outside approved boundaries [ai_ml_risk_register.csv, AIR-001, risk_description]
- Impact: confidentiality [assumption]
- Inherent: likelihood 4 × impact 5 = 20, Critical [ai_ml_risk_register.csv, AIR-001, inherent_score]
- Existing controls: tenant isolation, DLP on prompts, acceptable-use policy. Effectiveness: partially effective [ai_ml_risk_register.csv, AIR-001, existing_controls, control_effectiveness]
- Control type: mostly preventive (DLP blocks, isolation separates) [assumption]
- Residual: 3 × 4 = 12, High [ai_ml_risk_register.csv, AIR-001, residual_score]
- Treatment: mitigate. Turn on prompt-level DLP and stop vendor training on tenant data. Owner06, due 30 Nov 2026, in progress [ai_ml_risk_register.csv, AIR-001, treatment, planned_action, action_due_date, status]
- Judgment: the control exists but works only in part, and the due date is 39 days after the snapshot. I would track it weekly.

## 2. AIR-002: Customer service chatbot, prompt injection

- Threat and weakness: malicious input could override instructions, expose internal data or trigger unauthorised actions [ai_ml_risk_register.csv, AIR-002, risk_description]
- Impact: confidentiality and integrity [assumption]
- Inherent: 4 × 4 = 16, High [ai_ml_risk_register.csv, AIR-002, inherent_score]
- Existing controls: input filtering, restricted tool permissions. Partially effective [ai_ml_risk_register.csv, AIR-002, existing_controls, control_effectiveness]
- Control type: preventive [assumption]
- Residual: 3 × 3 = 9, Medium [ai_ml_risk_register.csv, AIR-002, residual_score]
- Treatment: mitigate. Run a red-team test and add output guardrails. Due 15 Dec 2026, status open [ai_ml_risk_register.csv, AIR-002, planned_action, action_due_date, status]
- Judgment: the action is still open, while the chatbot is already live [ai_ml_risk_register.csv, AIR-002, lifecycle_stage = Production]. The residual score assumes the planned test happens.

## 3. RSK-00001: Unsupported software

- Risk: end-of-life software still in production. Critical, open, plan agreed. Owner02 [risk_governance.csv, RSK-00001]
- Overdue: due 2 May 2026, 173 days late at snapshot [risk_governance.csv, RSK-00001, remediation_due_date]
- Exception: none [risk_governance.csv, RSK-00001, exception_id is blank]
- Compensating control: none recorded [risk_governance.csv, RSK-00001, compensating_control is blank]
- Asset: not linked [gap: risk_governance.csv has no asset_id column]
- Threat: attacker uses known flaws that will never be patched [assumption]
- Impact: confidentiality, integrity and availability [assumption]
- Inherent: 5 × 4 = 20, Critical [assumption, chosen to match Critical in risk_governance.csv]
- Control type needed: preventive (upgrade or replace) [assumption]
- Residual today: 20, Critical, because no control is recorded [assumption]
- Treatment: mitigate. Upgrade or replace, with an owner and a date. If that is not possible, request a formal exception with a compensating control [assumption]
- Judgment: this is the longest-overdue Critical risk with no safety net at all.

## 4. RSK-00013: Recovery test gap

- Risk: recovery test not done or not passed in the required interval. Critical, open, plan agreed. Owner14 [risk_governance.csv, RSK-00013]
- Overdue: due 6 Aug 2026, 77 days late [risk_governance.csv, RSK-00013, remediation_due_date]
- Exception: none [risk_governance.csv, RSK-00013, exception_id is blank]
- Asset: not linked [gap: risk_governance.csv has no asset_id column]
- Threat: ransomware or outage where nobody knows if a restore works [assumption]
- Impact: availability and integrity [assumption]
- Inherent: 5 × 4 = 20, Critical [assumption]
- Control type: recovery testing is detective (it finds restore problems); the backup itself is corrective [assumption]
- Residual today: 20, Critical, because the test has not happened [assumption]
- Treatment: mitigate. Run and pass a recovery test, then record the result [assumption]
- Judgment: backups that were never restored do not prove recovery. Related data: 10 of 60 tests are marked Pass although the integrity check failed [recovery_tests.csv, test_result and data_integrity_check].

## 5. RSK-00059: Data handling weakness

- Risk: data handled outside required classification or encryption controls. Critical, open. Owner06 [risk_governance.csv, RSK-00059]
- Overdue: due 18 Jul 2026, 96 days late [risk_governance.csv, RSK-00059, remediation_due_date]
- Exception: EXC-0059 approved, but expired on 13 Oct 2026, 9 days before the snapshot [risk_governance.csv, RSK-00059, exception_approval_status, exception_expiry_date]
- Approver and justification: not recorded [gap: exception_request_details.csv, EXC-0059, approver and business_justification are "TO FILL"; approval_date is blank]
- Compensating control: central logging and alerting [risk_governance.csv, RSK-00059, compensating_control]
- Is the control effective? Not recorded [gap: exception_request_details.csv, EXC-0059, compensating_control_effectiveness is "TO FILL"]
- Asset: not linked [gap: exception_request_details.csv, EXC-0059, affected_asset_id is "TO FILL"]
- Threat: data stored or sent without required encryption is exposed [assumption]
- Impact: confidentiality [assumption]
- Inherent: 4 × 5 = 20, Critical [assumption]
- Control type: detective (logging finds problems, it does not stop them) [assumption]
- Residual today: 3 × 5 = 15, High [assumption]
- Treatment: mitigate. Fix the data handling, or renew the exception with a named approver and proof that the control works [assumption]
- Judgment: the approval ran out and nobody can say who approved it in the first place.

## 6. RSK-00069: Cloud logging gap

- Risk: cloud service logging not enabled or not sent centrally. Critical, open. Owner16 [risk_governance.csv, RSK-00069]
- Overdue: not yet. Due 29 Oct 2026, 7 days after the snapshot [risk_governance.csv, RSK-00069, remediation_due_date]
- Exception: EXC-0069 still pending, and its expiry date (21 Sep 2026) passed 31 days before the snapshot [risk_governance.csv, RSK-00069, exception_approval_status, exception_expiry_date]
- Approver: not recorded [gap: exception_request_details.csv, EXC-0069, approver is "TO FILL"]
- Asset: not linked [gap: exception_request_details.csv, EXC-0069, affected_asset_id is "TO FILL"]
- Threat: an attack runs unnoticed because there are no logs [assumption]
- Impact: integrity and availability, mainly through late detection [assumption]
- Inherent: 4 × 5 = 20, Critical [assumption]
- Control type needed: detective (the missing control is logging) [assumption]
- Residual today: 20, Critical. The compensating control named on the exception is "central logging and alerting" [risk_governance.csv, RSK-00069, compensating_control], which is the very control that is missing. That observation is mine.
- Treatment: mitigate. Enable and forward logs before the 29 Oct due date [assumption]
- Judgment: the compensating control named in the exception is the thing that is broken, so it is not a real safety net.

---

## What these examples show

- Four of the six risks have no asset link, so impact cannot be tied to a real system [risk_governance.csv has no asset_id; exception_request_details.csv, affected_asset_id is "TO FILL" in all 34 rows].
- Both register risks that have an exception (RSK-00059, RSK-00069) have no named approver. The same is true for all 34 exceptions [exception_request_details.csv, approver is "TO FILL" in all 34 rows].
- The two AI risks have documented controls and due dates still ahead. The two overdue register risks with no exception (RSK-00001, RSK-00013) have no recorded control at all.
