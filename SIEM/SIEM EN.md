# 🔐 SIEM: Complete Guide for Blue Team
## Security Information and Event Management

---

## 📖 What is a SIEM?

**In 30 seconds:**
A SIEM is the **central tool that collects, normalizes, correlates, and analyzes logs** from all your security sources (firewalls, antivirus, AD, Windows, etc.) to **detect alerts, suspicious patterns, and potential security incidents**.

**It's not just visualization.** It is:
- ✅ **Aggregation**: Centralizes logs from hundreds of sources
- ✅ **Normalization**: Translates different formats into one standard
- ✅ **Correlation**: Links events from multiple sources
- ✅ **Detection**: Applies rules to generate alerts
- ✅ **Response**: Provides context for investigation

---

## 🎯 Why Does a SOC/Blue Team Need a SIEM?

### In Real Work

| Scenario | Without SIEM | With SIEM |
|-----------|----------|----------|
| Attacker tries a failed login in AD, then hits a server blocked by the firewall | You see 2 separate events in 2 different tools | SIEM correlates: "Same IP in both? → Critical alert" |
| Compliance (GDPR, PCI-DSS) | Where are all the logs? Retention? Auditing? | SIEM centralizes, retains, and reports automatically |
| Incident investigation | Jumping between 5 different tools | All evidence in one place, with context |
| Proactive detection | You only see what each tool alerts on | SIEM detects patterns no single tool would see |

### Interview Questions You'll Be Asked

1. **"Why is centralizing logs important?"** → Unified visibility, correlation detection, compliance, faster investigation.
2. **"Difference between a firewall that alerts and a SIEM?"** → The firewall only sees its own traffic; the SIEM sees the full context (AD login + firewall block + antivirus threat = coordinated attack).
3. **"What happens if a log never reaches the SIEM?"** → That event is never detected, never correlated, no audit trail. It's a critical blind spot.

---

## 🔍 Breaking Down the Concept

### Part 1: Logs vs Alerts (The Fundamental Difference)

A **log** is the raw event exactly as emitted by the source. This is how it really looks inside a SIEM (e.g., a Windows event ingested as JSON):

```json
{
  "timestamp": "2024-01-15T14:30:45Z",
  "host": "DC01.corp.local",
  "log_source": "Microsoft-Windows-Security-Auditing",
  "event_id": 4624,
  "logon_type": 3,
  "target_user": "jsmith",
  "src_ip": "192.168.1.100",
  "status": "success",
  "message": "An account was successfully logged on"
}
```

An **alert** is what the SIEM generates when a rule correlates several logs like this:

```text
[ALERT] SEVERITY=CRITICAL  RULE="Multiple Failed Logins"
matched_events=50  window=5m  threshold=5
source_ip=192.168.1.100 -> dest_host=DC01.corp.local
first_seen=2024-01-15T14:25:00Z  last_seen=2024-01-15T14:30:00Z
status=OPEN  assigned=unassigned
```

**Key point:** Not every log is an alert. A log is "information." An alert is "something needs attention."

### Part 2: The Real Flow in Your SOC

```
┌──────────────────────────────────────────────────────────────┐
│                     YOUR LOG SOURCES                         │
├─────────────────┬─────────────────┬─────────────────┬────────┤
│   FIREWALL      │   ANTIVIRUS     │   ACTIVE DIR    │ WINDOWS│
│  (logs)         │   (logs)        │   (logs)        │ (logs) │
└────────┬────────┴────────┬────────┴────────┬────────┴────┬───┘
         │                 │                 │             │
         └─────────────────┴─────────────────┴─────────────┘
                           │
                     (Protocol: Syslog)
                           │
                    ┌──────▼──────┐
                    │    SIEM     │
                    ├─────────────┤
                    │ 1. RECEIVES │ Centralizes all logs
                    │ 2. NORMALIZES│ Translates to a standard format
                    │ 3. ENRICHES │ Adds context (GeoIP, threat intel)
                    │ 4. CORRELATES│ Links events across sources
                    │ 5. APPLIES RULES│ Detects suspicious patterns
                    │ 6. GENERATES ALERTS│ Creates incidents
                    └──────┬──────┘
                           │
                    ┌──────▼──────────────────┐
                    │      SOC DASHBOARD      │
                    │  ✅ Alerts by severity   │
                    │  ✅ Event timeline       │
                    │  ✅ Correlations         │
                    │  ✅ Full context         │
                    └────────────────────────┘
```

