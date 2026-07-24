# 🔥 DETECTION ENGINEERING: Complete Guide for Blue Team
## Design, Validate, Measure, and Optimize Detection Rules

---

## 📖 What is Detection Engineering?

**In 30 seconds:**
Detection Engineering is the **engineering process that designs, validates, measures, and optimizes detection rules based on real attacks**. It's not just writing SIEM rules — it's an **iterative cycle of continuous improvement** that transforms logs into useful alerts through rules that balance precision (few false positives) and recall (few missed detections).

**Analogy:**
```
Writing a rule = Building a door
Detection Engineering = Designing, testing, measuring, adjusting, and improving that door

A poorly designed door:
├─ Stays open (low Recall = doesn't detect attacks)
├─ Gets stuck (high FP = too many false alarms)
└─ Needs constant maintenance (no measurements)

A well-engineered door:
├─ Opens/closes correctly (good Precision + Recall)
├─ Lasts for years (validated)
└─ Improves based on feedback (iteration)
```

---

## 🎯 Why Does a Blue Team Need Detection Engineering?

### Real-World Scenarios

| Scenario | Without Detection Engineering | With Detection Engineering |
|----------|------------------------------|--------------------------|
| New attack occurs | "How do we detect it?" | Rule designed, validated, in production in 24 hours |
| 1000 alerts/day from one rule | Analysts overwhelmed, ignore alerts | Rule tuned: 50 alerts/day, 95% precision |
| Attacker changes technique | "Our rule doesn't work anymore" | Threat hunt → new rule designed and validated |
| False positive detected | Nothing (keeps generating noise) | Investigated, rule adjusted, FP eliminated |
| Compliance (audit) | "Did we detect this attack?" | Dashboard showing: 47 detections, 45 validated, 2 FP |

### Interview Questions

1. **"What's the difference between writing a SIEM rule and Detection Engineering?"**
   - Expected: "A rule is a line of code. Detection Engineering is the process of designing it, validating it against historical logs, measuring precision/recall, tuning it, and maintaining it based on real incidents."

2. **"What's more important: detecting all attacks or minimizing false positives?"**
   - Expected: "Both. But the balance depends on context. For admin accounts = high recall (detect almost everything). For normal users = high precision (minimize FP)."

3. **"How would you validate a new rule before putting it in production?"**
   - Expected: "Against historical logs (does it detect known incidents?), measure precision/recall, see if it generates normal FP, tune, then validate again."

4. **"What's the best indicator that a rule is 'good'?"**
   - Expected: "Not just precision or recall. It's the balance (F1-Score), low volume (few alerts), and analysts think the alert is valuable."

---

## 🔍 The Concept Broken Down

### Part 1: Complete Rule Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│  PHASE 1: DISCOVERY                                         │
│  (Threat Hunter or Analyst detects new pattern)             │
├─────────────────────────────────────────────────────────────┤
│  Input: Real incident or documented attack pattern         │
│  Example: "We saw 50 connections to port 4444 from odd IP" │
│  Output: Documented need for detection                     │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 2: DESIGN                                            │
│  (Detection Engineer designs the rule)                      │
├─────────────────────────────────────────────────────────────┤
│  Questions: What logs do I need? What fields?              │
│             What values indicate attack?                   │
│             How many events = alert?                       │
│             In what timeframe?                             │
│  Output: Sigma rule or KQL rule written                    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 3: VALIDATION                                        │
│  (Test against historical logs)                             │
├─────────────────────────────────────────────────────────────┤
│  Question 1: Does it detect known historical incidents?   │
│  Question 2: Does it generate false positives?             │
│  Question 3: Are the FP "acceptable"?                      │
│  Output: TP, FP, Precision, Recall metrics                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 4: TUNING (if necessary)                            │
│  (Adjust rule based on validation)                          │
├─────────────────────────────────────────────────────────────┤
│  Option A: Low Precision? (many FP)                         │
│           → Make rule more specific                         │
│  Option B: Low Recall? (miss attacks)                       │
│           → Make rule broader                              │
│  Output: Adjusted rule, new metrics                         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 5: PRODUCTION                                        │
│  (Deploy in SIEM)                                           │
├─────────────────────────────────────────────────────────────┤
│  Actions:                                                   │
│  ├─ Deploy rule                                             │
│  ├─ Monitor alert volume (expected?)                        │
│  ├─ Create baselines (how many alerts/day is normal)       │
│  └─ Document: what it detects, why, how to respond        │
│  Output: Live rule in SIEM                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  PHASE 6: MONITORING & CONTINUOUS IMPROVEMENT              │
│  (Analysis of real alerts)                                  │
├─────────────────────────────────────────────────────────────┤
│  Weekly:                                                    │
│  ├─ Analyze genuine FP                                      │
│  ├─ Can I exclude this FP?                                 │
│  ├─ Are there attacks I'm missing?                         │
│  └─ New attack techniques to cover?                        │
│  Output: Improved rule, feedback loop closed               │
└─────────────────────────────────────────────────────────────┘
```

### Part 2: The Four Fundamental Metrics

Everything in Detection Engineering comes down to understanding these 4 metrics:

```
TP (True Positive)   = Correct alert. Real attacker detected.
FP (False Positive)  = False alert. No attack happened.
TN (True Negative)   = No alert AND no attack. ✓ Correct
FN (False Negative)  = No alert BUT attack happened. ✗ Worst error

