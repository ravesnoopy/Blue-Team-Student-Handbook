# 🎯 MITRE ATT&CK Framework: Complete Blue Team Mastery Guide

---

## 📖 What Is This?

MITRE ATT&CK (Adversarial Tactics, Techniques, and Common Knowledge) is the **globally-recognized, empirical knowledge base of adversary tactics and techniques** based on real-world breach data. It's not theoretical—it's based on actual attacks analyzed by security researchers. Think of it as a **universal map** where every attack ever documented is categorized into 14 tactic phases, each containing multiple techniques that show HOW attackers really work. For a blue team analyst, MITRE ATT&CK is your **Rosetta Stone**—it translates raw Windows Event Logs and network traffic into a standardized language that helps you understand attack patterns, prioritize threats, and build detection rules.

---

## 🎯 Why Does A SOC/Blue Team Professional Need This?

### **Real Job Scenarios**

1. **Incident Investigation: "What Attack Pattern Is This?"**
   - You investigate a breach: Event 4688 (PowerShell), Event 7045 (service), Event 5140 (network access)
   - Without MITRE: "Looks bad, but where do I focus?"
   - With MITRE: "This maps to Execution (T1059) → Persistence (T1543) → Lateral Movement (T1021). I know the attack chain."
   - You can now predict the attacker's next move

2. **Threat Hunting: "What Should I Search For?"**
   - Your SOC has 500 Windows machines generating 1 million events per hour
   - Without MITRE: Random searches, hope you find something
   - With MITRE: "We saw credential theft (T1110), so now hunt for Lateral Movement (T1021). Check for Kerberos tickets (T1558), RDP connections (T1021.001), and Pass-the-Hash (T1550.002)."
   - You hunt with purpose and find hidden breaches

3. **Detection Engineering: "What Should I Alert On?"**
   - Your team needs to build SIEM rules
   - Without MITRE: Build rules for individual events (noisy, many false positives)
   - With MITRE: "Map Techniques to Tactics. Build rules that detect technique combinations (not just single events). Alert when you see Execution + Persistence + Lateral Movement in sequence."
   - Your detections catch real attacks, not noise

4. **Threat Intelligence: "Who's Attacking Us?"**
   - Analyst says: "We saw these 12 techniques used in the attack"
   - With MITRE: You check which APT groups use those same 12 techniques
   - Example: If you see T1021.001 (RDP), T1110 (brute force), T1005 (data collection), you might be looking at Wizard Spider or FIN7
   - You now know the attacker's playbook and can predict their next moves

### **Interview Questions You'll Face**

- *"Walk me through how you'd map a Windows Event Log to MITRE ATT&CK techniques."*
- *"Explain the difference between a Tactic, a Technique, and a Procedure in MITRE."*
- *"You discover an attack using 7 different MITRE techniques. How would you prioritize which one to investigate first?"*
- *"Name 3 techniques used in lateral movement and how you'd detect each one."*
- *"How would you use MITRE to attribute an attack to a specific threat actor group?"*
- *"Design a detection rule for the Execution tactic that catches most PowerShell-based attacks but minimizes false positives."*

---

## 🔍 The Concept Broken Down

### **Part 1: The Foundation — What Are Tactic, Technique, and Procedure?**

MITRE has three layers that build on each other:

#### **Layer 1: TACTIC (What is the goal?)**

**Definition:** The tactical goal an adversary is trying to achieve at each phase of their attack.

**There are 14 Tactics in MITRE ATT&CK:**

| # | Tactic | Goal | When It Happens |
|---|--------|------|-----------------|
| 1 | **Reconnaissance** | Gather information about target | BEFORE breach |
| 2 | **Resource Development** | Build attack infrastructure | BEFORE breach |
| 3 | **Initial Access** | Get into the system | FIRST step |
| 4 | **Execution** | Run malware/tools | DURING breach |
| 5 | **Persistence** | Stay in the system | AFTER access |
| 6 | **Privilege Escalation** | Get admin rights | DURING breach |
| 7 | **Defense Evasion** | Hide from security tools | THROUGHOUT breach |
| 8 | **Credential Access** | Steal passwords/tokens | DURING breach |
| 9 | **Discovery** | Map network/systems | DURING breach |
| 10 | **Lateral Movement** | Move to other machines | AFTER gaining access |
| 11 | **Collection** | Gather target data | BEFORE exfil |
| 12 | **Command & Control** | Communicate with malware | THROUGHOUT breach |
| 13 | **Exfiltration** | Steal data out | NEAR end |
| 14 | **Impact** | Damage/destroy systems | FINAL step |

**Key insight:** Attackers don't always use all 14—they pick the ones they need. A ransomware gang might skip Collection and go straight to Impact. A spy might skip Impact and focus on Collection.

---

#### **Layer 2: TECHNIQUE (How is the tactic achieved?)**

**Definition:** The specific method used to accomplish a tactic.

**Example: Execution Tactic (T04) has many techniques:**

```
Tactic: EXECUTION (Execute code)
├─ Technique T1059: Command and Scripting Interpreter
│  └─ HOW: Use built-in interpreters (PowerShell, cmd, bash)
│
├─ Technique T1106: Native API
│  └─ HOW: Call Windows APIs directly (bypasses cmd/PowerShell)
│
├─ Technique T1053: Scheduled Task/Job
│  └─ HOW: Create scheduled task that runs malware at boot
│
├─ Technique T1609: Container Administration Command
│  └─ HOW: If cloud: use container APIs
│
└─ Technique T1559: Inter-Process Communication
   └─ HOW: Malware talks to another process to execute code
```

Each technique has multiple **sub-techniques** (the variant):

```
T1059: Command and Scripting Interpreter
├─ T1059.001: PowerShell
├─ T1059.003: Windows Command Shell (cmd.exe)
├─ T1059.005: Visual Basic
├─ T1059.007: JavaScript
└─ T1059.008: Unix Shell
```

**Why it matters for SOC:** 
- Same tactic (Execution), multiple techniques (T1059, T1106, T1053)
- Same technique (T1059), multiple sub-techniques (PowerShell vs cmd)
- A detection rule for T1059.001 (PowerShell) won't catch T1059.003 (cmd.exe)

---