### Part 3: Rules - The Heart of the SIEM

Rules are the SIEM's **detection engine**. Without rules, a SIEM just piles up logs.

**Where do rules come from?**
- ✅ **Threat Hunters**: Discover new attack patterns
- ✅ **Incident Response**: "This happened — how do we detect it next time"
- ✅ **Playbooks**: Documented procedures turned into rules
- ✅ **Standards**: MITRE ATT&CK, CIS Controls, etc.
- ✅ **Threat Intelligence**: "This IP is malicious, block it if it shows up"

**Who implements them?**
- **Detection Engineers** or **Security Engineers** write the rule in Sigma/KQL/SPL
- **SOC Analysts** validate that it works
- **SIEM Admins** deploy them to production

---

## ⚙️ How to Write a SIEM Rule (Sigma)

### Why Sigma?

Sigma is an **open standard** for writing detection rules that works across any SIEM:
- ✅ Splunk, ELK, Microsoft Sentinel, Sumo Logic, etc.
- ✅ Portable between tools
- ✅ Easy to read and collaborate on
- ✅ Used across the industry

### Basic Structure of a Sigma Rule

```yaml
title: Detection of Multiple Failed Login Attempts
id: c91f8a6a-5c45-11eb-ae93-0242ac120002
status: experimental
description: Detects 5+ failed login attempts in 5 minutes (possible brute-force attack)
author: Blue Team SOC
date: 2024-01-15

logsource:
  product: windows
  service: security

detection:
  selection:
    EventID: 4625
    LogonType: 3
    Status: 'C0000064'
  timeframe: 5m
  condition: selection | count(TargetUserName) by SourceIp >= 5

falsepositives:
  - Legitimate users forgetting their password
  - Automated processes with expired credentials

level: medium
tags:
  - attack.credential_access
  - attack.t1110_001
```

### Line-by-Line Breakdown

| Section | What it is | Your SOC |
|---------|---------|--------|
| **title** | Descriptive rule name | "AD Brute Force Detection" |
| **id** | Unique UUID | For tracking and versioning |
| **logsource** | Where do the logs come from? | product: windows, service: security |
| **selection** | Which logs match our search | EventID: 4625 (failed login) |
| **timeframe** | Correlation time window | 5m (5 minutes) |
| **condition** | Logic: when to fire the alert | >= 5 attempts = Alert |
| **level** | Severity (low/medium/high/critical) | medium |
| **tags** | MITRE ATT&CK techniques | attack.t1110_001 |

### What the raw log that triggers this rule looks like

```text
<134>Jan 15 14:25:03 DC01 MSWinEventLog	1	Security	61452	Mon Jan 15 14:25:03 2024	4625
Security	SYSTEM	User	Failure Audit	DC01	Logon
	An account failed to log on.
	Subject: Security ID: NULL SID  Account Name: -  Logon Type: 3
	Account For Which Logon Failed: Security ID: NULL SID  Account Name: jsmith
	Failure Information: Failure Reason: Unknown user name or bad password  Status: 0xC000006D  Sub Status: 0xC0000064
	Network Information: Workstation Name: -  Source Network Address: 198.51.100.22  Source Port: 51422
```

```text
<134>Jan 15 14:25:04 DC01 MSWinEventLog 1 Security 61453 ... 4625 ... Source Network Address: 198.51.100.22 ... Account Name: jsmith
<134>Jan 15 14:25:06 DC01 MSWinEventLog 1 Security 61454 ... 4625 ... Source Network Address: 198.51.100.22 ... Account Name: jsmith
<134>Jan 15 14:25:09 DC01 MSWinEventLog 1 Security 61455 ... 4625 ... Source Network Address: 198.51.100.22 ... Account Name: jsmith
<134>Jan 15 14:25:12 DC01 MSWinEventLog 1 Security 61456 ... 4625 ... Source Network Address: 198.51.100.22 ... Account Name: jsmith
```

