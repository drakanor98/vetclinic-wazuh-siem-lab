# Findings and Remediation Plan

## Scope

This document converts the Wazuh lab output into a practical defensive-security improvement plan. The environment is a single authorised Windows 11 endpoint monitored by an all-in-one Wazuh deployment.

## Prioritisation method

Priority is based on alert severity, potential business impact, exposure, likelihood, and remediation feasibility. Scanner severity alone is not treated as proof of exploitability.

## Finding 1: Sensitive file modification

**Observation:** Wazuh rule 550 detected a checksum change to `C:\VetClinic-Lab\Patient-Records\readme.txt`. Custom rule 100100 raised the event to level 12 and mapped it to MITRE ATT&CK T1565.001.

**Risk:** Unauthorised changes to clinical or administrative records could undermine integrity, availability, regulatory compliance, and confidence in treatment histories.

**Recommended controls:**

- Limit write access to approved roles and service accounts.
- Separate ordinary user, administrator, and application identities.
- Maintain tested backups and version history.
- Require a change ticket or recorded authorisation for planned modifications.
- Escalate unexpected level-12 alerts to an analyst for user, process, and timestamp validation.
- Preserve relevant endpoint, authentication, and file-change logs during investigation.

## Finding 2: Vulnerable software inventory

**Observation:** The documented scan produced 721 findings: 9 critical, 468 high, 230 medium, and 14 low. Major contributors included Windows 11, Firefox, 7-Zip, Node.js, and Python.

**Risk:** Unpatched or unsupported software can provide initial access, privilege escalation, defence evasion, or code-execution opportunities.

**Recommended controls:**

1. Confirm installed version, CVE applicability, patch availability, and endpoint exposure.
2. Patch verified critical findings first, especially those known to be exploited or reachable through common user activity.
3. Update or remove unsupported applications.
4. Record risk acceptance for items that cannot be remediated immediately.
5. Rescan and retain before-and-after evidence.

## Finding 3: CIS configuration gaps

**Observation:** The Windows endpoint's CIS assessment showed substantially more failed than passed controls in the initial lab review.

**Risk:** Weak local policy, logging, account controls, network configuration, or security-feature settings increase attack surface and reduce investigative visibility.

**Recommended controls:**

- Export the failed-control list and group it by identity, audit, Defender, firewall, network, and local-policy themes.
- Apply low-disruption settings first.
- Test each hardening batch and document justified exceptions.
- Create a restore point or VM checkpoint before high-impact changes.
- Re-run the assessment and compare the score after remediation.

## Finding 4: Dynamic manager addressing

**Observation:** Hyper-V's Default Switch changed the Wazuh VM address after host restarts, leaving the Windows agent configured for the previous address.

**Risk:** The endpoint becomes disconnected and telemetry is lost until its manager address is corrected.

**Recommended controls:**

- For this lab, verify `hostname -I` and the agent `<address>` after every restart.
- For a long-lived environment, use a stable virtual switch, reserved DHCP address, static addressing, or internal DNS name.
- Monitor disconnected agents and define a response threshold.

## Finding 5: Alert operations

**Observation:** High-severity detection works, but the lab does not yet connect alerts to a ticketing, paging, or case-management process.

**Risk:** A valid alert can be visible without being acknowledged, investigated, or resolved promptly.

**Recommended controls:**

- Define severity-based triage targets.
- Assign alert ownership and escalation contacts.
- Document investigation steps for rule 100100.
- Integrate email, webhook, or ticketing notification in a future iteration.
- Track false positives and tune the rule without weakening coverage.

## Suggested incident workflow for rule 100100

1. Confirm the endpoint, path, timestamp, rule ID, and severity.
2. Identify the user and process associated with the modification.
3. Check whether an approved change exists.
4. Review nearby authentication, Defender, PowerShell, and process events.
5. Isolate the endpoint if malicious activity is suspected and authorised procedures permit it.
6. Preserve evidence and restore the record from a trusted version if required.
7. Document root cause, impact, actions, and follow-up controls.

