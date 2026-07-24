# 🔄 LATERAL MOVEMENT: The Art of Moving Through the Network

---

## 📖 WHAT IS LATERAL MOVEMENT?

**30-Second Definition:**
Lateral movement is the technique an attacker uses to **move from one compromised host to another within the network**, after gaining initial access to a device. It's not about escalating privileges on the same machine; it's about **horizontally extending access** to other servers, workstations, or critical devices.

**Mental Model:**
```
Initial Access (e.g., phishing)
    ↓
Privilege Escalation (admin on THIS host)
    ↓
LATERAL MOVEMENT ← YOU ARE HERE (move to OTHER hosts)
    ↓
Persistence + Data Exfiltration
```

---

## 🎯 WHY DOES A SOC/BLUE TEAM PROFESSIONAL NEED THIS?

### Interview Questions You'll Face:
- *"How would you detect a lateral movement attack in your SIEM?"*
- *"What's the difference between privilege escalation and lateral movement?"*
- *"What indicators would you look for to identify Pass the Hash?"*
- *"How would you investigate if someone accessed a server they shouldn't?"*
- *"Which Windows events are associated with lateral movement?"*

### In Your SOC Job:
- **60-70% of breaches** use lateral movement to expand access
- Detecting lateral movement **BEFORE attackers reach critical servers** is your job
- Most attacks are discovered by **anomalies in lateral movement logs**, not initial access
- You need to understand **attacker techniques** to know **what to look for in defense**

---

## 🔍 THE CONCEPT BROKEN DOWN

### **Part 1: The Typical Scenario**

**REAL SOC SCENARIO:**
```
09:15 - Phishing attack → User opens malicious document
        ✓ Attacker has access to workstation "LAPTOP-JUAN"
        ✓ But as a normal user (no admin rights)

09:20 - Local Privilege Escalation
        ✓ Windows exploit (e.g., CVE-2021-1732)
        ✓ Attacker is now SYSTEM on LAPTOP-JUAN

09:25 - LATERAL MOVEMENT BEGINS
        ✓ Attacker sees there's a "FILE-SERVER-01"
        ✓ Uses compromised admin credentials to access via SMB
        ✓ Now on FILE-SERVER-01

09:30 - Continuous movement
        ✓ From FILE-SERVER-01, targets DB-SERVER
        ✓ Uses Pass the Hash technique
        ✓ Sensitive data exfiltration

10:00 - YOU (SIEM analyst) detect it in logs
        ✓ If done right, you stop it here
```

**Why didn't you catch it earlier?** Because attackers use techniques to make it look legitimate.

---

### **Part 2: WHY IS IT SO CRITICAL?**

#### The Problem:
A compromised server on your network is like having a "bridge" to all other servers it can reach.

**If an attacker has:**
- ✅ Admin access on FILE-SERVER
- ✅ Open network permissions
- ✅ Valid credentials (stolen or recycled)

**They can reach:**
→ Database servers
→ Domain Controller
→ Backup systems
→ Cloud infrastructure
→ Critical systems

#### Why attackers do this:
1. **Access expansion** → more hosts = more data
2. **Redundancy** → if one host is lost, they have 5 more
3. **Reach objectives** → they need to reach DB-SERVER, not just FILE-SERVER
4. **Hide activity** → spreading activity across multiple hosts avoids detection

---

### **Part 3: Specific Lateral Movement Techniques**

**Now the critical part: HOW ATTACKERS DO IT = HOW YOU DEFEND**

#### **#1: PASS THE HASH (PtH)**

**What is it?**
```
Attacker obtains the NTLM HASH of an account (e.g., administrator)
Without needing the plaintext password
Uses that hash directly in SMB, RDP, WinRM
System: "You have the correct hash → access granted"
```

**How it works technically:**
```
Windows NTLM authentication:
1. Client sends: username + NTLM HASH
2. Server validates the hash against its database
3. If match → access granted

Difference with password:
- Normal: password → hash → comparison
- PtH: attacker already has the hash → direct comparison
- Windows doesn't detect that the password was NEVER entered
```

**How does an attacker get the NTLM hash?**
- Dumping LSASS (Local Security Authority Subsystem Service)
- Extracting from SAM file (requires SYSTEM privileges)
- Tools: mimikatz, secretsdump.py, gsecdump