Visualization:
                   ATTACK EXISTS    NO ATTACK
ALERT FIRES     →    TP (good)        FP (bad)
NO ALERT        →    FN (worst)       TN (good)
```

**Concrete example:**

```
Incident: 50 failed login attempts in 5 minutes, same IP

If your rule generated alert: TP (correct, was attack)
If your rule didn't generate alert: FN (worst error, missed attack)
If your rule generated alert but was admin resetting password: FP
If night passed normally without alerts: TN
```

### Part 3: The Two Metrics That Matter

From TP, FP, TN, FN derive two critical metrics:

**PRECISION** = How many of my alerts are real?
```
Precision = TP / (TP + FP)

Example:
- 100 alerts generated
- 90 were real attacks (TP)
- 10 were false (FP)
- Precision = 90 / (90 + 10) = 90%

Meaning: Of 100 alerts, 90 are real.
If precision is 50%, half are false = NOISE
```

**RECALL** = How many real attacks do I detect?
```
Recall = TP / (TP + FN)

Example:
- In last 30 days, there were 100 real attacks
- My rule detected 95 (TP)
- Missed 5 (FN)
- Recall = 95 / (95 + 5) = 95%

Meaning: I detect 95 of every 100 attacks.
If recall is 50%, I miss half = DANGEROUS
```

**Balance: F1-Score**
```
F1-Score = 2 * (Precision * Recall) / (Precision + Recall)