→ 5 identical logs, same IP, same user, within less than 5 minutes → **the condition is met → the alert fires.**

### Real Example #1: Firewall Access Attempt from a Blocked IP

```yaml
title: Firewall Block + Subsequent Access Attempt
id: fw-detect-001
logsource:
  product: firewall
  service: fortinet

detection:
  blocked_traffic:
    EventType: BLOCKED
    DestinationIP: 10.0.0.0/8
  suspicious_source:
    SourceIP|filter: threat_intel_db
  timeframe: 10m
  condition: blocked_traffic AND suspicious_source

level: high
tags:
  - attack.reconnaissance
  - attack.t1592
```

**Raw log (CEF, exactly as a FortiGate emits it):**

```text
<134>Jan 15 14:32:10 FW-EDGE-01 CEF:0|Fortinet|FortiGate|7.2|traffic|Traffic Log|3|
  src=203.0.113.45 spt=54211 dst=10.0.5.20 dpt=445 proto=tcp
  act=deny reason=policy_deny policyid=12 srcintf=wan1 dstintf=internal
  threatintel_match=true threatintel_source=corp_ti_feed
```

### Real Example #2: Active Directory Detection

```yaml
title: Multiple Password Changes - Possible Compromise
id: ad-detect-002
logsource:
  product: windows
  service: security

detection:
  password_change:
    EventID: 4723
  admin_user:
    TargetUserName|contains:
      - 'admin'
      - 'domain admin'
      - 'service account'
  timeframe: 1h
  condition: password_change AND admin_user

level: critical
tags:
  - attack.persistence
  - attack.t1098
```

**Raw log:**

```text
<131>Jan 15 03:14:02 DC01 MSWinEventLog 1 Security 88231 Mon Jan 15 03:14:02 2024 4723
Security SYSTEM User Success Audit DC01 User Account Management
	An attempt was made to change an account's password.
	Subject: Security ID: CORP\svc_backup  Account Name: svc_backup  Logon ID: 0x3E7
	Target Account: Security ID: CORP\svc_backup  Account Name: svc_backup  Account Domain: CORP
```

### Real Example #3: Antivirus + Windows Correlation

```yaml
title: Antivirus Threat + Subsequent Windows Process Creation
id: av-detect-003
logsource:
  product: antivirus,windows
  service: defense

detection:
  av_threat:
    ThreatDetected: true
    ThreatLevel: high
  suspicious_process:
    EventID: 4688
    CommandLine|contains:
      - 'powershell'
      - 'cmd.exe'
      - 'whoami'
  timeframe: 5m
  condition: av_threat AND suspicious_process by SourceIP

level: critical
tags:
  - attack.execution
  - attack.t1059
```

**Correlated raw logs (antivirus + Windows):**

```text
<131>Jan 15 14:33:02 AV-SRV01 CEF:0|Symantec|Endpoint Protection|14.3|threat|Threat Detected|8|
  src=192.168.1.55 fname=invoice.exe threatName=Trojan.GenericKD.45123
  threatLevel=high act=quarantine cs1=SHA256:9f8a...c221
```

```text
<131>Jan 15 14:33:41 WKS-0055 MSWinEventLog 1 Security 209981 Mon Jan 15 14:33:41 2024 4688
Security SYSTEM User Success Audit WKS-0055 Process Creation
	A new process has been created.
	Creator Subject: Security ID: CORP\jdoe  Account Name: jdoe
	Process Information: New Process Name: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
	CommandLine: powershell.exe -nop -w hidden -enc SQBFAFgAIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQA...
```

---

## 🔗 Mapping to MITRE ATT&CK (For Reporting)

### Why Map?