**Example attack flow (for understanding, not execution):**
```bash
# Attacker with SYSTEM access:
mimikatz.exe
mimikatz > privilege::debug
mimikatz > sekurlsa::logonpasswords
# Output: NTLM hashes of current users

# Then attacks SMB with those hashes:
psexec.py -hashes :HASHNTLM username@target-server
```

**Why it's dangerous:**
- No password needed to be cracked
- Password never traverses the network
- Many tools accept hashes directly
- Old technique but STILL WORKS today

---

#### **#2: PASS THE TICKET (PtT) - Kerberos**

**What is it?**
```
Instead of NTLM hash, attacker steals a Kerberos Ticket
(TGT = Ticket Granting Ticket)
Uses it to authenticate to other hosts without the password
```

**When is it used?**
- Windows environments with Active Directory
- More modern than PtH
- Kerberos is the standard in AD

**Technical detail (what you need to know):**
```
Normal Kerberos flow:
User → requests TGT from Domain Controller
DC → grants valid TGT
User → uses TGT to request access to server X

PtT attack:
Attacker steals the TGT
Uses that TGT without knowing the password
Access granted
```

**Why it matters:**
- Harder to detect than PtH
- Standard in modern Windows environments
- Kerberos tickets are valid for hours
- Attacker can use them across the domain

---

#### **#3: RDP LATERAL MOVEMENT**

**What is it?**
Attacker accesses another host via **Remote Desktop Protocol (RDP)** using stolen credentials.

**Steps:**
```
1. Attacker has user credentials (admin or regular)
2. Identifies server with RDP enabled (port 3389)
3. Connects: mstsc.exe or xfreerdp to target
4. Interactive visual access to remote server
```

**Why it's effective:**
- RDP is legitimate and common (IT uses RDP daily)
- Stolen credentials can be from any user
- If an admin used RDP from that host, their credentials are cached

**Detection challenge:**
- Looks legitimate in logs
- Need to correlate: unusual time, unusual IP origin, unusual target

---

#### **#4: SMB LATERAL MOVEMENT (File Shares)**

**What is it?**
Attacker accesses **shared folders** on other hosts to:
- Explore sensitive files
- Deploy malware
- Exfiltrate data

**Steps:**
```
1. Attacker on HOST-A
2. Sees that HOST-B has \\HOST-B\Shared open
3. Accesses with compromised credentials: net use \\HOST-B\Shared
4. Explores files / deploys payload
```

**WHY THIS IS CRITICAL:**
- SMB is how Windows shares files between machines
- Many servers have open shares without restrictions
- SMB logs tell you EXACTLY WHICH FILES were accessed
- Attackers often target HR, Finance, Backup shares

---

#### **#5: WINRM (Windows Remote Management)**

**What is it?**
Attacker executes commands remotely via WinRM (port 5985).

**Typical command:**
```powershell
# Attacker:
Invoke-Command -ComputerName TARGET-SERVER -Credential $cred -ScriptBlock { whoami }
```

**Why it's dangerous:**
- It's legitimate (admins use WinRM)
- If port is open and you have credentials → full access
- Command execution = complete control
- Hard to distinguish from admin activity

---

### **Part 4: CRITICAL DIFFERENCE: Lateral Movement vs Privilege Escalation**

**THIS COMES UP IN INTERVIEWS:**

| Aspect | Privilege Escalation | Lateral Movement |
|--------|---------------------|------------------|
| **What is it?** | Escalate from user to admin ON SAME HOST | Move from one host to ANOTHER host |
| **Example** | User → SYSTEM on LAPTOP-JUAN | LAPTOP-JUAN (admin) → FILE-SERVER |
| **Prerequisites** | Initial access to the host | Privilege escalation + credentials of next host |
| **Result** | More permissions on 1 machine | Access to multiple machines |
| **Detected by** | Privilege elevation events | Remote access logs to multiple hosts |

**CORRECT ORDER OF ATTACK:**
```
1. INITIAL ACCESS (phishing, RCE) → basic access to 1 host
2. PRIVILEGE ESCALATION → admin on that host
3. LATERAL MOVEMENT ← IT COMES AFTER (need admin to move effectively)
4. PERSISTENCE (malware to maintain access)
5. DATA EXFILTRATION (steal data)
```

