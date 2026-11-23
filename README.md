# SOC Analyst Practice Portfolio: Alert Triage

*Learning write-up, October 2026. The three alerts below are fictional training scenarios, not real incidents.*

## About this portfolio

I have a degree in Telecommunications (2011) and I am changing my career to cybersecurity. My goal is an entry-level SOC analyst (Tier 1) role, remote or office-based. This portfolio shows how I practice alert triage: reading an alert, collecting evidence, making a decision, and writing a clear ticket note.

I am learning step by step, so I show my reasoning and also what I corrected after feedback.

## What I am studying

- TryHackMe: Pre Security path (in progress), Introduction to Offensive Security, a Linux basics room (moving between folders, reading and searching log files with `ls`, `cd`, `cat`, `grep`), and an Introduction to SOC room (SIEM alerts)
- Professor Messer's Security+ course (social engineering and malware topics so far)
- MITRE ATT&CK matrix: tactics (the attacker's goal) and techniques (the method)
- Cyber Kill Chain: the stages of an attack
- Technical English: reading alerts and writing short ticket notes

## My triage method

For every alert I answer five questions: **What** happened, **When**, **Where** (target), **Who** (source), and **Why** (is it intended or malicious?).

Then I classify the alert as **false positive**, **benign true positive**, **true positive**, or **undetermined**, and I decide to close it or escalate it. I decide from the total evidence, not from the alert title. A scary title can be harmless, and a quiet one can be serious.

## Case 1: Multiple failed logins, then a successful login (true positive)

**Alert:** P2, 13 October 2026, 03:12. Account `ayse.k` (HR), email and VPN portal.

**Timeline**

- 03:09 to 03:11: 47 failed logins from `203.0.113.45`
- 03:12: successful login from the same IP
- 03:14: new mailbox rule that forwards all incoming mail to an external address

**Evidence**

- The source IP is in a country where the company does no business, and it is flagged in threat intelligence
- The login happens at night, but the user normally works 09:00 to 18:00 from the office
- MFA is disabled on the account
- HR mailboxes hold sensitive personal data

**Assessment:** True positive. The account is very likely compromised, and data is already leaving through the forwarding rule. The forwarding rule is the most urgent finding.

**Decision:** Escalate to Tier 2 and recommend P1.

**MITRE ATT&CK:** Brute Force (T1110), Valid Accounts (T1078), Email Forwarding Rule (T1114.003)

**Ticket note**

> Account ayse.k (HR) had 47 failed logins in under 3 minutes from 203.0.113.45 (flagged on threat intel, country outside business footprint), followed by a successful login at 03:12 and creation of a mailbox rule forwarding all incoming mail to an external address at 03:14. MFA is disabled. Likely account compromise with active data exfiltration. Escalating to Tier 2; recommend raising to P1. Suggested actions: disable account and reset password, revoke active sessions, remove the forwarding rule, block the source IP, enable MFA.

**What I learned:** My first answer only repeated the alert title and the 47 failed attempts. I missed three key facts: the login was successful, the forwarding rule was created two minutes later, and MFA was off. Now I read all the details before I decide.

## Case 2: Impossible travel (false positive)

**Alert:** P3, 13 October 2026, 09:41. Account `mehmet.d` (Sales), email and CRM application.

**Timeline**

- 09:02: login from Barcelona
- 09:40: login from Singapore (38 minutes later)

**Evidence**

- MFA was approved for both logins
- Both logins used the same company laptop
- No unusual activity after the logins
- No earlier alerts on this account
- Both IPs are clean in threat intelligence
- Asset inventory: `192.0.2.77` is the company's cloud security gateway exit point in Singapore

**Assessment:** Benign. The user works from Barcelona and his traffic leaves through the gateway, so the website sees Singapore. I checked the asset inventory before closing the alert, because I should not close an alert with missing information.

**Decision:** Close as false positive. Recommend adding the gateway IP to the rule's exclusion list so the same alert does not repeat.

**Ticket note**

> Impossible travel alert for mehmet.d: logins at 09:02 (Barcelona) and 09:40 (Singapore). Singapore IP 192.0.2.77 is the company's cloud security gateway egress node per asset inventory. Both logins used the same company laptop with MFA approved, no unusual activity afterwards, IPs clean on threat intel. Benign; closing as false positive. Recommend adding the gateway IP to the rule's exclusion list.

**What I learned:** I used the evidence together (MFA, device, history, IP reputation) and I checked the missing piece before closing. I also corrected a small mistake: I wrote the first login time wrong (09:08 instead of 09:02). Wrong times in a ticket cause problems, so I now check them twice.

## Case 3: Large upload to personal cloud storage (undetermined)

**Alert:** P3, 14 October 2026, 22:47. Account `can.y` (Finance), company laptop, VPN from the usual home IP.

**Event:** At 22:35 to 22:47, one 1.8 GB ZIP file (`Q3_closing_backup.zip`) was uploaded to a personal cloud storage service. A data loss prevention (DLP) rule flagged it.

| Supports a harmless explanation | Supports a suspicious explanation |
| --- | --- |
| No earlier alerts on the account | First upload of this kind in 90 days |
| MFA approved, normal device and IP | Company data went to a personal account |
| User often works late | DLP rule flagged the upload |
| Quarter-end close is this week, and the file name matches it | One compressed file looks like bulk data movement |
| The service is not on the block list |  |

**Missing information (not evidence either way):** whether the user is leaving the company, and whether this upload was approved.

**Assessment:** Undetermined. I cannot call it benign until I verify it.

**Decision:** Escalate with a list of checks. I would first use my own tools: DLP classification of the file, other activity before the upload (large downloads, email, USB), and the approved-apps list. Then I would ask the user's manager (not the user directly) whether the upload was approved, and check the acceptable-use policy. Any question about resignation goes through the insider-risk process, not directly to HR. This kind of alert stays confidential, and I treat the user as innocent until the facts show otherwise.

**Ticket note**

> DLP alert for can.y (Finance): a single 1.8 GB ZIP ("Q3\_closing\_backup.zip") was uploaded to a personal cloud storage service from the assigned company laptop via VPN (usual home IP), 22:35-22:47. Device, IP, and MFA are normal, and the quarter-end close may explain the file. However, this is the first such upload in 90 days and company data went to a personal account, which may violate policy. Intent undetermined. Escalating to Tier 2 / insider-risk review. Requested: confirm with user's manager whether the upload was authorized, check DLP classification of the file contents, review related activity before the upload.

**What I learned:** "Benign" and "not sure" cannot both be true. When I am not sure, the status is undetermined, and my job is to ask the right people the right questions in the right order.

## Summary of the three cases

| Case | Alert title | Evidence together | Decision |
| --- | --- | --- | --- |
| 1 | Failed logins, then success | Account compromised | True positive, escalate |
| 2 | Impossible travel | Gateway exit point | False positive, close |
| 3 | Upload to personal cloud | Mixed | Undetermined, escalate with checks |

## Next steps

- Continue the TryHackMe SOC path and practice with a real SIEM tool (Splunk or Microsoft Sentinel)
- Prepare for the CompTIA Security+ exam
- Write more triage notes and improve my technical English every week
