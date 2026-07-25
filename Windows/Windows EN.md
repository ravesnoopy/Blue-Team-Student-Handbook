# 🪟 Windows in Blue Team Operations: Complete Security Guide

---

## 📖 What Is This?

Windows is the most widely deployed operating system globally, running on ~75% of enterprise endpoints. In a Security Operations Center (SOC), Windows is **the primary attack surface** and the primary source of security logs. Understanding Windows from a blue team perspective means knowing how to detect adversary activity, investigate compromises, and harden systems against attacks. It's not just about the OS itself—it's about the **event logs, registry, processes, and network activity** that reveal what's happening on a compromised machine.

---

## 🎯 Why Does A SOC/Blue Team Professional Need This?

### **Real Job Scenarios**

1. **Endpoint Compromise Detection**
   - Analyst at SOC receives alert: RDP failed login attempts (Event 4625) spike from 5 to 150 in 10 minutes
   - Your job: Investigate if this was a breach attempt or configuration issue
   - Without Windows knowledge: You miss the attack

2. **Lateral Movement Investigation**
   - Attacker compromises one Windows machine, creates a backdoor account (Event 4720)
   - Uses that account to access file shares (Event 5140) and move to other systems
   - Your job: Track the movement, stop it, find what was stolen
   - Without Windows knowledge: You don't know where to look

3. **Persistence Detection**
   - Attacker installs a service (Event 7045) that runs ransomware every morning at 3 AM
   - Or creates a scheduled task (Event 4698) hidden in Windows Task Scheduler
   - Your job: Find it before it executes
   - Without Windows knowledge: You miss the setup phase

4. **PowerShell Attack Response**
   - Adversary uses PowerShell (Event 4688) to encode and download malware into memory
   - It never touches disk, so traditional antivirus doesn't catch it
   - Your job: Detect memory-resident malware before it runs
   - Without Windows knowledge: You don't know PowerShell is the threat vector

### **Interview Questions You'll Face**

- *"Walk me through how you would investigate a suspicious Event ID 4688 in our SIEM."*
- *"What's the difference between Event Logs and the Windows Registry, and why does each matter?"*
- *"Describe a lateral movement attack on Windows and how you'd detect it."*
- *"Why is PowerShell more dangerous than cmd.exe?"*
- *"How would you detect a scheduled task created by malware?"*
- *"What Event IDs indicate an account compromise?"*

---

## 🔍 The Concept Broken Down

### **Part 1: The Three Layers of Windows You Must Know**

#### **Layer 1: Event Logs (What Happened)**
**Definition:** Structured records of system events generated in real-time when actions occur on Windows.

**Location:** Event Viewer → Windows Logs → Security, Application, System

**What they show:**
- User logons/logoffs (Event 4624, 4647)
- Process execution (Event 4688)
- Account creation (Event 4720)
- Service installation (Event 7045)
- Network connections (Event 5156)
- File access (Event 5140)

**Why it matters for SOC:**
- These logs are forwarded to your SIEM in real-time
- They're the primary data source for threat detection
- They're also the first thing attackers try to delete (Event 1102 = cover-up)

**Example:** User "admin" logs in at 3 AM from an unknown IP:
```
Event 4624: Successful Logon
User: admin
Logon Time: 2024-01-15 03:22:15
Source IP: 192.168.1.50
Logon Type: 3 (Network)
```

#### **Layer 2: Registry (How It's Configured)**
**Definition:** A hierarchical database that stores configuration data for the OS and applications.

**Key locations:**
- `HKEY_LOCAL_MACHINE\Software` = System-wide software settings
- `HKEY_LOCAL_MACHINE\System` = Device drivers, services
- `HKEY_CURRENT_USER\Software` = User-specific settings
- `HKEY_LOCAL_MACHINE\SAM` = Local user password hashes (offline only)

**What attackers hide here:**
- Malware persistence mechanisms
- Stolen credentials cached from failed logins
- Alternate data streams
- Disabled security features (Windows Defender, Firewall)

**Why it matters:**
- Reveals adversary techniques before they execute
- Shows long-term persistence mechanisms
- Indicates system hardening or deliberate weakening

**Example:** Attacker disables Windows Defender:
```
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender
DisableAntiSpyware = 1
```