Target:
- Precision > 80% (don't want 80% noise)
- Recall > 90% (don't want to miss 90% of attacks)
- F1-Score > 0.85 (good balance)
```

---

## ⚙️ How to Write a Rule From Scratch

### Step 1: Understand the Attack

**Scenario:** An attacker attempts lateral access to a SQL server.

**What logs will we see?**
```
Firewall: Connection attempt to port 1433 (SQL Server)
         Source: Internal IP (workstation)
         Destination: BBDD-SERVER (database server)
         Action: BLOCKED (unauthorized)

Windows (BBDD-SERVER): EventID 4625 (Failed Login)
                      Username: attacker_creds
                      Status: 0xC0000064 (bad password)

Why important?: Failed login to database at unusual time = suspicious
```

### Step 2: Design the Rule (Sigma)

```yaml
title: Lateral Movement Attack - SQL Server Login Attempts
id: sql-lateral-movement-001
status: experimental
description: Detects multiple failed login attempts against SQL Server (port 1433)
author: Detection Engineering Team
date: 2024-01-15

logsource:
  product: firewall
  service: fortinet

# PART 1: WHAT WE'RE LOOKING FOR
detection:
  # Condition A: Attempts to SQL port (1433)
  sql_access_attempt:
    EventType: DENIED
    DestinationPort: 1433
    DestinationIP: 10.0.9.15  # BBDD-SERVER
  
  # Condition B: From workstations (not admin)
  from_workstation:
    SourceIP|startswith:
      - '10.0.100.'   # Workstation subnet
      - '10.0.101.'
  
  # Condition C: Not a known administrator
  not_admin:
    SourceIP|exclude:
      - '10.0.50.10'   # Admin workstation
      - '10.0.50.11'
  
  # PART 2: TIMEFRAME
  timeframe: 5m
  
  # PART 3: ALERT CONDITION
  condition: sql_access_attempt AND from_workstation AND not_admin

# ADDITIONAL INFORMATION
falsepositives:
  - SQL administrators accessing from unusual workstations
  - Legitimate application trying to connect to SQL

level: high
tags:
  - attack.lateral_movement
  - attack.t1570
  - database
```

### Step 3: Validate Against Historical Logs

**Question:** What were the historical lateral movement incidents?

```
Historical (last 90 days):
┌─ Incident #1 (2024-01-10):
│  └─ Attacker tried to connect to BBDD-SERVER from WKS-0055
│     → Logs: Firewall DENIED, EventID 4625 on DB
│     → Would my rule detect it? YES ✓
│
├─ Incident #2 (2023-12-28):
│  └─ Attacker attempted port 1433 connection
│     → Logs: Firewall DENIED, SQL auth failed
│     → Would my rule detect it? YES ✓
│
└─ Incident #3 (2023-12-15):
   └─ Legitimate admin from new laptop (not in whitelist)
      → Logs: Firewall DENIED (IP not authorized)
      → Would my rule detect it? YES, but it's FP ✗
```

### Step 4: Execute Rule Against Historical Logs

**Simulation (retrospective test):**

```
Historical logs: 90 days

Rule executed against all logs:
├─ TP (true positives): 2 incidents detected
├─ FP (false positives): 1 legitimate admin
├─ TN (true negatives): 259,999 normal events
└─ FN (false negatives): 0 incidents missed

Metrics:
├─ Precision: 2 / (2 + 1) = 66.6% ← LOW (too much noise)
├─ Recall: 2 / (2 + 0) = 100% ← GOOD (detects everything)
└─ F1-Score: 0.8 ← ACCEPTABLE but precision is low
```

### Step 5: Tuning (Adjust the Rule)

**Problem identified:** Low precision (66.6%), especially due to legitimate admin FP.

**Solution:** Improve legitimate admin exclusions

```yaml
# ADJUSTED VERSION

detection:
  sql_access_attempt:
    EventType: DENIED
    DestinationPort: 1433
    DestinationIP: 10.0.9.15

  from_workstation:
    SourceIP|startswith:
      - '10.0.100.'
      - '10.0.101.'

  # IMPROVED VERSION: More granular exclusion
  known_legitimate:
    SourceIP|exclude:
      - '10.0.50.10'     # Admin workstation 1
      - '10.0.50.11'     # Admin workstation 2
      - '10.0.102.105'   # CIO laptop (new)
    # Alternatively, look for "admin" in User field if available:
    # User|exclude:
    #   - '*admin*'
    #   - 'CORP\sa_*'    # Service accounts

  timeframe: 5m
  condition: sql_access_attempt AND from_workstation AND not known_legitimate

level: high
```

**Validate again:**

```
New execution against historical logs:
├─ TP: 2 incidents detected
├─ FP: 0 (admin now in exclusion)
├─ Precision: 2 / (2 + 0) = 100% ← EXCELLENT
├─ Recall: 2 / (2 + 0) = 100% ← EXCELLENT
└─ F1-Score: 1.0 ← PERFECT
```

### Step 6: Deploy to Production

```
1. Change approved by security team
2. Documentation:
   ├─ What it detects: SQL Server lateral movement attempts
   ├─ Why it matters: SQL Server contains sensitive data
   ├─ How to respond: Investigate source, block IP if external
   └─ Metrics: Precision 100%, Recall 100%
3. Deploy to SIEM
4. Monitor alerts live for 7 days
5. Adjust if necessary
```

---

## 📊 Real Rule Examples (Sigma)

### Example 1: Brute Force Against Active Directory (Windows)

```yaml
title: AD Brute Force - Multiple Failed Logins
id: ad-brute-force-001
logsource:
  product: windows
  service: security

detection:
  failed_login:
    EventID: 4625
    LogonType: 3
    Status: 'C0000064'  # Wrong password
  
  timeframe: 5m
  condition: failed_login | count(TargetUserName) by SourceIp >= 5

level: high
tags:
  - attack.credential_access
  - attack.t1110_001

# Expected metrics:
# Precision: 85% (some FP from users forgetting password)
# Recall: 95% (detects almost all brute force)
```

**Raw log that triggers this rule:**

```text
<134>Jan 15 09:14:01 DC01 MSWinEventLog 1 Security ... 4625 ...
  Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:03 DC01 MSWinEventLog 1 Security ... 4625 ...
  Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:05 DC01 MSWinEventLog 1 Security ... 4625 ...
  Account Name: jsmith Status: 0xC0000064
[+3 more in 5 minutes, same IP]
→ ALERT FIRED
```

---

### Example 2: Credential Dumper Download (Antivirus)

```yaml
title: Credential Dumping Tool Downloaded
id: cred-dump-download-001
logsource:
  product: antivirus
  service: webfilter

detection:
  suspicious_tools:
    FileName|endswith:
      - 'mimikatz.exe'
      - 'lsass_dump.exe'
      - 'hash_dumper.exe'
      - 'passwords.txt'
      - 'ntds_extract.exe'
    Action: BLOCKED  # AV detected it
  
  # Or alternatively, from Windows (download):
  # EventID: 4688  # Process creation
  # CommandLine|contains|all:
  #   - 'powershell'
  #   - 'mimikatz'

  condition: suspicious_tools

level: critical
tags:
  - attack.credential_access
  - attack.t1003
```

**Raw log:**

```text
<134>Jan 15 14:32:22 AV-SRV01 CEF:0|Symantec|Endpoint Protection|14.3|download|File Blocked|8|
  src=192.168.1.55 fname=mimikatz.exe threatLevel=critical
  act=quarantine fileHash=SHA256:9f8a...c221
```

---

### Example 3: PowerShell Encoding (Windows)

```yaml
title: Suspicious PowerShell with Encoding Parameter
id: ps-encoded-command-001
logsource:
  product: windows
  service: security

detection:
  powershell_encoded:
    EventID: 4688  # Process creation
    Image|endswith: 'powershell.exe'
    CommandLine|contains|all:
      - '-Enc'
      - '-EncodedCommand'
  
  # Exclude legitimate IT team
  not_it_team:
    User|exclude:
      - 'CORP\IT_Team*'
      - 'CORP\Admin*'
      - 'CORP\sa_*'

  condition: powershell_encoded AND not_it_team

level: medium
tags:
  - attack.execution
  - attack.t1059_001
```

**Raw log:**

```text
<134>Jan 15 14:35:15 WKS-0231 MSWinEventLog 1 Security 209981 ... 4688 ...
  User: CORP\jsmith
  Process Name: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  Command Line: powershell.exe -nop -w hidden -enc JABzAGgAZWBs...
  ParentImage: C:\Program Files\Outlook\OUTLOOK.EXE
```

**Why is it suspicious?** PowerShell + encoding typically indicates malware.

---

### Example 4: C2 Connection (Firewall)

```yaml
title: Suspicious Outbound Connection to Known C2 IP
id: c2-connection-001
logsource:
  product: firewall
  service: fortinet

detection:
  outbound_c2:
    EventType: ALLOWED  # Firewall allowed it
    DestinationIP:
      - '198.51.100.50'  # Known C2 from threat intel
      - '203.0.113.45'   # Another C2
    Protocol|contains:
      - 'TCP'
      - 'UDP'
    Action: ALLOW
  
  timeframe: 1h
  condition: outbound_c2

level: critical
tags:
  - attack.command_and_control
  - attack.t1071
```

**Raw log:**

```text
<134>Jan 15 14:40:33 FW-EDGE-01 CEF:0|Fortinet|FortiGate|7.2|traffic|Traffic Log|3|
  src=192.168.1.100 spt=52301 dst=198.51.100.50 dpt=4444 proto=tcp
  act=allow reason=policy_allow policyid=5 srcintf=internal dstintf=wan1
  duration=3650 sentbyte=2048 rcvdbyte=5120
  # ← Active connection, data being sent/received
```

---

## 🧪 Complete Validation: Before and After

### Case: Improving an Existing Rule

**ORIGINAL Rule (with problems):**

```yaml
title: Any Firewall Block
description: Alert on any traffic block
detection:
  selection:
    EventType: BLOCKED
  condition: selection
level: high
```

**Problem:** Generates 10,000+ alerts/day (nearly useless)

```
Original rule validation:
├─ TP (real attacks blocked): 15
├─ FP (legitimate traffic blocked): 9,985
├─ Precision: 15 / (15 + 9,985) = 0.15% ← TERRIBLE
├─ Recall: 15 / (15 + 0) = 100% ← Detects everything but with noise
└─ F1-Score: 0.003 ← USELESS
```

**IMPROVED Rule v1 (after tuning):**

```yaml
title: Firewall Block from External IP to Internal Server
detection:
  selection:
    EventType: BLOCKED
    SourceIP|startswith: '203.'  # External IPs only
    DestinationIP: '10.0.9.15'   # Critical DB server
    DestinationPort: 1433        # SQL Server
  condition: selection
level: high
```

**Validation:**

```
├─ TP: 8
├─ FP: 2 (ISP issues, DNS resolution attempts)
├─ Precision: 8 / (8 + 2) = 80% ← BETTER
├─ Recall: 8 / (8 + 7) = 53% ← Low, missing some
└─ F1-Score: 0.64 ← Acceptable but recall is low
```

**IMPROVED Rule v2 (after more tuning):**

```yaml
title: Firewall Block - External to Critical Resource
detection:
  selection:
    EventType: BLOCKED
    SourceIP|startswith: '203.'
    DestinationIP:
      - '10.0.9.15'   # DB Server
      - '10.0.8.50'   # File Server
      - '10.0.7.100'  # Mail Server
    DestinationPort|in:
      - 1433   # SQL
      - 445    # SMB
      - 25     # SMTP
  timeframe: 1h
  condition: selection | count() by SourceIP >= 3  # 3+ blocks from same IP
level: high
```

**Final validation:**

```
├─ TP: 12
├─ FP: 1 (partner VPN with IP overlap)
├─ Precision: 12 / (12 + 1) = 92% ← EXCELLENT
├─ Recall: 12 / (12 + 2) = 86% ← GOOD
└─ F1-Score: 0.89 ← PROFESSIONAL
```

---

## ❌ Common Detection Engineering Mistakes

### Mistake #1: Write rule without validating against history

```
❌ WRONG:
"I write the rule, deploy it, and hope for the best"

✅ RIGHT:
"I write the rule, run it against 90 days of historical logs,
calculate precision/recall, then deploy"

Reason: Without prior validation, you're shooting blind.
```

---

### Mistake #2: Confuse "low precision" with "complex rule"

```
❌ WRONG:
"My rule has low precision, I need to make it more complex"

✅ RIGHT:
"My rule has low precision (too much noise), I need to make it MORE SPECIFIC:
- Exclude known systems
- Add additional conditions
- Increase event threshold"

More complex rule ≠ better precision.
More specific rule = better precision.
```

---

### Mistake #3: Forget business context

```
❌ WRONG:
"All PowerShell = alert. All failed login = alert"

✅ RIGHT:
"Context matters:
- Who? (admin vs normal user)
- When? (3 AM vs 10 AM)
- Where? (internal IP vs external)
- Why? (expected password change?)"

Example: 5 failed logins
├─ If admin at 3 AM from external IP: CRITICAL
├─ If normal user forgetting password: IGNORE
```

---

### Mistake #4: Don't document the rule

```
❌ WRONG:
Rule created, deployed, but:
├─ No one knows what it detects exactly
├─ No response playbook
├─ Next month: "Why 100 alerts of this?"

✅ RIGHT:
Each rule should document:
├─ What it detects (clear description)
├─ Why it matters (MITRE ATT&CK)
├─ Expected Precision/Recall
├─ How to respond (playbook)
├─ When to review (if metrics change)
└─ Who maintains the rule (owner)
```

---

## 🧪 Practical Exercises

### Exercise #1: Analyze an Existing Rule

**Rule:**

```yaml
title: Suspicious Admin Account Activity
detection:
  selection:
    User|contains: 'admin'
  condition: selection
level: high
```

**Question:** What's wrong with this rule?

**Expected answer:**
```
Problems:
1) TOO BROAD - "admin" is in many users: admin, admin_user, backup_admin, etc.
2) NO TIMEFRAME - Doesn't group events in time
3) NO THRESHOLD - Alerts on ANY event from an admin (1000+ alerts/day)
4) NO CONTEXT - Doesn't differentiate normal login from privilege changes
5) Precision: Probably <5%
6) Recall: Probably 100% (but massive noise)