MITRE ATT&CK is a **reference standard** that:
- ✅ **Communicates** what type of attack you detected (at executive/compliance level)
- ✅ **Categorizes** attack techniques in your report
- ✅ **Enables comparison** with other SOCs
- ✅ **Supports compliance**: "Did we detect T1110?" → Yes, 5 times

### MITRE ATT&CK Structure

```
attack.tactic → attack.technique → attack.sub_technique

Example:
attack.credential_access → attack.t1110 → attack.t1110_001
           (Tactic)               (Technique)      (Sub-technique)
           What for?          How do they do it?   Exact method?
```

### Main Tactics

| Tactic | Description | Examples |
|---------|------------|----------|
| **reconnaissance** | Gathering information before the attack | Port scanning, enumerating users |
| **resource_development** | Building attack infrastructure | Buying fake domains, setting up C2 |
| **initial_access** | First point of entry | Phishing, exploit, compromised credentials |
| **execution** | Running malicious code | PowerShell, Command, Macros |
| **persistence** | Maintaining lasting access | Creating accounts, installing backdoors |
| **privilege_escalation** | Gaining higher permissions | UAC bypass, kernel exploit |
| **defense_evasion** | Avoiding detection | Obfuscation, disabling antivirus |
| **credential_access** | Stealing credentials | Brute force, keylogger, dumping |
| **discovery** | Gathering system/network info | Enumerating shares, listing processes |
| **lateral_movement** | Moving through the network | Pass-the-hash, RDP, file sharing |
| **collection** | Gathering data of interest | Screen capture, email scraping |
| **command_and_control** | Communicating with the attacker | C2 channels, DNS tunneling |
| **exfiltration** | Stealing data | Upload to an external server, email |
| **impact** | Causing damage | Deleting data, ransomware, DDoS |

### How to Map Your Rule to MITRE

**Step 1: Identify WHAT your rule detects**
```
My rule detects: "5+ failed logins in 5 min from the same IP"
What is this? = An attempt to guess a password
```

**Step 2: Look up the technique in MITRE**
```
Go to https://attack.mitre.org
Search: "Brute Force"
Find: T1110 - Brute Force
  └─ T1110_001 - Password Guessing
  └─ T1110_002 - Password Spraying
  └─ T1110_003 - Password Cracking
  └─ T1110_004 - Credential Stuffing
```

**Step 3: Choose the correct sub-technique**
```
Does my rule detect multiple attempts of the SAME password against DIFFERENT users?
→ T1110_002 (Password Spraying)

Does my rule detect multiple attempts of DIFFERENT passwords against the SAME user?
→ T1110_001 (Password Guessing)
```

**Step 4: Add it to your rule**
```yaml
tags:
  - attack.credential_access
  - attack.t1110
  - attack.t1110_001
```

### Quick Mapping: Examples From Your SOC

| What you detect | Tactic | Technique | Sub-technique |
|-----------------|---------|---------|------------|
| Firewall blocks a connection to a malicious IP | reconnaissance | T1592 | Gather Victim Host Info |
| Multiple failed login attempts | credential_access | T1110 | T1110_001 (Password Guessing) |
| PowerShell runs a suspicious command | execution | T1059 | T1059_001 (PowerShell) |
| Admin user changes password at 3 AM | persistence | T1098 | T1098_002 (Account Manipulation) |
| Executable file downloaded from email | initial_access | T1566 | T1566_001 (Phishing - Attachment) |
| New service created on Windows | persistence | T1543 | T1543_003 (Windows Service) |
| User accesses a share from an external IP | lateral_movement | T1570 | Lateral Tool Transfer |
| SAM credential dumping | credential_access | T1003 | T1003_002 (LSASS Memory) |

### How to Report (Example)

```text
INCIDENT #2024-001215
Severity: HIGH
Type: Credential Access Attack

Description:
47 failed login attempts were detected in 3 minutes against user admin@corp.com
from IP 203.0.113.45 (location: China per GeoIP)

MITRE ATT&CK Mapping:
├─ Tactic: credential_access
├─ Technique: T1110 - Brute Force
└─ Sub-technique: T1110_001 - Password Guessing

Detection Sources:
✅ Rule: "Multiple Failed Logins" (SIEM)
✅ EventID: 4625 (Windows Security Log)
✅ Source: Active Directory

Actions Taken:
1. IP blocked at the firewall level
2. Account monitored for further changes
3. Password reset for the affected user
4. Escalated to the IR Team
```