#### **Layer 3: PROCEDURE (How does a specific group do it?)**

**Definition:** The actual implementation by a real-world threat actor group (APT).

**Example: APT-28 (Fancy Bear) executing T1059.001 (PowerShell):**

```
Group: APT-28 (Russia, Guccifer 2.0, attributed to GRU)
Tactic: Execution
Technique: T1059.001 (PowerShell)
Procedure: APT-28 uses Base64-encoded PowerShell commands to:
  1. Download additional tools
  2. Execute reconnaissance commands (Get-AdUser, Get-ADComputer)
  3. Establish command & control

Evidence:
- Used in 2016 DNC breach
- Used in 2020 SolarWinds supply chain attack
- Malware: Sofacy, NotPetya
```

**Why it matters for SOC:**
- You can say: "This attack pattern matches APT-28's known procedure"
- You can predict: "If this is APT-28, they'll probably try lateral movement next using Kerberos (they always do)"
- You can prepare: "APT-28 targets political campaigns and critical infrastructure—adjust defense posture"

---

### **Part 2: The 14 Tactics Explained (Your Mental Model)**

#### **BEFORE BREACH (Tactics 1-2)**

**Tactic 1: RECONNAISSANCE**
- Attacker gathers information about you before attacking
- Examples: Check your website, scan your IP range, find employees on LinkedIn
- SOC relevance: You won't see these in logs (happens externally)
- But intelligence teams track this for early warning

**Tactic 2: RESOURCE DEVELOPMENT**
- Attacker builds tools and infrastructure
- Examples: Register malicious domain, set up botnet, buy zero-day exploit
- SOC relevance: Firewall blocks malicious domains, DNS blocks C&C servers
- Prevention: Work with threat intel to identify malicious domains early

---

#### **INITIAL BREACH (Tactic 3)**