Improved version:
title: Admin Account Privileged Operation
detection:
  selection:
    EventID|in: [4728, 4729, 4730, 4723]  # Admin group changes, password changes
    User|contains: 'domain_admin'
  time_based:
    Hour: [22, 23, 0, 1, 2, 3, 4, 5]  # Unusual time (10 PM - 5 AM)
  condition: selection AND time_based
level: high
```

---

### Exercise #2: Design a New Rule

**Scenario:** Threat Intel reports attackers use msiexec.exe to install malware.

**Task:** Design a rule to detect this.

**Expected answer:**

```yaml
title: Suspicious msiexec.exe Execution with Network Activity
id: msiexec-suspicious-001
logsource:
  product: windows
  service: sysmon  # or Endpoint Detection

detection:
  msiexec_exec:
    EventID: 1  # Process creation
    Image|endswith: 'msiexec.exe'
    
  # Exclude legitimate installation
  not_legitimate:
    CommandLine|exclude:
      - '*software update*'
      - '*patch*'
      - '*install*'  # keywords of legitimate install
    ParentImage|exclude:
      - 'C:\Windows\System32\svchost.exe'
      - 'C:\Program Files\*'
  
  # Correlation: msiexec + network connection
  network_connection:
    EventID: 3  # Network connection
    Image|endswith: 'msiexec.exe'
    DestinationPort|exclude:
      - 80    # HTTP
      - 443   # HTTPS
    
  timeframe: 5m
  condition: msiexec_exec AND not_legitimate AND network_connection