#### **Layer 3: Processes & Memory (What's Running)**
**Definition:** The active programs executing in RAM at any given moment.

**Tools to see them:**
- Task Manager (GUI)
- PowerShell: `Get-Process`
- Command line: `tasklist`
- SIEM: Event 4688 (Process Creation)

**Why it matters:**
- Malware often runs only in memory (never touches disk)
- Event 4688 logs include the **full command line** (arguments, encoded payloads)
- Memory-resident attacks are the most dangerous because antivirus can't see them

**Example:** PowerShell executing encoded malware:
```
Event 4688: Process Creation
Image: C:\Windows\System32\powershell.exe
CommandLine: "powershell.exe -enc JABjAG8AbQA="
ParentImage: explorer.exe
```

---

### **Part 2: Why Windows Security Matters in a SOC**

**Windows is the attack target because:**

1. **It's everywhere** (~75% of enterprise endpoints)
2. **It's complex** (hundreds of moving parts, many misconfigurations)
3. **It runs business applications** (where the crown jewels live)
4. **It has built-in remote access** (RDP, WinRM) that attackers abuse
5. **It trusts other Windows machines by default** (Active Directory = forest fire if one burns)

**The attacker's perspective:**
- Compromise 1 Windows endpoint
- Use it to scout the network (reconnaissance)
- Steal credentials from that machine
- Use those credentials to access other systems
- Once in AD, compromise everything connected

**The blue team's perspective:**
- Detect the initial compromise at Event 4625 (RDP failed login spike)
- Stop lateral movement at Event 5140 (suspicious file access)
- Prevent persistence at Event 7045 (malicious service installation)
- Contain the damage by isolating the machine

---

### **Part 3: The 10 Event IDs You Must Memorize**

| Event ID | Name | Attack Phase | Detection Goal | Example |
|----------|------|--------------|-----------------|---------|
| **4688** | Process Creation | Execution | Malware/tool launch | powershell.exe with suspicious arguments |
| **4720** | User Account Created | Persistence | Backdoor account creation | "admin_backup" created outside change window |
| **4625** | Failed Logon | Initial Access | RDP brute force, dictionary attack | 200 failed attempts in 5 minutes |
| **4624** | Successful Logon | Success indicator | Anomalous user/time/location | User logs in at 3 AM from unknown IP |
| **7045** | Service Installed | Persistence | Malware service creation | Service "WindowsUpdate" installed with binary in %TEMP% |
| **4724** | Account Password Reset | Credential Theft | Lateral movement prep | Admin resets user password, then uses it to access file share |
| **5140** | Network Share Access | Lateral Movement | Data exfiltration, reconnaissance | User accesses sensitive file share at unusual hour |
| **4698** | Scheduled Task Created | Persistence | Scheduled malware execution | Task runs PowerShell at 3 AM daily |
| **1102** | Audit Log Cleared | Cover-up | Adversary evidence destruction | Security log cleared (IMMEDIATE ESCALATION) |
| **4769** | Kerberos Ticket Requested | Lateral Movement | Kerberoasting, privilege escalation | User requests ticket for domain admin service |

---

## ⚙️ What You MUST Memorize

### **The Event ID Memory System**

**4688 - Process Creation: "4-6-88" = Four processes (parent-child-grandchild-great-grandchild tree)**
- Parent: explorer.exe
- Child: powershell.exe
- Grandchild: cmd.exe
- Great-child: malware.exe

**Remember:** Each process in Event 4688 shows parent & child → build the execution tree

---

**4720 - User Created: "4-7-20" = 7 letters in "USUARIO" = user creation (Spanish mnemonic)**
- Backdoor accounts usually have names like: admin_backup, svc_restore, temp_admin
- Created outside business hours → red flag
- Used immediately → double red flag

---

**4625 - Failed Logon: "4-6-25" = 46 repeated attempts = brute force (6 attempts, repeated x25)**
- Single failure = normal
- 10 failures in 1 minute = brute force attempt
- 200 failures in 5 minutes = ATTACK IN PROGRESS

---

**7045 - Service Install: "7-0-45" = Windows has ~45 built-in services, new one = suspicious**
- Services run at SYSTEM level (highest privilege)
- They start automatically at boot
- Perfect for persistence