**Why the order matters:**
If attacker does NOT escalate before moving:
- Limited movement (only to shares where they have permission)
- Cannot dump hashes
- Cannot use Pass the Hash

If they ESCALATE first:
- Can dump all credentials (mimikatz)
- Can use Pass the Hash without limits
- AGGRESSIVE access across entire network

---

## ⚙️ WHAT YOU MUST MEMORIZE

### **Memorization #1: The 5 Main Techniques**

```
🔑 PtH (Pass the Hash)
   → Steals NTLM hash, uses it directly
   → Detect: Event ID 4625 + 4624 without password change

🎫 PtT (Pass the Ticket)
   → Steals Kerberos ticket
   → Detect: TGT usage without prior authentication event

🖥️ RDP
   → Remote visual access
   → Detect: Event ID 3389 from unusual internal IPs

📁 SMB
   → Access to network shares
   → Detect: Event ID 5140 (share access) + unusual files

💻 WinRM
   → Remote command execution
   → Detect: Event ID 91 (WinRM connection)
```

### **Memorization #2: Critical Windows Event IDs**

```
Windows Event Viewer → Security logs:

4624 = Successful logon
4625 = Failed logon attempt
4672 = Special privileges assigned (admin)
4720 = User account created / password changed
3389 = RDP connection attempt
5140 = Share object accessed
5145 = Share object detailed check
91   = WinRM connection

PATTERN TO WATCH:
4625 (fail) + 4624 (success) + NO 4720 = LIKELY PtH
```

### **Memorization #3: Red Flags of Lateral Movement**

**🚨 CRITICAL RED FLAGS:**
- ✗ User accessing server they've NEVER accessed before
- ✗ Access to servers at 3 AM (outside business hours)
- ✗ Multiple failed logins (4625) followed by success (4624)
- ✗ Admin credentials used on non-admin typical hosts

**⚠️ INVESTIGATE THESE:**
- Unusual SMB activity (many files accessed)
- WinRM from IPs that normally don't use WinRM
- RDP from workstations (usually admin → workstation, not reverse)
- Multiple users accessing same critical server in short time

---

## 📚 WHAT YOU MUST UNDERSTAND DEEPLY

### **Comprehension #1: Why attackers need multiple accounts**

**Scenario:**
```
Attacker compromises account: "juan@company.com" (regular user)
Escalates to SYSTEM
Can they access FILE-SERVER?

Depends:
- If FILE-SERVER requires specific admin → NO
- If FILE-SERVER has open permissions → YES

BUT there's a problem: If they make 10 connections as JUAN,
SOC sees the pattern and gets suspicious.

ATTACKER'S SOLUTION: Use multiple accounts
- Connection 1 with juan@company.com
- Connection 2 with maria@company.com
- Connection 3 with admin-service@company.com
- Connection 4 with backup-admin@company.com

Result: Logs are dispersed, alerts scattered across different users,
correlation is difficult. And if one account gets disabled,
they have 3 more.
```

**Why this matters for Blue Team:**
- You must correlate events FROM MULTIPLE USERS
- If you see 4 different users accessing same server in 1 hour → RED FLAG
- Don't just look for 1 suspicious user, look for PATTERNS

### **Comprehension #2: "Slow and Low" in Lateral Movement**

**What is it?**
Attacker moves SLOWLY to avoid detection.

**Example:**
```
Option 1 (OBVIOUS - SOC sees immediately):
09:15 - Connection to FILE-SERVER
09:16 - Downloads 100 GB
09:17 - Connection to DB-SERVER
09:18 - Downloads data
09:19 - Connection to DC (Domain Controller)
RESULT: Massive alert, detection in minutes

Option 2 (SLOW AND LOW - Hard to detect):
09:15 - Connection to FILE-SERVER (downloads 5 MB)
14:30 - Connection to DB-SERVER (explores folders)
18:45 - Passive DC reconnaissance (no access)
Day 2: 11:00 - Access to DC

RESULT: Events distributed over time, seem legitimate, hard to correlate
```