level: high
tags:
  - attack.execution
  - attack.t1218_009

falsepositives:
  - Legitimate Windows updates installing via msiexec
  - Corporate software deployment systems
```

**Expected metrics:**
```
Precision: ~85% (some FP from updates, but manageable)
Recall: ~90% (detects most malicious msiexec)
F1-Score: 0.87 (good)
```

---

### Exercise #3: Measure an Existing Rule

**Historical data (last month):**

```
Rule: "Connections to port 4444"

Retrospective execution:
├─ Alerts generated: 120
├─ Confirmed attacks afterwards: 12
├─ Confirmed false positives: 8
├─ Undetected attacks: 3

Questions:
1) What's the Precision?
2) What's the Recall?
3) Is this a good rule?
```

**Expected answer:**

```
1) Precision = TP / (TP + FP)
   = 12 / (12 + 8)
   = 12 / 20
   = 60%
   
   Interpretation: Of 120 alerts, only 60% were real.
   40% was noise.

2) Recall = TP / (TP + FN)
   = 12 / (12 + 3)
   = 12 / 15
   = 80%
   
   Interpretation: Detected 80% of attacks.
   20% slipped through.

3) Is it good?
   - Precision of 60% is low (too much noise)
   - Recall of 80% is acceptable but could improve
   - F1-Score = 2 * (0.6 * 0.8) / (0.6 + 0.8) = 0.686
   
   Conclusion: NEEDS TUNING
   
   Option A: Increase precision (fewer FP):
   ├─ Exclude known legitimate IPs using port 4444
   ├─ Add condition: only alert if from external IP
   
   Option B: Increase recall (detect more attacks):
   ├─ Include related ports: 4445, 4446, etc.
   ├─ Add correlation with other events