---

**1102 - Log Cleared: "1-1-0-2" = First and most critical event**
- If you see this: assume breach
- Attacker ALWAYS clears logs to hide evidence
- This event itself proves they were there

---

**4769 - Kerberos Ticket: "4-7-69" = Kerberos protocol (port 88 + 69 = tickets)**
- Normal: user requests ticket for printer
- Suspicious: user requests ticket for domain controller or domain admin service
- This is how attackers move laterally

---

### **Key Terms You Must Know**

| Term | Definition | SOC Relevance |
|------|-----------|---------------|
| **RDP** | Remote Desktop Protocol—allows remote login to Windows | Primary attack vector for initial access |
| **WinRM** | Windows Remote Management—PowerShell remoting | Less visible than RDP, harder to detect |
| **Active Directory (AD)** | Central directory of all users/computers/permissions in domain | Compromise AD = compromise everything |
| **NTFS** | NT File System—permissions (who can read/write files) | Shows who can access what on disk |
| **Share Permissions** | Network access rights (separate from NTFS) | Attackers use to move data to other systems |
| **Kerberos** | Authentication protocol used in Windows networks | Attackers forge tickets to move laterally |
| **PowerShell** | Modern scripting language in Windows (executes in memory) | Primary tool for post-exploitation |
| **Task Scheduler** | Automated task execution at specified times | Used by attackers for persistence |
| **Services** | Background processes that run at system startup | Used by attackers for persistence |

---

## 📚 What You MUST Understand

### **Understanding 1: The Attack Chain on Windows**

Every compromise follows this pattern:

```
1. INITIAL ACCESS
   └─ Event 4625: Brute force RDP
      OR Event 4688: Phishing link opens malware

2. PERSISTENCE
   └─ Event 4720: Create backdoor account
   └─ Event 7045: Install malicious service
   └─ Event 4698: Create scheduled task
   └─ Event 4624: Backdoor account logs in successfully

3. CREDENTIAL THEFT
   └─ Attacker runs lsass.exe memory dump
   └─ Registry shows cached credentials
   └─ Event 4720 shows new account with stolen password

4. LATERAL MOVEMENT
   └─ Event 4625: Failed attempts on other machines (password spray)
   └─ Event 4624: Successful login to other machines
   └─ Event 5140: Access to network shares to steal data

5. COVER-UP
   └─ Event 1102: Attacker clears security log
   └─ Registry deletion attempts
```

**Your job:** Interrupt this chain as early as possible (ideally at step 1 or 2)

---

### **Understanding 2: Why PowerShell is More Dangerous Than cmd.exe**

| Feature | cmd.exe | PowerShell |
|---------|---------|-----------|
| **Execution** | Disk-based | Memory-based |
| **Detectability** | Easy (writes to disk) | Hard (only in RAM) |
| **Capabilities** | Limited | Full .NET access |
| **Obfuscation** | Minimal | Can encode entire commands |
| **Remoting** | No | Yes (WinRM) |
| **Examples in wild** | Legacy malware | Modern ransomware, APT tools |

**Example PowerShell attack (impossible with cmd):**
```powershell
# Encoded command (base64) - looks like gibberish
powershell.exe -enc JABjAG8AbQA=

# Decodes to: $com=....(download malware)
# Attacker: malware never touches disk
# Analyst: sees Event 4688 with encoded argument
```

---

### **Understanding 3: Active Directory is Your Crown Jewel**

Think of Active Directory like a bank's vault:

- **Normal user** = Customer (can access their own account)
- **Domain Admin** = Bank manager (can access everything)
- **Domain Controller** = Vault itself (stores all credentials, all permissions)

**If attacker compromises Domain Admin or Domain Controller:**
- They can reset passwords for ANY user (Event 4724)
- They can add themselves to ANY group (Event 4728)
- They can access ANY file share (Event 5140)
- **Result:** Entire organization compromised in minutes

**Attack example (Kerberoasting):**
```
1. Attacker finds user "service_account"
2. Runs: Invoke-Kerberoast (PowerShell tool)
3. Requests Kerberos ticket for service_account (Event 4769)
4. Offline, cracks the ticket (takes hours, not real-time)
5. Gets cleartext password for service_account
6. Uses it to access high-privilege systems
```