---

## 🚨 Your Methodology: Alert Response

### The Process: Evaluate → Investigate → Decide

```
┌─────────────────────────────────────────────────────────┐
│           ALERT ARRIVES AT THE SOC (Dashboard)          │
├─────────────────────────────────────────────────────────┤
│ STEP 1: EVALUATE                                         │
│  └─ Severity: LOW / MEDIUM / HIGH / CRITICAL            │
│  └─ Context: Legitimate user? Normal hours?             │
│  └─ Source: Where does it come from? Reliable?          │
│  └─ Time: When did it happen?                           │
├─────────────────────────────────────────────────────────┤
│ STEP 2: INVESTIGATE (if it needs attention)              │
│  └─ Any similar previous events?                         │
│  └─ Do other systems confirm it?                         │
│  └─ Is there a legitimate explanation?                    │
├─────────────────────────────────────────────────────────┤
│ STEP 3: DECIDE                                            │
│  ├─ ✅ INVESTIGATE: Tier 1/2 analysis + documentation    │
│  ├─ ⬆️ ESCALATE: Tier 3 / Incident Response Team        │
│  └─ ✋ CLOSE: False Positive (document why)              │
├─────────────────────────────────────────────────────────┤
│ STEP 4: ACT                                               │
│  └─ Gather evidence, contain, eradicate, verify          │
│  └─ Document everything in case of an audit              │
└─────────────────────────────────────────────────────────┘
```

### Severity: When to Investigate vs Close?

| Severity | Examples | Action |
|-----------|----------|--------|
| **LOW** | 1 failed login, allowed share access, closed port | Review in weekly report, likely close |
| **MEDIUM** | 5+ failed logins in 10 min, permission change, blocked executable | Investigate within 2-4 hours, correlate with events |
| **HIGH** | Multiple failed logins in 2 min + subsequent successful access, credential dumper download | Investigate NOW, escalate if confirmed |
| **CRITICAL** | Admin account modified, malicious service created, ransomware detected, AD dump | Escalate IMMEDIATELY to IR, start response |

### Example: Investigating a MEDIUM Alert

**Alert exactly as it appears on the SIEM dashboard:**

```text
[ALERT #4471] SEVERITY=MEDIUM  RULE="Suspicious PowerShell Execution"
event_id=4688  host=WKS-0231  time=2024-01-15T14:32:00Z
user=jsmith@corp.com
command_line="powershell.exe -Enc JABzAGgAZQBs..."
status=NEW
```

```
STEP 1: EVALUATE
✓ Severity: MEDIUM (encoded PowerShell can be legitimate or malicious)
✓ User: jsmith - known developer (LOWER RISK)
✓ Time: 14:32 during business hours (LEGITIMATE)
✓ Source: Workstation, not a server (LOWER RISK)

STEP 2: INVESTIGATE
? Are there prior events of this user running PowerShell?
  → Check the last 7 days: YES, ran something similar 2 weeks ago (LEGITIMATE)

? Do other systems see suspicious activity from this user?
  → Check firewall: NO anomalies
  → Check antivirus: NO threats

? What does that base64 command do?
  → Decoded: "Get-Process | Export-CSV report.csv" (LEGITIMATE - a report)

STEP 3: DECIDE
✅ CLOSE - False Positive
Reason: Documented user, similar historical activity, harmless command
```

**Closure documented in the SIEM:**

```text
[ALERT #4471] STATUS=CLOSED  disposition=FALSE_POSITIVE
closed_by=analyst_maria  closed_at=2024-01-15T15:10:00Z
notes="Known developer, decoded command = Get-Process export CSV. Recommend whitelisting WKS-0231/jsmith."
```

---