```

---

## 🎯 Interview Questions by Level

### Level 1 (Entry-Level)

**Q1: "What is Detection Engineering?"**
> "The process of designing, validating, measuring, and optimizing detection rules based on real attacks. It's not just writing SIEM rules — it's an iterative cycle of continuous improvement."

**Q2: "What's the difference between a SIEM rule and Detection Engineering?"**
> "A SIEM rule is a line of code that detects a pattern. Detection Engineering is the process of validating that rule against historical logs, measuring its precision/recall, adjusting it, and keeping it up-to-date."

**Q3: "Why are false positives a problem?"**
> "Because they create noise. If 80% of my alerts are false, analysts start ignoring alerts, including real ones. FP must be minimized through tuning."

### Level 2 (Mid-Level)

**Q1: "How would you write a rule to detect Kerberoasting?"**
> "I'd look for EventID 4769 (Service Ticket requested) with RC4 encryption (easy to crack), multiple Service Tickets in short timeframe. Exclude legitimate system requests. Validate against historical logs of known incidents, measure precision/recall, and tune based on FP."

**Q2: "Which is more important: Precision or Recall?"**
> "Both. But it depends on context. For Domain Admin accounts = high Recall (detect almost everything, accept some noise). For normal users = high Precision (minimize FP, accept missing some). Ideal is balance: Precision >80%, Recall >90%."

**Q3: "How would you validate a rule before production?"**
> "1) Run rule against 90 days of historical logs. 2) Measure TP, FP, TN, FN. 3) Calculate Precision and Recall. 4) Compare with known incidents (does it detect them?). 5) If precision <80%, tune. 6) Once validated, deploy with monitoring."

### Level 3 (Senior / Detection Engineer)

**Q1: "How would you design a Detection Engineering program from scratch?"**
> "1) Threat Modeling: Who are our main adversaries? What techniques (MITRE ATT&CK)? 2) Prioritization: Which attacks are most likely and impactful? 3) For each technique: Design rule, validate against history, measure. 4) Implementation: Deploy in tiers (test → staging → production). 5) Monitoring: Analyze FP, improve rules weekly. 6) Evolution: Add new techniques based on threat intel and newly detected attacks."

**Q2: "What's the biggest challenge in Detection Engineering?"**
> "Balancing Precision and Recall. High Precision = few FP but miss attacks. High Recall = detect almost everything but lots of noise. Solution: 1) Understand business risk tolerance, 2) Iterative tuning, 3) Real incident feedback, 4) Automate tuning if possible."

**Q3: "How would you measure the success of your Detection Engineering program?"**
> "1) Rule metrics: Precision, Recall, F1-Score of each rule. 2) Operational metrics: How many attacks detected vs. missed? 3) Response time: From alert to investigation? 4) Impact: How many incidents prevented or shortened? 5) Trends: Are attack techniques evolving? Are our rules evolving with them? True success is when the org says: 'Your rules detected X attack in 5 minutes vs. 5 days manually.'"

---

## 🔗 Connections to Other Topics

- 🔥 **SIEM**: Platform where rules are deployed
- 📊 **MITRE ATT&CK**: Framework guiding which techniques to detect
- 🔐 **Active Directory**: AD logs are critical for detection engineering
- 🌐 **Windows Security Logs**: EventIDs are the basis of many rules
- 🔍 **Threat Hunting**: Discoveries become rules
- 🛡️ **Incident Response**: Each incident = opportunity for new rule
- 📈 **Threat Intelligence**: Feeds what rules to create (malware hashes, C2 IPs)

---

## 💾 TL;DR - Quick Reference

### Rule Lifecycle
```
Discovery → Design → Validation → Tuning → Production → Monitoring
                                  ↑_________________↑
                                  (feedback loop)