**Your detection:** Alert on Event 4769 for unusual service ticket requests

---

### **Understanding 4: The Registry Reveals Secrets**

Attackers use the Registry to:

1. **Disable security features**
   ```
   DisableAntiSpyware = 1
   DisableRealtimeMonitoring = 1
   ```

2. **Add persistence hooks**
   ```
   Run key: malware.exe (executes at every boot)
   Runonce key: backdoor.ps1 (executes once)
   ```

3. **Store stolen data**
   ```
   Recent files list
   Cached network credentials
   Browser history
   ```

**SOC perspective:** Registry inspection is forensic work (happens AFTER breach discovery via Event Logs)

---

## 🚨 Practical Application: The Real Attack Scenario

### **Scenario: Ransomware Attack on Windows Endpoint**

**Timeline of Events:**

**23:15 - Initial Access**
```
Event 4625 (Failed Logon)
User: unknown
Source IP: 203.0.113.45
Attempts: 1 of many to come
Reason: Brute force attempt via RDP
```

**23:47 - Brute Force Escalates**
```
Event 4625 (Failed Logon) - repeated 150 times
Source IP: 203.0.113.45
Attempts in 30 minutes: 150 (obvious attack pattern)
```

**00:05 - Breach Successful**
```
Event 4624 (Successful Logon)
User: admin (legitimate account)
Source IP: 203.0.113.45 (external, matches failed attempts)
Logon Type: 3 (Network/RDP)
→ ATTACKER IS NOW ON SYSTEM
```

**00:12 - Reconnaissance**
```
Event 4688 (Process Creation)
Image: C:\Windows\System32\cmd.exe
CommandLine: "cmd /c whoami"
Parent: explorer.exe
→ ATTACKER LEARNING ABOUT THE SYSTEM
```

**00:15 - PowerShell for Advanced Attack**
```
Event 4688 (Process Creation)
Image: C:\Windows\System32\powershell.exe
CommandLine: powershell.exe -enc JABsaXN0... [encoded command]
Parent: cmd.exe
→ ATTACKER RUNNING ENCODED MALWARE (bypasses antivirus)
```

**00:18 - Persistence Setup**
```
Event 7045 (Service Installed)
ServiceName: WindowsUpdate
ImagePath: C:\ProgramData\malware.exe
StartType: Auto
→ MALWARE WILL RUN EVERY BOOT
```

**00:20 - Backdoor Account Creation**
```
Event 4720 (User Account Created)
NewAccountName: svc_backup
→ ATTACKER CAN LOG IN EVEN IF ORIGINAL ACCOUNT IS DISABLED
```

**00:25 - Data Exfiltration Begins**
```
Event 5140 (Network Share Access)
ShareName: \\server\Finance
UserName: admin
AccessMask: Read (malware reading files to encrypt)
→ ATTACKER MAPPING OUT WHAT TO ENCRYPT
```

**03:00 - Ransomware Executes (Scheduled for off-hours)**
```
Event 4688 (Process Creation)
Image: C:\ProgramData\ransomware.exe
CommandLine: ransomware.exe /encrypt /key:stolen_key
Parent: services.exe
→ FILES BEING ENCRYPTED
→ BUSINESS IS DOWN
```

**03:05 - Cover-Up**
```
Event 1102 (Audit Log Cleared)
ClearedBy: svc_backup
→ ATTACKER DESTROYING EVIDENCE
```

---

### **Your Response as Blue Team Analyst**

| Time | Action | Priority |
|------|--------|----------|
| 23:47 | Alert on 4625 spike (150 failed RDP in 30 min) | MEDIUM - Block IP |
| 00:05 | Alert on 4624 from same IP as failed attempts | **HIGH** - Isolate machine immediately |
| 00:15 | Alert on PowerShell with encoded arguments | **CRITICAL** - Kill process, investigate |
| 00:18 | Alert on 7045 service installation (malware) | **CRITICAL** - Delete service, remove malware |
| 00:20 | Alert on 4720 new account creation | **HIGH** - Disable account, reset admin password |
| 00:25 | Alert on 5140 unusual share access | **HIGH** - Investigate what was accessed |
| 03:05 | Alert on 1102 log cleared | **CRITICAL** - Assume containment failed, escalate |