**Tactic 3: INITIAL ACCESS**
- Attacker enters the system for the first time
- Top Techniques:
  - **T1566: Phishing** (email with malicious link/attachment)
  - **T1133: External Remote Services** (RDP exposed to internet, attacker brute forces)
  - **T1199: Trusted Relationship** (Attack partner's vendor, use that access to reach you)
  - **T1190: Exploit Public-Facing Application** (Website vulnerability)
  - **T1195: Supply Chain Compromise** (Attack software supply chain)

**SOC Detection:**
```
If T1566 (Phishing):
  → Event 4688: User clicks link, malware.exe launches
  → Event 5156: Outbound connection to attacker server

If T1133 (RDP Brute Force):
  → Event 4625: 200 failed RDP logins in 10 minutes
  → Event 4624: Successful RDP from external IP
```

**Your job:** Detect and block at THIS tactic if possible. Everything else follows.

---

#### **EXECUTION PHASE (Tactic 4)**

**Tactic 4: EXECUTION**
- Attacker runs code/malware on the system
- Top Techniques:
  - **T1059: Command and Scripting Interpreter** (PowerShell, cmd, bash)
  - **T1106: Native API** (Call Windows APIs directly—harder to detect)
  - **T1053: Scheduled Task/Job** (Create task to run malware later)
  - **T1648: Serverless Execution** (Cloud: AWS Lambda, Azure Functions)
  - **T1204: User Execution** (User double-clicks malware—social engineering)

**SOC Detection:**
```
Event 4688: Process Creation
Image: powershell.exe
CommandLine: powershell.exe -enc JABzAGM= [ENCODED COMMAND]
Parent: explorer.exe

→ Maps to T1059.001 (PowerShell)
```

**Critical insight:** Event 4688 is your PRIMARY source for detecting Execution techniques. If you master Event 4688 + MITRE mapping, you catch 70% of attacks at this phase.

---

#### **PERSISTENCE PHASE (Tactic 5)**

**Tactic 5: PERSISTENCE**
- Attacker ensures they stay in system even after reboot
- Top Techniques:
  - **T1547: Boot or Logon Autostart Execution** (Registry keys, startup folders)
  - **T1543: Create or Modify System Process** (Malicious service, driver)
  - **T1053: Scheduled Task/Job** (Task runs at boot or on schedule)
  - **T1136: Create Account** (Create backdoor user account)
  - **T1137: Office Application Startup** (Malicious macro in Office)

**SOC Detection:**
```
Event 7045: Service Installed
ServiceName: WindowsUpdate (legitimate name, malicious binary)
ImagePath: C:\ProgramData\malware.exe

→ Maps to T1543.003 (Windows Service)

Event 4698: Scheduled Task Created
TaskName: \Microsoft\Windows\System Restore\SystemRestore
Command: C:\malware.exe

→ Maps to T1053.005 (Scheduled Task)

Event 4720: User Account Created
NewAccountName: svc_admin (backdoor account)

→ Maps to T1136.001 (Local Account)
```

**Your job:** If you miss Initial Access, catch them here. They MUST create persistence or lose access on reboot.

---

#### **PRIVILEGE ESCALATION (Tactic 6)**

**Tactic 6: PRIVILEGE ESCALATION**
- Attacker goes from user → admin or local admin → domain admin
- Top Techniques:
  - **T1548: Abuse Elevation Control Mechanism** (Bypass UAC)
  - **T1134: Access Token Manipulation** (Token theft, token impersonation)
  - **T1547: Boot or Logon Autostart Execution** (Modify boot process)
  - **T1542: Pre-OS Boot** (Modify BIOS/firmware—rare but devastating)
  - **T1611: Escape to Host** (Escape from container to host OS)

**SOC Detection:**
```
Event 4688: Process Creation
Image: C:\temp\uacbypass.exe
ParentImage: explorer.exe
IntegrityLevel: Medium → High (escalation!)

→ Maps to T1548 (UAC Bypass)

Event 4720: User Account Created
NewAccountName: admin_backup
Comment: "(empty)" or suspicious
GroupMembership: Administrators

→ Maps to T1136 (Create Privileged Account)
```

**Why it matters:** Once attacker has admin, they can:
- Read all files
- Install persistence
- Create backdoor accounts
- Move laterally to all machines in domain

---

#### **DEFENSE EVASION (Tactic 7)**

**Tactic 7: DEFENSE EVASION**
- Attacker hides their tracks from security tools
- Top Techniques:
  - **T1027: Obfuscation or Encoding** (Encode PowerShell, hide strings)
  - **T1562: Impair Defenses** (Disable Windows Defender, Firewall, logging)
  - **T1070: Indicator Removal** (Delete logs, clear history)
  - **T1036: Masquerading** (Malware named svchost.exe, looks legitimate)
  - **T1556: Modify Authentication Process** (Bypass MFA, install fake authenticator)

**SOC Detection:**
```
Event 1102: Audit Log Cleared
ClearedBy: svc_admin
Time: 2024-01-15 03:05:00

→ IMMEDIATE ESCALATION
→ Maps to T1070.001 (Clear Windows Event Logs)

Event 4104: PowerShell Script Block Logging
ScriptBlockText: Set-MpPreference -DisableRealtimeMonitoring $true

→ Maps to T1562.001 (Disable or Modify Tools)

Event 4688: Process Creation
Image: powershell.exe
CommandLine: powershell.exe -enc JABzAGM= [ENCODED]

→ Maps to T1027.010 (Command Obfuscation)
```

**Critical:** T1070 (Indicator Removal) is the BIGGEST red flag. If you see Event 1102 (log cleared), assume active breach and escalate immediately.

---

#### **CREDENTIAL ACCESS (Tactic 8)**

**Tactic 8: CREDENTIAL ACCESS**
- Attacker steals passwords, tokens, or hash credentials
- Top Techniques:
  - **T1110: Brute Force** (Guess passwords offline or online)
  - **T1056: Input Capture** (Keylogger, screenshot tool)
  - **T1187: Forced Authentication** (Trick you into giving credentials)
  - **T1111: Multi-Stage Channels** (Credential phishing from fake login page)
  - **T1621: Multi-Factor Authentication Request Generation** (MFA fatigue attack)

**SOC Detection:**
```
Event 4625: Failed Logon (repeated)
TargetUserName: admin
FailureReason: Invalid password
Count: 200 in 10 minutes

→ Maps to T1110 (Brute Force)

Event 4688: Process Creation
Image: mimikatz.exe (credential dumper tool)

→ Maps to T1003 (OS Credential Dumping)

Event 5140: Network Share Access
ShareName: \\server\C$ (admin share access = suspicious)

→ Maps to T1550.002 (Pass-the-Hash)
```

**Why it matters:** Once attacker has credentials, Lateral Movement becomes easy.

---

#### **DISCOVERY (Tactic 9)**

**Tactic 9: DISCOVERY**
- Attacker maps the network, finds systems, identifies targets
- Top Techniques:
  - **T1087: Account Discovery** (Enumerate user accounts: Get-AdUser)
  - **T1010: Enumerate Local and Network Users** (net user, whoami)
  - **T1580: Cloud Infrastructure Discovery** (Find cloud resources)
  - **T1538: Cloud Service Discovery** (List buckets, databases)
  - **T1217: Browser Bookmarks** (Find internal tools/URLs)

**SOC Detection:**
```
Event 4688: Process Creation (Reconnaissance Tools)
Image: powershell.exe
CommandLine: Get-AdUser -Filter * | Select Name

→ Maps to T1087 (Account Discovery)

Event 4688: Process Creation
CommandLine: net view \\domaincontroller

→ Maps to T1010 (Enumerate Network Resources)

Event 5140: Network Share Access
ShareName: \\server\C$, \\server\Admin$
IntegrityLevel: Access from admin account

→ Maps to T1135 (Network Share Discovery)
```

**Why it matters:** Discovery tells you what data exists and where. Attackers ALWAYS do this before stealing.

---

#### **LATERAL MOVEMENT (Tactic 10)**

**Tactic 10: LATERAL MOVEMENT**
- Attacker moves from one machine to another in the network
- Top Techniques:
  - **T1021: Remote Services** (RDP, SSH, WinRM)
  - **T1550: Use Alternate Authentication Material** (Pass-the-Hash, Pass-the-Ticket, token theft)
  - **T1570: Lateral Tool Transfer** (Copy tools between machines)
  - **T1570: Lateral Tool Transfer** (Tools spread laterally)
  - **T1091: Replication Through Removable Media** (USB worm)

**SOC Detection:**
```
Event 4625: Failed Logon (external IP)
+ Event 4624: Successful Logon (same external IP)
SourceIp: 10.0.1.50 (internal machine—attacker lateral moving)

→ Maps to T1021.001 (RDP)

Event 4769: Kerberos Service Ticket Requested
ServiceName: (unusual service account)
UserName: admin

→ Maps to T1550.003 (Pass-the-Ticket / Kerberoasting)

Event 5140: Network Share Access
ShareName: \\other_server\Finance
SourceMachine: compromised_endpoint

→ Maps to T1570 (Lateral Movement via Network Share)
```

**Why it matters:** This is WHERE breaches spread. Stop lateral movement and you contain the breach.

---

#### **COLLECTION (Tactic 11)**

**Tactic 11: COLLECTION**
- Attacker gathers target data (what they came for)
- Top Techniques:
  - **T1123: Audio Capture** (Record conversations)
  - **T1119: Automated Exfiltration** (Setup automatic data collection)
  - **T1115: Clipboard Data** (Steal clipboard contents)
  - **T1005: Data from Local System** (Copy files from disk)
  - **T1114: Email Collection** (Export mailbox, steal emails)

**SOC Detection:**
```
Event 5140: Network Share Access
ShareName: \\server\Finance, \\server\HR, \\server\Payroll
AccessMask: Read
SourceMachine: compromised_endpoint
TimeOfDay: 03:00 AM (off-hours)

→ Maps to T1005 (Data from Local System)

Firewall Log: Large Outbound Connection
DestinationIP: attacker_server
BytesTransferred: 5 GB
Protocol: FTP/HTTPS

→ Not Collection itself, but EVIDENCE of collection happened
```

**Why it matters:** This is the attacker's objective. They're stealing your crown jewels here.

---

#### **COMMAND & CONTROL (Tactic 12)**

**Tactic 12: COMMAND & CONTROL**
- Attacker communicates with malware to give it commands
- Top Techniques:
  - **T1071: Application Layer Protocol** (HTTP/HTTPS C&C)
  - **T1008: Fallback Channels** (If primary C&C is down, use backup)
  - **T1001: Data Obfuscation** (Encrypt C&C traffic)
  - **T1095: Non-Application Layer Protocol** (Custom protocol, not HTTP)
  - **T1571: Non-Standard Port** (C&C on unusual ports)

**SOC Detection:**
```
Event 5156: Network Connection
DestinationIP: suspicious_IP
DestinationPort: 8080 (unusual)
Protocol: HTTP
ProcessImage: malware.exe
Frequency: Every 5 minutes (beacon pattern)

→ Maps to T1071.001 (Application Layer Protocol - HTTP)

DNS Log:
Query: malware.attacker.com
Frequency: Every 5 minutes
SourceIP: compromised_endpoint

→ Maps to T1071.004 (DNS)
```

**Why it matters:** C&C traffic is evidence the malware is ACTIVE. Stop this, and you've neutered the attack.

---

#### **EXFILTRATION (Tactic 13)**

**Tactic 13: EXFILTRATION**
- Attacker transfers stolen data outside the network
- Top Techniques:
  - **T1048: Exfiltration Over Alternative Protocol** (Not HTTP: FTP, SFTP, DNS)
  - **T1041: Exfiltration Over C2 Channel** (Use existing C&C connection)
  - **T1020: Automated Exfiltration** (Malware automatically sends data)
  - **T1030: Data Transfer Size Limits** (Chunk large files into smaller pieces)
  - **T1537: Transfer Data to Cloud Account** (Cloud storage: AWS S3, Azure Blob)

**SOC Detection:**
```
Firewall Log: Large Outbound Transfer
Source: compromised_endpoint
Destination: attacker_server OR cloud_storage
Direction: Outbound
Volume: 10 GB+
Protocol: HTTPS/FTP

→ Maps to T1041 or T1048

DNS Log: DNS tunneling attempt
Query: [large_data_encoded].attacker.com
Frequency: Continuous

→ Maps to T1048.003 (Exfiltration Over Unencrypted/Obfuscated Non-C2 Protocol)
```

**Why it matters:** If data leaves the network, the breach is COMPLETE. Stop earlier phases to prevent this.

---

#### **IMPACT (Tactic 14)**

**Tactic 14: IMPACT**
- Attacker damages, destroys, or disrupts systems
- Top Techniques:
  - **T1486: Encrypt Sensitive Data** (Ransomware)
  - **T1561: Disk Wipe** (Destroy data on disk)
  - **T1491: Defacement** (Change website appearance)
  - **T1531: Account Access Removal** (Lock users out of accounts)
  - **T1529: Service Stop** (Shut down critical services)

**SOC Detection:**
```
Event 4688: Process Creation
Image: ransomware.exe
CommandLine: ransomware.exe /encrypt /key:stolen_key

→ Maps to T1486 (Encrypt Sensitive Data)

File System Changes: Files renamed to .encrypted
Created: ransom_note.txt
All files in C:\Users\* modified

Event 5145: Network Share Object Access
ShareName: \\server\*
AccessType: Write (changing files)

→ Maps to T1561 (Disk Wipe) or T1486 (Encryption)
```

**Why it matters:** If you reach this tactic, containment failed. Prevention should happen MUCH earlier.

---

### **Part 3: Mapping Windows Events to MITRE Tactics**

**This is the CRITICAL skill that makes you a top analyst:**

Every Windows Event can (and should) be mapped to a MITRE Tactic and Technique.

```
COMPLETE ATTACK TIMELINE WITH MITRE MAPPING:

TIME | EVENT | WINDOWS ID | TACTIC | TECHNIQUE | ACTION
-----|-------|------------|--------|-----------|--------
23:15| RDP brute force starts | 4625 | Initial Access | T1133 | ALERT: Block IP
23:47| 200 failed logins | 4625 x200 | Initial Access | T1110 | ESCALATE
00:05| RDP success | 4624 | Initial Access | T1133 ✓ | ISOLATE MACHINE
00:12| Attacker runs cmd | 4688 | Execution | T1059.003 | TRACK PROCESS
00:15| PowerShell encoded | 4688 | Execution+Evasion | T1059.001+T1027 | KILL PROCESS
00:18| Service installed | 7045 | Persistence | T1543.003 | DELETE SERVICE
00:20| Backdoor account | 4720 | Persistence | T1136.001 | DISABLE ACCOUNT
00:25| Admin share access | 5140 | Lateral Movement | T1021.002 | BLOCK ACCESS
03:00| Ransomware runs | 4688 | Execution+Impact | T1486 | CONTAINMENT FAIL
03:05| Logs cleared | 1102 | Defense Evasion | T1070.001 | ASSUME BREACH
```

---

## ⚙️ What You MUST Memorize

### **The 14 Tactics (In Attack Order)**

**Mnemonic: "RRIEPD ECLCI"** (sounds like "Reap-ed Eckly")

Actually, better mnemonic in **SPANISH/ENGLISH mix**:

**"RAPI-PED-ECCE"**
- **R**econnaissance
- **A**cceso (Initial Access)
- **P**ersistencia
- **I**nfiltración (Execution in Spanish = "ejecución", but think "infiltrate and execute")
---
- **P**rivilegio (Privilege Escalation = "escalada de privilegios")
- **E**vasión (Defense Evasion)
- **D**escubrimiento (Discovery)
---
- **E**ncuentra (Lateral Movement = "encuentra" = find/encounter other systems)
- **C**redenciales (Credential Access)
- **C**omando (Command & Control)
- **E**xfiltración
- **I**mpacto (Impact)

---

**Better: Sequential Order Mnemonic:**

```
FIRST PHASE (Before Breach):
1. Reconnaissance (👁️ Spy)
2. Resource Development (🔧 Prepare)

SECOND PHASE (Getting In):
3. Initial Access (🚪 Enter)
4. Execution (▶️ Run)
5. Persistence (🔒 Stay)

THIRD PHASE (Going Deep):
6. Privilege Escalation (📈 Go Up)
7. Defense Evasion (🎭 Hide)
8. Credential Access (🔑 Steal Keys)
9. Discovery (🗺️ Map)
10. Lateral Movement (🚶 Move)

FINAL PHASE (Getting Out):
11. Collection (🎁 Gather)
12. Command & Control (📞 Control)
13. Exfiltration (📤 Export)
14. Impact (💥 Damage)
```

---

### **Top 30 Techniques You MUST Know**

These are the techniques you'll see in 90% of real breaches:

| Tactic | Technique ID | Technique Name | Common Tool | Windows Event |
|--------|--------------|----------------|-------------|---------------|
| Initial Access | T1566 | Phishing | Email attachment | 4688 (malware runs) |
| Initial Access | T1133 | External Remote Services | RDP brute force | 4625→4624 |
| Execution | T1059.001 | PowerShell | Encoded scripts | 4688 |
| Execution | T1059.003 | Cmd.exe | System commands | 4688 |
| Execution | T1106 | Native API | Windows API calls | 4688 (indirect) |
| Execution | T1053.005 | Scheduled Task | Task Scheduler | 4698 |
| Persistence | T1547 | Boot or Logon Autostart | Registry + .lnk | 4104 (Registry) |
| Persistence | T1543.003 | Windows Service | Malicious svc | 7045 |
| Persistence | T1053.005 | Scheduled Task | Task runs at boot | 4698 |
| Persistence | T1136.001 | Create Local Account | Backdoor user | 4720 |
| Privilege Escalation | T1548 | Abuse Elevation Control | UAC bypass | 4688 |
| Privilege Escalation | T1134 | Access Token Manipulation | Token theft | 4688 (indirect) |
| Defense Evasion | T1027 | Obfuscation | Encode commands | 4104 (Script block) |
| Defense Evasion | T1562.001 | Disable Defender | Registry change | 4104 |
| Defense Evasion | T1070.001 | Clear Windows Event Logs | Log deletion | 1102 |
| Credential Access | T1110 | Brute Force | Password guessing | 4625 (failed logins) |
| Credential Access | T1003 | OS Credential Dumping | Mimikatz | 4688 (mimikatz.exe) |
| Credential Access | T1556 | Modify Authentication | Disable MFA | 4104 (Policy change) |
| Discovery | T1087 | Account Discovery | Get-ADUser | 4688 (PowerShell) |
| Discovery | T1010 | Enumerate Local Users | whoami, net user | 4688 |
| Discovery | T1135 | Network Share Discovery | net view, 135 port | 4688 |
| Lateral Movement | T1021.001 | RDP | Windows RDP | 4625→4624 (remote) |
| Lateral Movement | T1021.006 | Windows Admin Shares | C$, Admin$, IPC$ | 5140 |
| Lateral Movement | T1550.002 | Pass-the-Hash | PtH tools | 4624 (unusual account) |
| Lateral Movement | T1558.003 | Kerberoasting | hashcat, crack | 4769 (unusual tickets) |
| Collection | T1005 | Data from Local System | Copy files | 5140 (share access) |
| Collection | T1114 | Email Collection | Export mailbox | 5140 (mailbox access) |
| Command & Control | T1071.001 | HTTP | Beacon over HTTP | 5156 (network) |
| Exfiltration | T1041 | Exfiltration Over C2 | Steal via C&C | 5156 (large transfer) |
| Impact | T1486 | Encrypt Sensitive Data | Ransomware | 4688 (exe runs) |

---

## 📚 What You MUST Understand

### **Understanding 1: Not All Attacks Use All 14 Tactics**

Different attacker types use different tactics:

```
RANSOMWARE GANG (FIN7):
Reconnaissance → Resource Dev → Initial Access → Execution →
Persistence → Privilege Escalation → Credential Access →
Lateral Movement → Collection → C&C → Exfiltration → Impact
(All 14, because they want maximum damage)

SPY (APT-28/Russia):
Reconnaissance → Resource Dev → Initial Access → Execution →
Persistence → Privilege Escalation → Discovery → Credential Access →
Lateral Movement → Collection → C&C
(Skip Exfiltration+Impact—they don't want to be noticed)

OPPORTUNIST (Script Kiddie):
Initial Access → Execution → Impact
(Minimal tactics—just wants to cause chaos fast)

CRYPTOCURRENCY MINER:
Initial Access → Execution → Persistence → C&C
(Skip Lateral Movement—don't need to move, just CPU)
```

**For SOC:** Know which tactics to prioritize based on threat actor profile.

---

### **Understanding 2: MITRE Tactic Chains Show Attack Progression**

A single attack typically follows this pattern:

```
Initial Access (How did they get in?)
         ↓
    Execution (What did they run?)
         ↓
    Persistence (How did they stay?)
         ↓
    Privilege Escalation (Did they go up?)
         ↓
    Discovery/Lateral Movement (Did they explore/move?)
         ↓
    Collection (What did they steal?)
         ↓
    Exfiltration (Did they get it out?)
         ↓
    Impact (Did they cause damage?)
```

**For detection:** If you stop at Step 1, no breach. If you stop at Step 3, contained. If you reach Step 7, you've lost.

---

### **Understanding 3: Defense Evasion is Continuous**

Defense Evasion (T07) is NOT a single step—it's **throughout the entire attack**:

```
Reconnaissance (T01)
└─ Defense Evasion: VPN, spoofed identity
    
Initial Access (T03)
└─ Defense Evasion: Phishing looks legitimate
    
Execution (T04)
└─ Defense Evasion: Encode PowerShell
    
Persistence (T05)
└─ Defense Evasion: Legitimate service name (WindowsUpdate)
    
Lateral Movement (T10)
└─ Defense Evasion: Use legitimate admin tools (PsExec, RDP)
    
Collection (T11)
└─ Defense Evasion: Access during off-hours, use legitimate tools
```

**For detection:** Every tactic can have Defense Evasion mixed in. A "legitimate" PowerShell command might be malicious.

---

### **Understanding 4: Procedures Show Group Patterns**

Different APT groups have signatures:

```
APT-28 (Russia/GRU):
- Always uses T1059.001 (PowerShell with obfuscation)
- Always uses T1550.003 (Pass-the-Ticket for lateral movement)
- Signature: Base64 encoding + Get-AdUser reconnaissance

Lazarus (North Korea):
- Always uses custom malware (not off-the-shelf)
- Signature: T1059 with custom scripting language
- Always includes destructive component (T1561 Disk Wipe)

FIN7 (Unknown/Financial):
- Always uses T1021.001 (RDP for persistence)
- Uses legitimate tools (LOLBins)
- Signature: Multi-stage infection (phishing → downloader → payload)
```

**For threat hunting:** If you see APT-28's signature techniques, you might be targeting APT-28. Adjust hunting strategy.

---

## 🚨 Practical Application: Real Attack Investigation

### **Scenario: You Get Alert at 2 PM Friday**

**Alert:** "Suspicious PowerShell execution detected on MARKETING-PC-05"

**Raw Events:**

```
14:22 - Event 4688: powershell.exe execution
        CommandLine: powershell.exe -enc JABkAG8AdwBuAGwAbw...
        Parent: explorer.exe
        
14:23 - Event 5156: Network connection
        DestinationIP: 192.0.2.100
        DestinationPort: 8080
        Process: powershell.exe
        
14:24 - Event 7045: Service installed
        ServiceName: WindowsDefenderService (fake name!)
        ImagePath: C:\ProgramData\windowsdefender.exe
        
14:25 - Event 5140: Network share access
        ShareName: \\FINANCE-SERVER\Payroll
        UserName: marketing_user
        
14:26 - Event 4625: Failed RDP attempt
        TargetServer: FINANCE-SERVER
        TargetUserName: admin
        SourceIP: 192.168.1.50 (internal—attacker lateral moving)
```

---

### **Your Analysis with MITRE:**

```
EVENT → MITRE TACTIC → MITRE TECHNIQUE → SEVERITY → ACTION

14:22 PowerShell
└─ EXECUTION (T04)
   └─ T1059.001 (PowerShell)
   └─ Severity: HIGH (code execution)
   └─ Action: Investigate where command came from

14:23 Network connection
└─ COMMAND & CONTROL (T12)
   └─ T1071.001 (HTTP)
   └─ Severity: CRITICAL (malware communicating home)
   └─ Action: Block IP 192.0.2.100, check if malware running

14:24 Service installed
└─ PERSISTENCE (T05)
   └─ T1543.003 (Windows Service)
   └─ Severity: CRITICAL (attacker staying in system)
   └─ Action: Delete service immediately

14:25 Network share access
└─ COLLECTION (T11)
   └─ T1005 (Data from Local System)
   └─ Severity: CRITICAL (targeting sensitive payroll data)
   └─ Action: Check if data was copied out

14:26 Failed RDP attempt
└─ LATERAL MOVEMENT (T10)
   └─ T1021.001 (RDP)
   └─ Severity: CRITICAL (attempting to spread)
   └─ Action: Check FINANCE-SERVER logs for compromise
```

---

## ❌ Common Mistakes Students Make

### **Mistake 1: Memorizing Technique Numbers Instead of Technique Names**

**Wrong:**
> "This event maps to T1059"

**Correct:**
> "This PowerShell execution maps to T1059 (Command and Scripting Interpreter), specifically T1059.001 (PowerShell). The attacker is trying to download a second-stage payload."

**Why it matters:** 
- Technique number is just a reference ID
- Technique NAME tells you WHAT happened
- Sub-technique tells you the VARIANT
- Description tells you WHY it matters

---

### **Mistake 2: Thinking Each Event Maps to Only ONE Tactic**

**Wrong:**
```
Event 4688 PowerShell = EXECUTION only
```

**Correct:**
```
Event 4688 PowerShell -enc JAB...
├─ EXECUTION (T1059.001) — it's running PowerShell
├─ DEFENSE EVASION (T1027) — it's encoded (obfuscated)
├─ LATERAL MOVEMENT (T1021) — if run remotely
└─ DISCOVERY (T1087) — if running Get-ADUser

One event, multiple tactics!
```

**Why it matters:**
- Single-tactic thinking = missing the full attack picture
- Multi-tactic analysis = you understand the attacker's strategy

---

### **Mistake 3: Not Using Procedures to Attribute Attacks**

**Wrong:**
> "This attack used PowerShell and brute force, so it could be anyone"

**Correct:**
> "This attack used:
> - T1059.001 with Base64 encoding (APT-28 signature)
> - T1550.003 Pass-the-Ticket (APT-28 preferred)
> - Reconnaissance via Get-ADUser (APT-28 typical)
> This matches APT-28's documented procedures. Probability: 80%"

**Why it matters:**
- Knowing the attacker's identity helps predict next moves
- Attribution informs threat level and response priority
- Ransomware gang ≠ nation-state ≠ lone hacker (different responses)

---

### **Mistake 4: Only Looking at "Attack" Techniques, Missing Normal Operations**

**Wrong:**
> "PowerShell execution is always suspicious"
> (Alert on all Event 4688 PowerShell)
> (Result: 10,000 false positives per day)

**Correct:**
> "PowerShell execution by IT admins at 9 AM = normal
> PowerShell execution by marketing user at 2 AM with encoded command = suspicious
> PowerShell execution with obfuscation (-enc argument) = always suspicious"

**Why it matters:**
- Context matters (who, when, what, where)
- Not all technique usage is malicious
- Baseline normal behavior first, then alert on deviations

---

## 🧪 Practice Scenario: Map This Attack to MITRE

**Scenario:** You investigate a breach that occurred over 2 days.

**Timeline:**

```
Day 1, 14:00 - Attacker sends phishing email to 50 users
Day 1, 14:15 - One user clicks link, malware downloads
Day 1, 14:30 - Malware establishes persistence (installs service)
Day 1, 15:00 - Malware connects to attacker server (C&C)
Day 1, 16:00 - Attacker disables Windows Defender
Day 1, 17:00 - Attacker creates backdoor account
Day 1, 20:00 - Attacker dumps credentials from LSASS
Day 2, 08:00 - Attacker moves laterally to database server
Day 2, 09:00 - Attacker accesses financial database
Day 2, 10:00 - Attacker exports 1 GB of data
Day 2, 11:00 - Attacker deletes Event Logs to hide tracks
```

---

### **Your Task:**

Map EACH event to:
1. **MITRE Tactic** (number)
2. **MITRE Technique** (ID + name)
3. **Severity** (Low/Medium/High/Critical)
4. **Detection Point** (If using Windows logs, which Event ID?)

**Example format:**

```
Day 1, 14:00 - Phishing Email
├─ Tactic: #3 Initial Access
├─ Technique: T1566 (Phishing)
├─ Severity: HIGH
└─ Detection: Email gateway logs (not Windows Event)
```

**Can you try mapping the rest?** (I'll provide solutions after)

---

## 🎯 Interview Questions You Might Get

### **Easy Questions (L1)**

**Q: "Explain the difference between a MITRE Tactic and Technique."**

**Expected Answer:**
> "A Tactic is the strategic goal (e.g., Execution = run code), while a Technique is the specific method to achieve that goal (e.g., T1059 PowerShell). Multiple Techniques can achieve the same Tactic."

---

**Q: "Name 3 techniques used in Lateral Movement."**

**Expected Answer:**
> "T1021.001 Remote Services (RDP), T1550.002 Pass-the-Hash (using stolen hashes to authenticate), and T1558.003 Kerberoasting (stealing and cracking Kerberos tickets)."

---

### **Medium Questions (L2)**

**Q: "You see Event 4688 showing PowerShell execution with -enc argument. Map this to MITRE and explain why it's suspicious."**

**Expected Answer:**
> "This maps to:
> - Tactic: Execution (T04), Technique: T1059.001 (PowerShell)
> - Also Defense Evasion (T07), Technique: T1027 (Obfuscation)
> 
> It's suspicious because:
> 1. Legitimate PowerShell scripts aren't typically Base64 encoded
> 2. Encoding indicates the attacker is hiding the command contents
> 3. The -enc flag is almost NEVER used in normal administration
> 4. This is a classic malware execution pattern"

---

**Q: "You're hunting for Pass-the-Hash attacks. What MITRE Technique is this, what Events would you look for, and what would indicate success?"**

**Expected Answer:**
> "This is T1550.002 (Use Alternate Authentication Material - Pass-the-Hash), which is part of Lateral Movement (T10).
>
> What I'd look for:
> - Event 4625: Unusual failed logon attempts with strange source IPs
> - Event 4624: Successful logon without corresponding logon script (offline use)
> - Event 4688: Credential dumping tools (mimikatz, hashdump)
> - Event 5140: Unusual network share access from low-privilege user
>
> Indicators of success:
> - Attacker accessing shares with admin hash
> - Lateral movement to domain controller
> - Escalation to Domain Admin account"

---

### **Hard Questions (L3)**

**Q: "Design a detection rule for a ransomware attack using MITRE. Start from Initial Access, chain through to Impact, and explain what IOCs you'd alert on and what false positives you'd expect."**

**Expected Answer:**
> "Ransomware detection strategy using MITRE chain:
>
> Stage 1: INITIAL ACCESS (T1566 - Phishing)
> - Alert: Email with suspicious attachment or link
> - IOC: Malicious domain, suspicious file hash
> - False Positive: Legitimate business emails with PDFs
> - Mitigation: Email gateway scanning, user training
>
> Stage 2: EXECUTION (T1059.003 - cmd.exe, T1204 - User Execution)
> - Alert: Email attachment launches malware
> - IOC: Unknown executable in temp directory
> - False Positive: Legitimate software installers
> - Mitigation: Sandboxing, behavioral analysis
>
> Stage 3: PERSISTENCE (T1547 - Autostart, T1543 - Service)
> - Alert: New service installed with unusual name
> - IOC: Service pointing to temp or AppData directory
> - False Positive: Legitimate software updates
> - Mitigation: Whitelist known services, alert on new ones
>
> Stage 4: DEFENSE EVASION (T1562 - Disable Defenses)
> - Alert: Windows Defender disabled
> - IOC: Event 1102 (log clearing), Registry modification
> - False Positive: Intentional admin management
> - Mitigation: Alert on after-hours changes, require approval
>
> Stage 5: DISCOVERY (T1087 - Account Discovery)
> - Alert: Get-ADUser, net user commands
> - IOC: PowerShell reconnaissance commands
> - False Positive: Legitimate admin scripts
> - Mitigation: Baseline normal admin activity, alert on anomalies
>
> Stage 6: LATERAL MOVEMENT (T1021.001 - RDP)
> - Alert: Unusual RDP connections between machines
> - IOC: Lateral RDP to file servers from endpoints
> - False Positive: Legitimate IT administration
> - Mitigation: Restrict RDP with EDR, monitor unusual patterns
>
> Stage 7: COLLECTION (T1005 - Local Data Collection)
> - Alert: Sudden file access to shares
> - IOC: Large volume of reads from financial/HR shares
> - False Positive: Legitimate backup processes
> - Mitigation: Monitor off-hours access, baseline file access
>
> Stage 8: IMPACT (T1486 - Encrypt Sensitive Data)
> - Alert: Mass file encryption, file extensions changing
> - IOC: Unknown encryption process, ransom note creation
> - False Positive: Legitimate backup operations
> - Mitigation: File integrity monitoring, recovery plans
>
> This layered approach catches ransomware at MULTIPLE stages instead of relying on a single signature."

---

## 🔗 How This Connects to Everything Else

### **MITRE → Windows (Event Logs)**
- MITRE Techniques → Windows Event IDs
- T1059 (PowerShell) → Event 4688
- T1543 (Service) → Event 7045
- T1070 (Log clearing) → Event 1102
- Master Windows logs AND MITRE, you have complete detection coverage

### **MITRE → Active Directory**
- T1087 (Account Discovery) → Get-ADUser reconnaissance
- T1550.003 (Pass-the-Ticket) → Kerberos attack
- T1021.001 (RDP) → Lateral movement in domain
- T1134 (Token Manipulation) → Credential theft in AD
- AD is where the most valuable attacks happen—MITRE helps you defend it

### **MITRE → SIEM**
- Build detection rules using MITRE techniques
- Correlate multiple events using MITRE tactics
- Alert when attack chain progresses through tactics
- SIEM WITHOUT MITRE = missing the forest for the trees

### **MITRE → Threat Intelligence**
- Map threat actors to techniques they use
- Build profiles: "APT-28 always uses X, Y, Z techniques"
- Predict next moves based on group procedures
- Prioritize hunting based on known threats

### **MITRE → Incident Response**
- Use MITRE to scope breach: "How far did attacker progress? Initial Access or Exfiltration?"
- Use MITRE to remediate: "For each tactic, what do we need to fix?"
- Use MITRE to communicate: "Business stakeholders understand 'they got to Lateral Movement' better than 'T1021'"

---

## 💾 TL;DR For Busy People

### **The 14 Tactics (Quick Reference)**

| # | Tactic | Goal | Top Technique | Windows Event |
|---|--------|------|---------------|---------------|
| 1 | Reconnaissance | Gather info | OSINT | None |
| 2 | Resource Dev | Build tools | Domain registration | Firewall |
| 3 | Initial Access | Enter system | Phishing (T1566) | 4688 |
| 4 | Execution | Run code | PowerShell (T1059) | 4688 |
| 5 | Persistence | Stay in system | Service (T1543) | 7045, 4698 |
| 6 | Priv Escalation | Get admin | UAC bypass (T1548) | 4688 |
| 7 | Defense Evasion | Hide activity | Obfuscation (T1027) | 1102 |
| 8 | Cred Access | Steal keys | Brute Force (T1110) | 4625 |
| 9 | Discovery | Map network | Account enum (T1087) | 4688 |
| 10 | Lateral Movement | Move around | RDP (T1021) | 4625, 5140 |
| 11 | Collection | Gather data | File access (T1005) | 5140 |
| 12 | Command & Control | Control malware | HTTP (T1071) | 5156 |
| 13 | Exfiltration | Steal out | Over C2 (T1041) | Firewall |
| 14 | Impact | Damage | Ransomware (T1486) | 4688 |

### **What Makes You Dangerous:**

1. **You know the 14 tactics in order** ✅
2. **You can map Windows events to MITRE** ✅
3. **You understand that one event = multiple tactics** ✅
4. **You know the top 30 techniques** ✅
5. **You can attribute attacks to APT groups** ✅

---

## 📌 Production Reality: How This Actually Works

### **Real SOC: Building MITRE-Based Detection**

**Scenario:** Your SOC needs to detect ransomware before encryption happens.

**MITRE-Based Approach:**

```
Alert on: Initial Access (T1566 - Phishing)
├─ Rule: Email with .exe or macro attachment
├─ Action: Block attachment, quarantine email

Alert on: Execution (T1204 - User Execution)
├─ Rule: User launches .exe from attachment
├─ Action: Kill process, alert analyst

Alert on: Persistence (T1547 - Autostart)
├─ Rule: Registry modification adding startup
├─ Action: Block persistence, alert

Alert on: Defense Evasion (T1562 - Disable Defenses)
├─ Rule: PowerShell disables Defender
├─ Action: IMMEDIATE ESCALATION

Alert on: Lateral Movement (T1021 - RDP)
├─ Rule: RDP from workstation to workstation
├─ Action: Block connection, isolate machines

Alert on: Collection (T1005 - Local Data)
├─ Rule: Large file access from endpoint
├─ Action: Alert, investigate data being stolen

Alert on: Impact (T1486 - Encryption)
├─ Rule: Mass file modification, .encrypted extension
├─ Action: CONTAINMENT - isolate machine immediately
```

**Result:**
- Stop at Stage 1 → No breach
- Stop at Stage 3 → Minor compromise
- Stop at Stage 7 → Contained before damage
- Miss all stages → Ransomware success

---

### **Real Breach Timeline (Why MITRE Matters)**

**Ransomware incident, actual timeline:**

```
Monday 14:00 - Email arrives (Reconnaissance complete)
Monday 14:05 - User clicks link (Initial Access complete)
Monday 14:10 - Malware runs (Execution complete)
Monday 14:15 - Service installed (Persistence complete) ← SHOULD ALERT HERE
Monday 15:00 - Defender disabled (Evasion complete) ← WINDOW CLOSING
Monday 16:00 - Creds stolen (Credential Access complete) ← POINT OF NO RETURN
Monday 17:00 - Database server accessed (Lateral Movement complete) ← CONTAINED?
Tuesday 08:00 - Data copied (Collection complete) ← TOO LATE
Tuesday 09:00 - Data exfiltrated (Exfiltration complete) ← GAME OVER
Tuesday 10:00 - Files encrypted (Impact complete) ← TOTAL LOSS

Post-incident: Logs deleted (Defense Evasion continued) ← Cover-up
```

**If SOC had MITRE-based detection:**
- Alert at Monday 14:15 (Persistence)
- Immediate investigation
- Breach contained in 1 hour

**If SOC didn't:**
- No alerts until Tuesday 10:00 (Impact)
- Breach already complete
- 24+ hours of damage

---

## 📚 Further Reading

### **Official Resources**
- [MITRE ATT&CK Framework](https://attack.mitre.org) - Official site, bookmark this
- [MITRE Groups](https://attack.mitre.org/groups/) - Real APT groups with procedures
- [MITRE Software](https://attack.mitre.org/software/) - Malware mapped to techniques

### **Learning Resources**
- MITRE ATT&CK for Threat Detection (YouTube channel)
- SpecterOps: From ATT&CK to Defense (write-ups)
- ThreatHunting.net: MITRE-based hunts
- DetectLabs: Scenario-based MITRE exercises

### **Books**
- *The Art of Memory Forensics* - Map techniques to memory artifacts
- *Incident Response & Computer Forensics* - Map MITRE to investigation
- *Operator Handbook* - Attacker perspective (understand procedures)

### **Tools**
- **Mitre ATT&CK Navigator** - Visualize techniques by group/detection
- **Cyber Kill Chain Mapper** - Map techniques to defensive controls
- **ThreatHunting Workbench** - Find gaps in detection using MITRE

---

## 🎓 Final Thought

MITRE ATT&CK is your **Rosetta Stone for cyber security**. It translates:

- Raw logs → Attack tactics
- Isolated events → Attack chains  
- Unknown malware → Known APT groups
- Random alerts → Strategic understanding

Master MITRE, and you go from "analyzing events" to "understanding attacks"—which is the difference between a junior analyst and a senior threat hunter.

**You've got this.** 🎯

---

**Document Version:** 1.0  
**Last Updated:** January 2024  
**Created for:** Blue Team Security Training  
**Next Topic:** Active Directory Security