**Defense against this:**
- Behavioral baseline (what's "normal" for each user)
- Machine learning in SIEM (detect subtle deviations)
- Correlate events across HOURS/DAYS, not just minutes

### **Comprehension #3: Legitimate access vs lateral movement**

**THE HARD PART:** An admin accessing FILE-SERVER via RDP is... legitimate.
An attacker accessing FILE-SERVER via RDP also looks the same in basic logs.

**HOW TO DIFFERENTIATE?**

| Factor | Legitimate | Lateral Movement |
|--------|-----------|------------------|
| **Time** | Business hours | 3 AM / weekends |
| **Source IP** | Admin's known IP | Different IP (compromised workstation) |
| **Pattern** | Same server accessed | New server each time |
| **Users** | Same user always | Multiple different users |
| **Speed** | Normal admin actions | Automated scripts/commands very fast |
| **Objective** | Legitimate work | Hunting for credentials / sensitive data |

**In SIEM:**
```
RED FLAG:
IF (user == "admin" AND 
    time == "03:00 AM" AND 
    source_ip DIFFERENT_FROM_NORMAL AND 
    server NEVER_ACCESSED_BEFORE)
THEN Alert: "Possible lateral movement with stolen admin credentials"
```

---

## 🚨 PRACTICAL DETECTION IN SOC (BLUE TEAM FOCUS)

### **Where do you see the logs?**

1. **Windows Event Viewer** (local, impractical at scale)
2. **SIEM** (centralized, collects from all hosts)
3. **Proxy logs** (if lateral movement crosses internet)
4. **Firewall logs** (connections between hosts)
5. **Endpoint Detection & Response (EDR)** (very sensitive to movements)

### **SOC Investigation Flow**

**STEP 1: Alert arrives in SIEM**
```
Alert: "User 'juan' accessed FILE-SERVER at 02:45 AM"
(event that normally doesn't happen)
```

**STEP 2: You investigate**
```
Questions:
- Is juan an admin? Does juan need FILE-SERVER access?
- From what IP did juan connect?
- What did juan do after connecting? (files, commands)
- Are there other suspicious events from juan that night?
```

**STEP 3: Correlation**
```
If you see:
- 02:45 AM: juan → FILE-SERVER (RDP)
- 02:50 AM: juan → DB-SERVER (SMB)
- 02:55 AM: maria (different account) → BACKUP-SERVER

Conclusion: Coordinated lateral movement with multiple accounts
```

**STEP 4: Containment (Immediate)**
```
Actions:
- Reset password of juan
- Reset password of maria
- Check if servers were modified
- Hunt for persistence (backdoors)
- Isolate compromised hosts
```

---

### **Typical SIEM queries to detect Lateral Movement**

#### **Query 1: Unusual server access**
```
event_id: (4624 OR 3389 OR 5140) AND
host: FILE-SERVER AND
user NOT IN (admin-juan, admin-maria, system) AND
timestamp: [today]

Result: Unauthorized users accessing FILE-SERVER
```

#### **Query 2: Multiple servers in short time**
```
event_id: (4624 OR 3389) AND
timeframe: 1 hour AND
count(distinct hosts) > 5

Result: One user accessing 5+ servers in 1 hour (suspicious)
```

#### **Query 3: Pass the Hash indicator**
```
event_id: 4625 (failed login) AND
event_id: 4624 (successful login) AND
NO event_id: 4720 (password change) AND
same user AND same host
within 2 minutes

Result: Typical PtH pattern
```

#### **Query 4: RDP anomaly**
```
event_id: 3389 AND
timestamp: (after hours OR weekend) AND
source_ip: internal_subnet (not remote)

Result: Unusual internal RDP connections
```

#### **Query 5: Multi-user access to critical servers**
```
event_id: (4624 OR 3389) AND
host IN (DC, DB-SERVER, FILE-SERVER) AND
count(distinct users) > 3 AND
timeframe: 1 hour

Result: Multiple users accessing critical systems
```

---

## ❌ COMMON MISTAKES STUDENTS MAKE

### **Mistake 1: Confusing lateral movement with privilege escalation**

**Student says:**
> "Lateral movement is when you go from regular user to admin"

**Correction:**
- That's PRIVILEGE ESCALATION (escalating ON THE SAME HOST)
- Lateral movement is moving TO ANOTHER HOST
- **Correct order:** Privilege escalation → Lateral movement

**Consequence in interview:**
They ask: "Describe the attack flow after initial compromise"
If you say "lateral movement" when you mean "privilege escalation" = conceptual failure

---

### **Mistake 2: Thinking Pass the Hash needs the plaintext password**

**Student says:**
> "Pass the Hash is stealing the password and using it on another host"

**Correction:**
- PtH does NOT need the plaintext password
- Only needs the NTLM HASH
- Attacker NEVER discovers the actual password

**Consequence:**
You'll investigate a PtH event looking for "password changed" events = won't find it

---

### **Mistake 3: Only looking at events from one user**

**Student says:**
> "If I search for events from user 'admin', I'll see all lateral movement"

**Correction:**
- Attacker uses MULTIPLE ACCOUNTS
- Events are scattered across different users
- You must correlate events FROM MULTIPLE USERS

**Consequence:**
Attacker moves 10 times but you investigate 1 user → miss it entirely

---

### **Mistake 4: Only investigating successful logins**

**Student says:**
> "I'll search for Event ID 4624 (successful login)"

**Correction:**
- Typical lateral movement pattern: failures + 1 success
- Many failed attempts = credential stuffing / attack
- Success after 20 failures = RED FLAG

**Correct search:**
```
4625 (failed) + 4624 (success) for same user/host within 5 minutes
```

---

### **Mistake 5: Forgetting to check Event ID 4720**

**Student says:**
> "If there's a successful login, that's lateral movement"

**Correction:**
- Legitimate admins have successful logins too
- KEY DIFFERENTIATOR: Legitimate login has 4720 (password was used)
- PtH has NO 4720 (hash bypassed password)

**Pattern to watch:**
```
4625 → 4625 → 4625 (failed attempts with wrong password)
     ↓
4624 (success WITHOUT 4720 password event) = LIKELY PtH
```

---

## 🧪 PRACTICE: ANALYZE THIS SOC SCENARIO

**REAL SCENARIO:**
```
Timeline of events in your SIEM:

01:30 AM - LAPTOP-MARIA (workstation)
  Event 4625: Failed RDP login to FILE-SERVER (user: admin-backup)
  Event 4625: Failed RDP login to FILE-SERVER (user: admin-backup)
  Event 4625: Failed RDP login to FILE-SERVER (user: admin-backup)
  Event 4624: Successful RDP login to FILE-SERVER (user: admin-backup)

01:35 AM - FILE-SERVER
  Event 5140: Share accessed - \\FILE-SERVER\Confidential
  Event 5140: Share accessed - \\FILE-SERVER\HR_DATA
  Event 5145: Object "salary_2024.xlsx" accessed
  Event 5145: Object "employee_list.csv" accessed

01:40 AM - FILE-SERVER
  Event 4624: Successful login to DB-SERVER (user: db-admin)

01:45 AM - DB-SERVER
  Event 91: WinRM command executed: "whoami"
  Event 91: WinRM command executed: "dir C:\Backups"

---

QUESTIONS TO ANSWER:

1. WHAT LATERAL MOVEMENT INDICATORS DO YOU SEE?
2. WHAT STAGE OF THE ATTACK IS THIS?
3. WHAT WOULD YOU DO NOW AS SOC ANALYST?
4. WHICH TECHNIQUE WAS USED? (PtH, RDP, SMB, WinRM, etc)
```

### **ANSWERS:**

**1. Lateral movement indicators:**
- ✓ Multiple failed RDP attempts (4625) → credential attack
- ✓ Success after failures → valid credentials found
- ✓ Access to sensitive files (HR_DATA, salaries) → DATA HUNTING
- ✓ Movement to DB-SERVER → access expansion
- ✓ WinRM commands on DB-SERVER → command execution

**2. Attack stage:**
- ✅ Initial access: YES (access to LAPTOP-MARIA)
- ✅ Privilege escalation: Probably YES (have admin credentials)
- ✅ Lateral movement: NOW HAPPENING (FILE-SERVER → DB-SERVER)
- ⏳ Persistence: Not yet visible
- ⏳ Exfiltration: In progress (accessing sensitive data)

**3. Immediate actions:**
```
1. IMMEDIATE ISOLATION:
   - Disconnect LAPTOP-MARIA from network
   - Reset password for admin-backup
   - Reset password for db-admin
   - Isolate DB-SERVER

2. INVESTIGATION:
   - Who normally uses admin-backup?
   - Was that user the one who compromised LAPTOP-MARIA?
   - What else did they do on DB-SERVER?

3. SEARCH FOR PERSISTENCE:
   - Did they create new accounts?
   - Install backdoors?
   - Modify system files?

4. SCOPE ASSESSMENT:
   - Are there other compromised hosts?
   - How much data was exfiltrated?
```

**4. Techniques used:**
- RDP lateral movement (port 3389)
- SMB file share access
- WinRM command execution
- Likely: Pass the Hash (if hash was stolen from LAPTOP-MARIA)

---

## 🎯 INTERVIEW QUESTIONS YOU'LL GET

### **Level Easy (L1 - Junior Analyst):**

**Q1:** "What is lateral movement and why is it dangerous?"
**Expected answer:**
> "Lateral movement is when an attacker who has access to one host tries to access other hosts in the network. It's dangerous because if they get admin on one server, they can compromise the entire network. It's typically the phase after privilege escalation."

**Q2:** "What's the difference between RDP and SMB lateral movement?"
**Expected answer:**
> "RDP is visual remote access (like screen sharing). SMB is access to shared folders. RDP requires interactive user session, SMB is silent. Both can be detected in logs but in different ways."

**Q3:** "What Windows event indicates RDP login?"
**Expected answer:**
> "Event ID 3389 in Security logs, or look for connections on port 3389."

---

### **Level Medium (L2 - Senior Analyst):**

**Q1:** "Walk me through a complete attack: Initial Access → Lateral Movement. What MUST happen before lateral movement?"
**Expected answer:**
> "First: initial access (phishing). Second: privilege escalation to admin on THAT host. THIRD: lateral movement. Without admin, moving to other systems is very limited. With admin, I can dump credentials, use Pass the Hash, and access any system I have network access to."

**Q2:** "How would you differentiate Pass the Hash from a legitimate RDP login?"
**Expected answer:**
> "PtH has a pattern: multiple 4625 (failed) followed by 4624 (success) with NO password change event. Legitimate RDP would have normal login sequence. I'd also use heuristics: unusual time, different source IP, non-routine server access, abnormally fast actions."

**Q3:** "If you see 5 different users accessing FILE-SERVER within 1 hour, what's your hypothesis?"
**Expected answer:**
> "Hypothesis: lateral movement with multiple compromised accounts. Attacker is distributing activity to avoid correlation. I'd investigate: do all IPs originate from same source? All outside business hours? What files did they access? Is there a pattern of escalation afterward?"

---

### **Level Hard (L3 - Senior / Threat Hunter):**

**Q1:** "Design a SIEM search to detect multi-stage lateral movement happening over days, not minutes."
**Expected answer:**
> "I'd establish behavioral baseline: what hosts/times are normal for each user. Search for deviations: new hosts accessed, new hours, new IPs. Correlate events across multiple users with similar patterns. Use statistical modeling or ML to detect 'communities' of correlated activity. Look for progression toward critical systems."

**Q2:** "How would you distinguish between legitimate admin activity and attacker using stolen admin credentials?"
**Expected answer:**
> "Context is critical. I'd check: is this the normal time/day for that admin? Is the source IP the admin's usual IP? Are they accessing systems they normally don't touch? Are they hunting for data (HR folders, passwords) vs doing routine maintenance? Correlate with EDR alerts (mimikatz execution) or unauthorized system changes."

**Q3:** "What alternatives to Pass the Hash exist, and how would you detect each?"
**Expected answer:**
> "Pass the Ticket (Kerberos) - look for TGT usage without prior authentication. Credential injection in memory - need EDR to see it. Credential stuffing automation - look for patterns of multiple 4625s. Each has different signature. PtH is old but works because many systems still use NTLM."

---

## 🔗 HOW LATERAL MOVEMENT CONNECTS TO EVERYTHING

### **In the Attack Chain:**
```
Foundations (TCP/IP, authentication)
    ↓
Network Basics (subnets, routing)
    ↓
Initial Access (phishing, exploit)
    ↓
Privilege Escalation (exploit local)
    ↓
LATERAL MOVEMENT ← YOU ARE HERE
    ↓
Active Directory (compromise DC)
    ↓
Persistence (backdoors)
    ↓
Incident Response (detection & mitigation)
```

### **In Blue Team Defense:**
```
Detection Engineering
  → What queries do I write to detect LM?
  
SIEM
  → Where do I centralize logs?
  
Windows/Linux/AD
  → What events does each OS generate during LM?
  
DFIR
  → How do I investigate if LM already occurred?
  
Incident Response
  → How do I contain active lateral movement?
```

### **In Interviews:**
```
"How would you correlate lateral movement events?"
→ Answer uses: SIEM knowledge + AD knowledge + log analysis

"How would you write a detection rule?"
→ Answer uses: Detection Engineering + Windows events

"How would you investigate a compromised host?"
→ Answer uses: DFIR + Windows forensics + AD knowledge
```

---

## 💾 TL;DR FOR BUSY PEOPLE

| Concept | Definition | How to Detect |
|---------|-----------|---------------|
| **Lateral Movement** | Move from one host to another | Multiple hosts accessed by same user in short time |
| **Pass the Hash** | Steal NTLM hash, use without password | Event 4625 + 4624 without event 4720 |
| **Pass the Ticket** | Steal Kerberos ticket | TGT usage without prior authentication |
| **RDP lateral** | Remote visual access | Event ID 3389 from unusual source IP |
| **SMB lateral** | Access network shares | Event ID 5140 + sensitive files accessed |
| **WinRM lateral** | Remote command execution | Event ID 91 from suspicious origin |
| **Privilege Escalation** | User → Admin on SAME host | Privilege elevation events (4672) |
| **Critical difference** | PE = same host, LM = different host | Look for events on DIFFERENT hosts |

**Key Pattern to Remember:**
```
4625 → 4625 → 4625 (failed attempts)
          ↓
     4624 (success without 4720) = LIKELY PtH
```

---

## 📌 PRODUCTION REALITY

### **What you actually do in a real SOC:**

**Scenario 1: Automated alert**
```
08:15 AM - Your SIEM triggers alert
"User 'admin-backup' RDP to FILE-SERVER at 02:45 AM (unusual hour)"

Your job:
1. Is it a false positive? (search for context)
2. If real, how many hosts were accessed?
3. What data was accessed?
4. Immediate isolation
5. Threat hunt: other compromised hosts?
```

**Scenario 2: Threat hunting (finding what automated alerts miss)**
```
Your auditor: "We implemented SMB logging 3 days ago"

Your job:
1. Hunt for SMB patterns that SHOULD be alerts
2. Find users accessing non-routine shares
3. Use multi-user correlation
4. Document findings
```

**Scenario 3: Incident response**
```
Malware found on LAPTOP-JUAN

Your investigation:
1. When was first access?
2. What hosts did JUAN access next?
3. What accounts were compromised?
4. When was lateral movement detected?
5. Remediation: reset all compromised credentials
```

### **What Senior Analysts will tell you:**

> "90% of the sophisticated attacks we discover are found through LATERAL MOVEMENT LOGS, not initial access. Initial access is easy to hide. Lateral movement is hard to do silently. So MASTER LATERAL MOVEMENT DETECTION."

---

## 📚 FURTHER READING

### **MITRE ATT&CK Framework:**
- **T1021: Remote Service Session Initiation**
  https://attack.mitre.org/techniques/T1021/
  - T1021.001: Remote Desktop Protocol
  - T1021.002: SMB/Windows Admin Shares
  - T1021.006: Windows Remote Management

- **T1550: Use Alternate Authentication Material**
  https://attack.mitre.org/techniques/T1550/
  - T1550.002: Pass the Hash
  - T1550.003: Pass the Ticket

- **T1570: Lateral Tool Transfer**
  https://attack.mitre.org/techniques/T1570/

### **Windows Event IDs (Microsoft Docs):**
- Event IDs 4624-4625: Logon events
- Event ID 4720: Password change
- Event ID 3389: RDP connections
- Event IDs 5140-5145: SMB share access

### **Safe Practice Environments:**
- HackTheBox: Lateral movement scenarios
- TryHackMe: Active Directory attack rooms
- SANS Cyber Aces: Log analysis exercises

---

## 🎓 NEXT TOPICS TO STUDY

**After mastering this, study:**

1. **Active Directory Attacks** (80% of lateral movement is AD-based)
2. **Privilege Escalation techniques** (prerequisite)
3. **Detection Engineering** (write your own rules)
4. **Windows Forensics** (find evidence post-attack)
5. **Threat Hunting** (proactive searching)

---

**Last updated:** July 2026  
**Level:** Intermediate (post-Foundations)  
**Study time:** 4-6 hours  
**Required for:** SOC Analyst, Threat Hunter, Incident Response Specialist