**If you caught Event 00:05 (4625 spike):** Ransomware prevented
**If you caught Event 00:15 (PowerShell):** Ransomware prevented
**If you caught Event 00:18 (malicious service):** Ransomware prevented  
**If you only catch Event 03:05 (log cleared):** You're 3 hours too late

---

## ❌ Common Mistakes Students Make

### **Mistake 1: Thinking "One Event ID = One Alert"**

**Wrong approach:**
```
Alert: Event 4688 process creation detected
Response: Look at one process, clear it if it looks ok
```

**Correct approach:**
```
Alert: Event 4688 process creation detected
Response: 
1. Look at PARENT process (where did this come from?)
2. Look at CHILD processes (what did this launch?)
3. Look at TIMING (when did it run?)
4. Look at FREQUENCY (how many times?)
5. Cross-reference with other events (5140, 4720, etc.)
```

**Real consequence:** Analyst clears the parent process and misses the whole attack chain. Attacker stays in system.

---

### **Mistake 2: Ignoring Event 1102 (Log Cleared)**

**Wrong approach:**
```
Event 1102 appears
Analyst: "Probably scheduled maintenance"
Response: None
```

**Correct approach:**
```
Event 1102 appears
Analyst: "Someone is destroying evidence"
Response: IMMEDIATE ESCALATION
- Check if log clearing was authorized
- If not: ASSUME ACTIVE BREACH
- Isolate machine, collect forensics, call incident response
```

**Real consequence:** Attacker successfully covers their tracks and remains undetected for months.

---

### **Mistake 3: Not Understanding the Difference Between Failed Logon (4625) and Successful Logon (4624)**

**Wrong approach:**
```
4625 (Failed) = Alert
4624 (Successful) = Normal
```

**Correct approach:**
```
4625 (Failed) × 200 = Brute force attempt (alert)
4624 (Successful) from same IP = BREACH (escalate immediately)
```

**Real consequence:** Analyst sees successful login as normal and doesn't connect it to the failed attempts. Breach begins.

---

### **Mistake 4: Not Checking the CommandLine in Event 4688**

**Wrong approach:**
```
Event 4688: powershell.exe started
Analyst: "PowerShell is normal, users use it all the time"
Response: Ignore
```

**Correct approach:**
```
Event 4688: powershell.exe started
Check: What is the FULL CommandLine?
- If: -enc JABzAGM... (encoded) = ALERT
- If: -NoProfile (no user profile) = ALERT
- If: -ExecutionPolicy Bypass (bypass restrictions) = ALERT
- If: -NonInteractive -NoProfile (scripted) = ALERT
- If: Normal user launching notepad = OK
```

**Real consequence:** Analyst misses PowerShell exploitation because they didn't read the full command line.

---

## 🧪 Practice Scenario: Incident Investigation

### **Scenario: You Get an Alert at 2 PM**

**Alert:** "Suspicious PowerShell execution detected on WORKSTATION-05"

**Raw Event Data:**
```
Event 4688: Process Creation
Time: 2024-01-15 14:22:33
Computer: WORKSTATION-05
User: jsmith
Image: C:\Windows\System32\powershell.exe
CommandLine: powershell.exe -enc JgBjAG8AbQA=
ParentImage: C:\Windows\Explorer.exe
ParentProcessID: 4892

Followed by:

Event 5140: Network Share Access
Time: 2024-01-15 14:23:45
Computer: WORKSTATION-05
User: jsmith
ShareName: \\FileServer01\Payroll
AccessMask: Read

Followed by:

Event 5140: Network Share Access
Time: 2024-01-15 14:24:12
Computer: WORKSTATION-05
User: jsmith
ShareName: \\FileServer01\HR
AccessMask: Read
```

### **Analysis Questions:**

**Q1:** Is this definitely an attack, or could it be legitimate?
- A) Definitely attack
- B) Could be legitimate (user using PowerShell for work)
- C) Need more data to decide
- D) Ask user directly

**Q2:** What's the most concerning event here?
- A) PowerShell execution itself
- B) The encoded PowerShell command
- C) File access to Payroll and HR
- D) The timing