## ❌ Common Mistakes Analysts Make

### Mistake #1: Confusing "Alert" with "Threat"

```
❌ WRONG:
"We have 200 alerts today, must be a major attack"

✅ RIGHT:
"We have 200 alerts, but 195 are false positives (users forgetting passwords).
Only 5 need investigation. Of those, 3 are legitimate, 2 are potential threats."

Lesson: An alert ≠ an attack. You need context.
```

### Mistake #2: No Baseline of Normal Activity

```
❌ WRONG:
"User downloaded 500 MB, ALERT!"
Reality: They're a data engineer who downloads 500 MB DAILY.

✅ RIGHT:
Baseline: User normally downloads 400-600 MB/day
Anomaly: If they download >2GB OR outside business hours, THEN alert

Lesson: You need to know what's "normal" for that user/system.
```

### Mistake #3: Overly Broad Rules (Noise)

```
❌ WRONG:
Rule: "Anyone who runs PowerShell = Alert"
Result: 1000+ alerts/day from legitimate users
Analysts overwhelmed, start ignoring alerts

✅ RIGHT:
Rule: "User runs PowerShell WITH the -enc (encoding) parameter
       AND the IP is NOT in the developer whitelist"
Result: 5-10 alerts/day, mostly legitimate but still worth checking

Lesson: Specificity > breadth
```

### Mistake #4: Not Documenting False Positives

```
❌ WRONG:
Same analyst, same false positive, 10 times a month
Always closed without documenting why

✅ RIGHT:
First false positive: "User X downloaded a PDF from email"
You document: "Vendor ABC sends automatic PDFs, known false positive"
Later: Add to a whitelist or tune the rule
Result: Less noise, less unnecessary investigation

Lesson: FPs are opportunities to improve rules.
```

---

## 🧪 Practical Exercises

### Exercise #1: Write Your First Rule

**Scenario (raw firewall log):**

```text
<134>Jan 15 09:14:22 FW-EDGE-01 CEF:0|Fortinet|FortiGate|7.2|traffic|Traffic Log|3|
  src=198.51.100.9 spt=51044 dst=10.0.9.15 dpt=22 proto=tcp
  act=deny reason=policy_deny policyid=44 srcintf=wan1 dstintf=db_segment
```

```
Task: Write a Sigma rule to detect this (external SSH to an internal DB server).

Expected answer:
- logsource: firewall
- selection: DestinationPort=22 AND DestinationIP=internal_db AND SourceIP=external
- condition: selection
- level: high
- tags: attack.lateral_movement, attack.t1570
```

### Exercise #2: Classify Events

```
Event 1: Admin logs in at 3 AM from an IP in China
What severity? What action?
→ CRITICAL/HIGH - Investigate NOW
   Is the user traveling? On call? Context is key.

Event 2: Developer runs PowerShell on a workstation during business hours
What severity? What action?
→ LOW/MEDIUM - Review context
   Is it documented? Do other developers do this? Likely close.

Event 3: 3 failed login attempts in 30 minutes (normal user)
What severity? What action?
→ LOW - Close
   Within normal range, user probably forgot their password.
```

### Exercise #3: Map to MITRE

```
Scenario: Your SIEM detects that a user changed their account password
and then logged in with the new password from a new IP (never seen before).

Task: Map this to MITRE ATT&CK

Expected answer:
- Action: Password change + access from a new IP
- Which technique? → T1098 (Account Manipulation) or T1078 (Valid Accounts)
- Why? → Attacker compromises account, changes password, maintains access
- Report: "Account Persistence Detection (T1098) on user jsmith"
```

---

## 🎯 Interview Questions You'll Be Asked

### Level 1 (Entry-Level SOC Analyst)

**Q1: "What is a SIEM?"**
> "A tool that centralizes logs from multiple sources, normalizes them, and generates rule-based alerts to detect security incidents."

**Q2: "Difference between a log and an alert?"**
> "A log is an event that happened (raw information). An alert is when a SIEM rule detects that a log (or combination of logs) represents suspicious or malicious behavior."