```

### The 4 Metrics
```
TP = Real attack detected (good)
FP = False alarm (bad, creates noise)
TN = No alert + no attack (good)
FN = Attack not detected (worst)
```

### The 2 Metrics That Matter
```
Precision = TP / (TP + FP) → % of alerts that are real (Target: >80%)
Recall = TP / (TP + FN)   → % of attacks detected (Target: >90%)
```

### Basic Rule Structure (Sigma)
```yaml
title: Descriptive name
logsource: (which logs)
detection:
  selection: (what we're looking for)
  timeframe: (time window)
  condition: (when to alert)
level: (severity)
tags: (MITRE ATT&CK)
```

### Tuning Checklist
```
Low precision? → Make more specific
  - Exclude known systems
  - Add conditions
  - Increase threshold

Low recall? → Make broader
  - Reduce threshold
  - Include attack variations
  - Correlate with other events
```

---

## 📌 Real-World Reality

### Real Case #1: Rule that saved the company

```
Rule: "EventID 4720 + 4728 + 4723 in <5 minutes"
(Account created + added to admin + password changed)

Result: Detected AD compromise in 3 minutes
Without rule: Attack would have lasted hours or days

Impact: Prevented data exfiltration
```

### Real Case #2: Rule generating too much noise

```
Before: "Any failed login = ALERT"
└─ 50,000 alerts/day, precision 2%

After: "5+ failed logins + same user + same IP + <5 min"
└─ 500 alerts/day, precision 85%

Lesson: Specificity > quantity
```

---

## 📚 Next Steps

1. **Learn Sigma** (the rule standard)
2. **Practice writing rules** against real logs
3. **Validate against history** (don't deploy without validating)
4. **Always measure** (Precision, Recall, F1-Score)
5. **Continuous iteration** (each incident = opportunity to improve)

---

## 📌 Remember

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Detection Engineering ≠ Writing rules                    │
│  Detection Engineering = Design, validate, measure,       │
│                        optimize, and maintain rules       │
│                                                            │
│  A rule without validation = An attack without prevention │
│  A rule without metrics = Driving blind                   │
│  A rule without tuning = Incomplete work                  │
│                                                            │
│  Success is when analysts say:                           │
│  "This alert is always valuable, I never ignore it"      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

**Last Updated:** 2024
**Version:** 1.0
**For:** Blue Team Students - TripleTen Bootcamp