**Q3:** What should you do FIRST?
- A) Kill the PowerShell process
- B) Disable the user account
- C) Check if jsmith normally accesses those shares at this time
- D) Escalate to incident response

**Q4:** What additional events would you want to see?
- A) Event 4720 (new accounts created)
- B) Event 4698 (scheduled tasks created)
- C) Event 4724 (password resets)
- D) All of the above

### **Solution:**

**Q1 Answer: C) Need more data**
- Reason: PowerShell itself isn't malicious, and file access could be normal. But the COMBINATION is suspicious.

**Q2 Answer: B) Encoded PowerShell command**
- Reason: Legitimate PowerShell commands are usually not encoded. Encoding = hiding something.

**Q3 Answer: C) Check if jsmith normally accesses those shares**
- Reason: Jumping to action could alert the attacker if they're still active. First, validate if this is normal or abnormal behavior.

**Q4 Answer: D) All of the above**
- Reason: If there ARE Events 4720/4698/4724, this confirms persistence/escalation tactics.

---

## 🎯 Interview Questions You Might Get

### **Easy Questions (L1 - Junior Analyst)**

**Q: "Explain the difference between Event Log and Registry."**

**Expected answer:**
> "Event Log is a record of what happened (actions taken on the system), while Registry is configuration data stored persistently. Event Log goes to SIEM for real-time detection; Registry is checked during forensics."

---

**Q: "What does Event ID 4688 tell you?"**

**Expected answer:**
> "It's Process Creation - it logs when a new process starts, including the program name, who ran it, the full command line arguments, and the parent process. This is critical because malware execution shows up here."

---

**Q: "Why is PowerShell more dangerous than cmd.exe?"**

**Expected answer:**
> "PowerShell runs in memory and can encode its commands, making it harder to detect. It has access to the full .NET framework, so it can do much more damage than cmd. And it's built into Windows, so there's no suspicious file to find."

---

### **Medium Questions (L2 - Mid-Level Analyst)**

**Q: "Walk me through how you'd investigate a suspected ransomware attack on a Windows endpoint."**

**Expected answer:**
> "First, I'd check Event 4625 for failed RDP attempts to identify the initial attack. Then Event 4624 for successful logon from external IP. Event 4688 for suspicious process execution (especially PowerShell). Event 7045 for malicious service installation. Event 5140 for data being read before encryption. And critically, check if Event 1102 was cleared (evidence destruction). If all these events exist, it's a confirmed breach and I'd escalate to incident response to isolate the machine before containment fails."

---

**Q: "An attacker compromises a Windows machine but doesn't create new accounts or services. How would you detect them?"**

**Expected answer:**
> "Even without creating persistence mechanisms, they have to DO something, so:
> - Event 4688: If they executed any tools (PowerShell, cmd, malware)
> - Event 5140: If they accessed file shares to steal data
> - Event 4625 + 4624: The initial breach via RDP or other logon
> - Memory analysis: If they're running malware only in RAM
> - Outbound network connections: If they're communicating with command & control
> If they don't touch Event Logs or Registry, the only trail is their actions (4688, 5140, network)."

---

### **Hard Questions (L3 - Senior/Interview)**

**Q: "Design a detection strategy for Kerberoasting attacks on Windows."**

**Expected answer:**
> "Kerberoasting exploits the Kerberos protocol to crack service account passwords offline. Detection:
> 1. Monitor Event 4769 (Kerberos Service Ticket requested) for unusual service ticket requests
> 2. Alert on multiple 4769 events from the same user in short timeframe
> 3. Focus on service accounts (vs user accounts) in ticket requests
> 4. Correlate with Event 5140 or 4688 (what did they do after getting the ticket?)
> 5. Investigate if user who requested ticket has legitimate reason to access that service
> Implementation: SIEM rule that triggers on >5 Event 4769 per user per hour, filters out known-good traffic, escalates to incident response"

---

**Q: "An attacker has admin credentials to a Windows endpoint. How would you detect that they're using those credentials, even if they delete Event Logs?"**