**Q3: "Why is centralizing logs important?"**
> "Because it lets you detect patterns no individual tool would see. Example: a failed login (AD) + a blocked server access attempt (firewall) = a coordinated attack the SIEM correlates."

### Level 2 (Mid-Level SOC Analyst / Detection Engineer)

**Q1: "How would you write a rule to detect brute force in AD?"**
> "Look for EventID 4625 (failed login) with the same user across multiple IPs, or the same IP across multiple users, within a short timeframe (5 min). If >= 5 failed attempts, fire an alert. Map it to MITRE T1110 (Brute Force)."

**Q2: "How would you handle a false positive in your SIEM?"**
> "First document why it's a false positive. Then consider: exclude the user/IP? adjust the threshold? add a filter? Then measure whether it improves without losing real detections."

**Q3: "How would you correlate events from multiple sources?"**
> "Using common fields: SourceIP, TargetUser, Timestamp. Example: a failed login (AD) + a malicious file (antivirus) + a blocked connection (firewall) within <5 min, same IP → a correlated HIGH alert."

### Level 3 (Senior Analyst / Detection Engineer)

**Q1: "How would you design a detection program from scratch?"**
> "1) Map threats and adversaries. 2) Use MITRE ATT&CK to document techniques. 3) Create Sigma rules per technique. 4) Validate against historical logs. 5) Tune to reduce false positives. 6) Measure TP/FP/FN. 7) Iterate."

**Q2: "How would you evaluate a rule's effectiveness?"**
> "Precision = TP/(TP+FP). Recall = TP/(TP+FN). F1-Score = average of both. Targets: Precision >80%, Recall >90%. If precision is low, make the rule more specific; if recall is low, make it broader."

**Q3: "How would you explain MITRE ATT&CK to an executive?"**
> "It's a shared language for talking about attacks: 'We detected 15 T1110 (Brute Force) attempts this month, blocked 14 (93%), 1 succeeded and was escalated to IR. That's up 87% from last month, so I recommend patching the exposed service.'"

---

## 💾 TL;DR - Quick Reference

### What is a SIEM?
A tool that centralizes logs → normalizes → correlates → generates alerts → provides context for investigation.

### Key Components
- **Logs**: Raw events from firewalls, AD, Windows, antivirus
- **Alerts**: Logs that match SIEM rules
- **Rules**: Logic (written in Sigma) that defines what's suspicious
- **Correlation**: Linking events across multiple sources
- **MITRE ATT&CK**: Standard classification of attack techniques

### Analysis Process
1. **Evaluate**: Severity, context, time, user, source
2. **Investigate**: Look for prior events, correlations, legitimate explanations
3. **Decide**: Investigate further, escalate to IR, or close as FP
4. **Act**: Contain, eradicate, document

### Basic Rule Structure

```yaml
logsource: (product and service)
detection:
  selection: (what we're looking for)
  timeframe: (time window)
  condition: (when to alert)
level: (severity)
tags: (MITRE ATT&CK)
```

### MITRE Mapping Quick Reference
- **Tactic**: What for? (credential_access, persistence, etc.)
- **Technique**: How? (T1110 - Brute Force)
- **Sub-technique**: Exact method? (T1110_001 - Password Guessing)

### Important Metrics
- **False Positives** (FP): False alerts → noise → needs tuning
- **True Positives** (TP): Correct alerts → real detections
- **Precision**: What % of my alerts are real? (Target: >80%)
- **Recall**: What % of attacks do I catch? (Target: >90%)

---

## 📊 Rule Tuning: The Art of Balance

### The Balance Problem

```
TOO SPECIFIC (Few alerts)             TOO BROAD (Many alerts)
├─ HIGH Precision                     ├─ LOW Precision (80% noise)
├─ LOW Recall (misses attacks)       ├─ HIGH Recall (catches almost everything)
├─ Examples:                          ├─ Examples:
│  • Failed login = 10 in 1 min       │  • PowerShell = always alert
│  • PowerShell + rare param + IP     │  • Any password change
│                                     └─ 1000+ alerts/day = useless
└─ Result: 5 alerts/day = OK
```