**Expected answer:**
> "If Event Logs are deleted (Event 1102), they're destroyed at the endpoint level. But:
> 1. SIEM should have already forwarded logs before deletion
> 2. Network SIEM data (firewall, DNS, proxy) still shows outbound connections
> 3. File server logs (Event 5140 on the SERVER) show who accessed files
> 4. Time-of-access data on files themselves reveals who accessed them
> 5. Registry analysis post-breach shows what was changed
> The attacker can only delete LOCAL event logs; they can't delete logs on servers they accessed. So investigate those servers' Event Logs for the attacker's username."

---

**Q: "What would indicate Active Directory compromise, and how would you respond?"**

**Expected answer:**
> "AD compromise indicators:
> - Event 4720/4722: Multiple user accounts created/modified outside change window
> - Event 4728: Mass addition of users to Domain Admins group
> - Event 4769: Ticket requests for sensitive service accounts
> - Event 4625: Credential spray attacks (same password tested on many accounts)
> Response:
> 1. IMMEDIATE: Isolate DC from network (disconnect network cable)
> 2. Check for backdoor accounts (4720) and delete them
> 3. Reset passwords for all Domain Admin accounts
> 4. Check Group Policy Objects (GPOs) for malicious changes
> 5. If compromised: assume entire AD forest is compromised, plan rebuild
> This is the nuclear option because AD compromise = entire organization compromised"

---

## 🔗 How This Connects to Everything Else

### **Windows → Active Directory**
- Windows machines authenticate to AD using Kerberos
- Compromised Windows endpoint = potential entry point to AD
- AD compromise = all Windows machines compromised
- Study AD if you want to understand enterprise security

### **Windows → Network Security**
- Windows generates outbound network connections (Event 5156)
- Firewall logs show what the endpoint is trying to communicate with
- DNS requests show what the endpoint is resolving
- Cross-reference: If Event 4688 shows PowerShell execution, check firewall logs for outbound connections to malware C&C

### **Windows → SIEM**
- Event Logs are the primary data source for Windows monitoring
- SIEM collects, parses, and correlates Windows events
- Correlation: Event 4625 + Event 4624 from same source = breach
- Without Windows Event Log knowledge, you can't use SIEM effectively

### **Windows → Incident Response**
- IR collects forensic artifacts from Windows (Registry, EventLog, MFT)
- Timeline reconstruction: Which events happened in what order?
- Attribution: Who was the attacker (which user account)?
- Damage assessment: What files were accessed/modified?

### **Windows → Threat Hunting**
- Hunt for signs of persistence: Event 7045 (services), Event 4698 (tasks)
- Hunt for reconnaissance: Event 4688 (whoami, ipconfig commands)
- Hunt for lateral movement: Event 5140 (file access patterns)
- Hunt for exfiltration: Event 5140 + network logs (large data transfers)

---

## 💾 TL;DR For Busy People

### **The 10 Most Critical Event IDs (Memorize These)**

| Event ID | What | Why Alert |
|----------|------|-----------|
| **4688** | Process started | Malware/tool execution |
| **4720** | User created | Backdoor account |
| **4625** | Login failed | Brute force, password spray |
| **4624** | Login succeeded | + 4625 from same IP = breach |
| **7045** | Service installed | Malware persistence |
| **4724** | Password reset | Credential theft |
| **5140** | Share accessed | Data theft, lateral movement |
| **4698** | Task created | Scheduled malware |
| **1102** | Logs cleared | Cover-up = ESCALATE |
| **4769** | Kerberos ticket | Lateral movement prep |

### **The Attack Sequence (In Order)**

1. **Event 4625** (RDP brute force) → Attacker testing passwords
2. **Event 4624** (successful login) → Attacker in the system
3. **Event 4688** (PowerShell) → Attacker executing tools
4. **Event 7045** or **4698** (service/task) → Attacker creating persistence
5. **Event 5140** (share access) → Attacker stealing data
6. **Event 1102** (logs cleared) → Attacker destroying evidence

**Interrupt at ANY of these steps and you stop the attack.**

### **One-Sentence Summary**

*"Windows Event Logs are your SOC's primary threat detection data source; master the top 10 Event IDs, understand how they chain together, and you can detect 80% of attacks before significant damage occurs."*

---

## 📌 Production Reality: How This Actually Works

### **Real SOC Environment**

**Scenario:** You're an analyst at a mid-size company with 500 Windows endpoints.