### How to Tune a Rule

**Step 1: Measure baseline**
```
"My brute-force rule generates 50 alerts/day"
How many are TP (true positives)? = 3
How many are FP (false positives)? = 47
Precision: 3/(3+47) = 6% ← TERRIBLE
```

**Step 2: Investigate the FPs**
```
Of the 47 FPs:
- 30: Users forgetting their password (EXPECTED)
- 12: Automated processes with expired credentials (EXPECTED)
- 5: Users on variable IPs (EXPECTED - remote workers)
```

**Step 3: Adjust**
```
Option A - Raise the threshold:
  Before: >= 5 attempts → After: >= 15 attempts
  Result: 20 alerts/day, 3 TP (better ratio)

Option B - Exclude known users:
  Add a whitelist for service accounts and automated processes

Option C - Change the timeframe:
  Before: 5 minutes → After: 1 minute (more restrictive)

Option D - Add conditionals:
  If failed login AND (IP never seen before AND outside business hours)
  Result: Only alert on real anomalies
```

**Step 4: Re-measure**
```
After changes:
New precision: 12 / (12+8) = 60% ← BETTER
New alerts/day: 20 (vs 50 before)
Analysts not overwhelmed ✓
Real attacks detected ✓
```

### Tuning Checklist

```
Does your rule generate too many alerts?
☐ Raise the threshold (5 events → 10 events)
☐ Reduce the timeframe (5 min → 1 min)
☐ Add extra conditions
☐ Exclude known users/IPs
☐ Require multiple sources (not just 1)

Does your rule miss attacks (low recall)?
☐ Lower the threshold
☐ Increase the timeframe
☐ Remove unnecessary conditions
☐ Add variations of the behavior
☐ Include different event types

Not sure what to do?
☐ Analyze the last 100 TP and 100 FP
☐ What differentiates TP from FP?
☐ Add that difference as a condition
```

---

## 🔗 Connections to Other Topics

- 🛡️ **Detection Engineering**: SIEM rules are the foundation of any detection program
- 📊 **MITRE ATT&CK Framework**: How to classify and report detected attacks
- 🔍 **Incident Response**: SIEM generates the alerts the IR team investigates
- 🔐 **Active Directory Security**: AD logs are critical in a SIEM for detecting compromise
- 🪟 **Windows Security Logs**: EventIDs (4625, 4688, etc.) are key sources
- 🔥 **Firewall & IDS/IPS**: Network logs feed SIEM detection
- 📈 **Threat Hunting**: Threat hunter discoveries become SIEM rules

---

## 📚 Reflection Questions (To Go Deeper)

1. **"What happens if a log source (e.g., AD) isn't connected to the SIEM?"** → A critical blind spot — you won't detect attacks that only appear in those logs.
2. **"How do you know if a rule is 'good'?"** → High TP rate, low FP rate.
3. **"Who decides whether an alert gets escalated to IR?"** → The Tier 1/2 analyst, based on severity, context, and availability.
4. **"How do you document an incident detected by the SIEM?"** → Alert + Timeline + Correlations + MITRE mapping + Actions taken.
5. **"What's more important: detecting attacks or avoiding false alarms?"** → BOTH are critical. Balance is the art of tuning.

---

## 🎓 Next Steps

1. Practice writing Sigma rules for your SOC's use cases
2. Map your alerts to MITRE ATT&CK in your reports
3. Analyze your false positives and tune your rules
4. Document everything — this will become your detection engineer portfolio

---

## 📌 Remember

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│  A SIEM isn't magic. It's logic + data + rules.       │
│                                                        │
│  Good logs → Normalized → Specific rules              │
│  ↓                                                     │
│  Useful alerts → Fast investigation → Response         │
│                                                        │
│  Your job is to make that chain flawless.              │
│                                                        │
└────────────────────────────────────────────────────────┘
```

---

**Last updated**: 2024
**Version**: 1.0
**For**: Blue Team Students - TripleTen Bootcamp