**Daily Reality:**
- All 500 endpoints forward Event Logs to SIEM every 30 seconds
- SIEM ingests ~1 million events per hour (just from Windows)
- Your job: Find the 1 malicious event among 1 million legitimate ones

**Your tools:**
```
Rule 1: Alert if Event 4625 > 20 in 10 minutes (brute force)
Rule 2: Alert if Event 4688 with PowerShell -enc argument (encoded malware)
Rule 3: Alert if Event 7045 with binary in %TEMP% (malware service)
Rule 4: Alert if Event 1102 is generated (log clearing = breach)
Rule 5: Alert if Event 5140 outside business hours (after-hours data theft)
```

**False Positives You'll Deal With:**
- PowerShell script by IT admin = looks like malware
- Service installation by software updater = looks like malware persistence
- System rebooting = generates many events in quick succession
- User account lockout = generates many 4625 events

**Your job:** Build rules that catch real attacks while ignoring false positives. This is the art of SOC work.

---

### **Real Breach Timeline**

**Company XYZ gets ransomware:**

```
Day 1 (2 PM): Attacker starts RDP brute force (Event 4625 spike)
→ SOC alerts
→ Analyst dismisses as "probably a user with bad password"

Day 1 (3 PM): Attacker succeeds (Event 4624 from external IP)
→ No alert configured (oops!)
→ Attacker now on network, undetected

Day 2-3: Attacker builds persistence (Event 7045, 4698)
→ Alerts exist but are ignored as "system maintenance"

Day 4: Attacker moves laterally (Event 5140, 4724)
→ Events are generated but not correlated
→ Analyst sees each event in isolation, misses pattern

Day 5 (3 AM): Ransomware executes
→ Files encrypted
→ Business down for 3 days
→ Ransom demanded

Day 5 (3:05 AM): Attacker clears logs (Event 1102)
→ Evidence partially destroyed
→ Forensics becomes harder

**Post-mortem:**
> "If SOC had correlated Event 4625 + Event 4624 on Day 1, we would have caught it in minutes. If they had understood Event 7045 and 4698, we would have caught it on Day 2. Instead, all the data was there and we missed it."
```

---

### **What This Means for Your Career**

1. **Windows knowledge is non-negotiable** in any SOC role
2. **Event Log understanding is your foundational skill** (not optional)
3. **Building effective detection rules** requires knowing what normal looks like
4. **Learning SIEM without Windows knowledge** is like learning to drive without understanding engines

**Next steps after mastering this:**
- Learn Active Directory in depth (where Windows security gets complex)
- Learn SIEM correlation (how to connect multiple events into patterns)
- Learn Threat Hunting (proactive searching for missed attacks)
- Learn Incident Response (what to do after you find the attack)

---

## 📚 Further Reading

### **Books**
- *Windows Security Internals* by Mark Russinovich (deep dive)
- *The Art of Memory Forensics* (capture malware in memory)
- *Incident Response & Computer Forensics* by Chris Ryan

### **Documentation**
- [Microsoft Security Audit Events](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-events)
- [Windows Event Log Recommendations](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation)
- [Mitre ATT&CK Framework - Windows Techniques](https://attack.mitre.org/platforms/windows/)

### **Labs & Practice**
- DetectLabs: Windows event log analysis scenarios
- TryHackMe: Windows Fundamentals path
- AlteredSecurity: Active Directory labs
- SANS Holiday Challenge (annual, free)

### **Tools to Learn**
- Event Viewer (native Windows)
- PowerShell (for log querying)
- Splunk or ELK (SIEM platforms)
- Velociraptor (endpoint visibility)

---

## 🎓 Final Thought

Windows is the battlefield where cyber defense happens. Every day, hundreds of thousands of attacks target Windows endpoints. Your job as a blue team analyst is to:

1. **See the attack** (understand Event Logs)
2. **Understand the attack** (know the techniques and tactics)
3. **Stop the attack** (respond before damage)

Master Windows, and you master the foundation of modern SOC work.

**You've got this.** 🛡️

---

**Document Version:** 1.0  
**Last Updated:** January 2024  
**Created for:** Blue Team Security Training  
**Next Topic:** Active Directory Security
