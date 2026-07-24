# 🔐 ACTIVE DIRECTORY: Complete Guide for Blue Team
## Identities, Policies, Auditing, and Attacks

---

## 📖 What is Active Directory?

**In 30 seconds:**
Active Directory (AD) is **Microsoft's centralized service that manages identities (users, groups, computers), security policies (GPO), permissions, and auditing across a Windows corporate network**. It's not just password storage — it's the **brain of the infrastructure that controls who can do what, where, and when**.

**Analogy:**
```
Password CSV = A static file with data
Active Directory = A city with government, laws, police, and records

In the CSV: "jsmith: password123"
In AD: "jsmith can access these servers, cannot install software,
       must change password every 30 days, is in these groups,
       account locked after 5 failed logins, etc."
```

---

## 🎯 Why Does a SOC/Blue Team Need to Understand AD?

### In Real Work

| Scenario | Without Understanding AD | With Understanding AD |
|-----------|--------------------------|------------------------|
| Attacker creates a hidden admin account | "How did they get in?" | Search EventID 4720 in AD audit logs |
| User can't access shared folder | "Why?" | Check NTFS permissions + group membership |
| Admin password changed at 3 AM | "Is that normal?" | No, check EventID 4723 + source IP |
| Malware attempts lateral movement | "Can it?" | Depends on delegated permissions in AD |
| Service account is compromised | "What access does it have?" | Check groups + ACLs in AD |

### Interview Questions

1. **"What is Active Directory and why is it important?"**
   - Expected: "A centralized system managing identities, policies, and auditing. If compromised, attackers control the entire network."

2. **"What's the difference between an Account and an Object in AD?"**
   - Expected: "An account is a user. An object is any entity (user, group, computer, printer). Attackers compromise accounts but create objects for persistence."

3. **"How would you detect a hidden admin account created by an attacker?"**
   - Expected: "Look for EventID 4720 (Account created), especially at odd hours, and correlate with EventID 4728 (Added to Administrators group)."

4. **"What is a Golden Ticket?"**
   - Expected: "A forged Kerberos ticket. An attacker can do anything without needing a password. Very dangerous because it's hard to detect."

---

## 🔍 Breaking Down the Concept

### Part 1: AD's Hierarchical Structure

```
DOMAIN (example: corp.local)
│
├── FOREST (Tree of domains)
│   └── Contains 1+ related domains
│
├── Domain Controllers (DC)
│   ├── DC01
│   ├── DC02
│   └── Replicate information between them
│
└── ORGANIZATIONAL UNITS (OUs) - Logical containers
    ├── OU=Users
    │   ├── OU=Administrativo
    │   │   └── Admin accounts
    │   ├── OU=Developers
    │   │   └── Developer accounts
    │   └── OU=Employees
    │       └── Regular user accounts
    │
    ├── OU=Computers
    │   ├── OU=Servers
    │   ├── OU=Workstations
    │   └── OU=Laptops
    │
    └── OU=Groups
        ├── Group: Administrators
        ├── Group: Domain Admins
        ├── Group: Finance-Users
        └── Group: IT-Support
```

**Why it matters:** Attackers target policy changes at the FOREST level or modify admin groups within an OU.

### Part 2: Objects vs Accounts vs Groups

**OBJECT** = Entity in the directory

```json
{
  "objectClass": "user",
  "sAMAccountName": "jsmith",
  "displayName": "John Smith",
  "userPrincipalName": "jsmith@corp.local",
  "mail": "jsmith@corp.local",
  "department": "Sales",
  "manager": "mgonzalez",
  "memberOf": ["group-sales", "group-vpn-access"],
  "lastLogonTimestamp": "2024-01-15T14:32:00Z",
  "pwdLastSet": "2024-01-01T08:00:00Z",
  "accountExpires": "2025-01-01",
  "userAccountControl": "512"
}
```

**ACCOUNT** = Specific login concept

```
An ACCOUNT can be:
├─ User (user) - Interactive login
├─ Service Account - Run services/scheduled tasks
├─ Computer Account - Machine authentication on network
└─ Managed Service Account (MSA) - Special service type
```

**GROUP** = Collection of objects (typically users and computers)

```
Example Group: "Domain Admins"
├─ Members: user1, user2, user3, ...
├─ Permissions: Full Control on all servers
├─ Scope:
│   ├─ Global (within domain)
│   ├─ Domain Local (on specific server)
│   └─ Universal (multiple domains in org)
└─ Type:
    ├─ Security group (for permissions)
    └─ Distribution group (for emails)
```

**Why it matters:** Attackers compromise accounts but maintain persistence via groups (especially "Domain Admins" or hidden groups).

### Part 3: The Authentication Protocol (Kerberos)

Active Directory uses **Kerberos** for authentication. Here's the flow:

```
┌───────────┐
│   USER    │ "I want access to FILE-01 server"
└─────┬─────┘
      │ 1. Sends credentials
      ▼
┌──────────────────────────────────────┐
│  Authentication Server (AS)          │
│  (Part of the Domain Controller)     │
│  - Validates password                │
│  - Issues TGT (Ticket Granting Ticket)
└─────┬────────────────────────────────┘
      │ 2. Receives TGT (valid for 10 hours by default)
      ▼
┌──────────────────────────────────────┐
│  Ticket Granting Server (TGS)        │
│  (Part of the Domain Controller)     │
│  - Receives TGT                      │
│  - Issues Service Ticket             │
└─────┬────────────────────────────────┘
      │ 3. Receives Service Ticket
      ▼
┌──────────────────────────────────────┐
│       FILE-01 (Server)               │
│  - Validates Service Ticket          │
│  - Grants access                     │
└──────────────────────────────────────┘
```

**In an attack (Golden Ticket):**

```
Attacker + AD compromise = Can forge a TGT
→ Access to ANY server without needing a password
→ Valid for 10 hours (by default)
→ Very hard to detect
```

---

## ⚙️ Active Directory EventIDs — The Heart of Auditing

### Most Critical EventIDs

When AD emits an event, it's logged in the **Security Event Log** of Domain Controllers.

| EventID | Event | Example Log | Criticality | Detection |
|---------|-------|-------------|-------------|-----------|
| **4624** | Successful login | User jsmith logged on | 🟡 Medium | Only if unusual IP or time |
| **4625** | Failed login | Failed password for jsmith | 🟡 Medium | 5+ in 5 min = brute force |
| **4720** | Account created | New user account created | 🔴 CRITICAL | Any account creation at 2-5 AM |
| **4722** | Account enabled | User account enabled | 🔴 CRITICAL | Reactivating a disabled account |
| **4723** | Password changed | Password changed for admin | 🔴 CRITICAL | Especially if admin at odd hours |
| **4728** | Member added to group | User added to Administrators | 🔴 CRITICAL | Always, especially if admin group |
| **4729** | Member removed from group | User removed from group | 🟠 HIGH | If it's a critical group |
| **4730** | Group deleted | Security group deleted | 🔴 CRITICAL | Possible audit group removal |
| **4739** | Policy changed | Domain policy changed | 🔴 CRITICAL | Changes in GPO (antivirus, audit, firewall) |
| **4752** | Member added to global group | Added to global security group | 🟠 HIGH | Depends on the group |
| **4768** | Kerberos ticket granted | Kerberos TGT issued | 🟡 Medium | Base for Golden Ticket detection (noisy) |
| **4769** | Service Ticket requested | Service Ticket requested | 🟡 Medium | Kerberoasting detection |
| **4771** | Kerberos auth failed | Pre-authentication failed | 🟡 Medium | Possible Kerberoasting attempt |
| **5136** | AD object modified | Directory Service Object Modified | 🔴 CRITICAL | Changes in sensitive attributes |

### Raw Log Examples

**EventID 4720 (Account created) - CRITICAL:**

```text
<134>Jan 15 03:14:22 DC01 MSWinEventLog 1 Security 94512
Mon Jan 15 03:14:22 2024 4720 Security SYSTEM User Success Audit
DC01 User Account Management

A user account was created.

Target Account Name: backdoor_admin
Target Account ID: CORP\S-1-5-21-3623811015-3361044348-30300820-2157
Creator Subject: Security ID: CORP\Administrator Account Name: Administrator
Domain Name: CORP Logon ID: 0x3E7

Attributes:
SAM Account Name: backdoor_admin
Display Name: (none)
User Principal Name: -
Home Directory: -
Script Path: -
Profile Path: -
User Workstations: -
Password Last Set: <never>
Account Expires: <never>
Primary Group ID: 513
AllowReversibleEncryption - Disabled
```

**Why is it critical?**
- Time: 3:14 AM (abnormal)
- Account Name: "backdoor_admin" (suspicious)
- Creator: Administrator (could be attacker with privileges)
- No initial password (indicates it will be changed later)

---

**EventID 4728 (Member added to Administrators group) - CRITICAL:**

```text
<134>Jan 15 03:16:45 DC01 MSWinEventLog 1 Security 94513
Mon Jan 15 03:16:45 2024 4728 Security SYSTEM User Success Audit
DC01 User Account Management

A member was added to a security-enabled global group.

Group: Group Name: Administrators
Group Domain: CORP
Actors: Subject Security ID: CORP\S-1-5-21-3623811015-3361044348-30300820-500
Member Added: Member Name: backdoor_admin
Member SID: CORP\S-1-5-21-3623811015-3361044348-30300820-2157
```

**Why is it critical?**
- Affected Group: Administrators (maximum privileges)
- User Added: backdoor_admin (the suspicious account)
- Correlation with EventID 4720: Created 2 minutes ago

---

**EventID 4723 (Password changed) - CRITICAL if admin:**

```text
<134>Jan 15 03:18:00 DC01 MSWinEventLog 1 Security 94514
Mon Jan 15 03:18:00 2024 4723 Security SYSTEM User Success Audit
DC01 User Account Management

An attempt was made to change an account's password.

Subject: Account Name: Administrator
Domain Name: CORP
Logon ID: 0x3E7

Target Account: Account Name: domain_admin_main
Account Domain: CORP

Change Type: Computer Access Control (Caller changed password of target account)
```

**Why is it critical?**
- Main admin account (primary administrator)
- Time: 3:18 AM (attacker working)
- Visible pattern: 4720 (create) → 4728 (add to group) → 4723 (change password)

---

### EventID Correlation — Detecting a Complete Attack

**Scenario: Attacker establishes persistence in AD**

```
Timeline:
2024-01-15 03:14:22 → EventID 4720: Account "backdoor_admin" created
2024-01-15 03:16:45 → EventID 4728: "backdoor_admin" added to "Administrators"
2024-01-15 03:18:00 → EventID 4723: Password changed for "backdoor_admin"
2024-01-15 03:20:15 → EventID 4624: "backdoor_admin" successful login from IP 198.51.100.45

SIEM correlation:
└─> 4720 + 4728 + 4723 + 4624 in 6 minutes = CRITICAL ATTACK
    Action: Block IP, disable account, investigate access
```

---

## 🚨 Attack Techniques Against AD (MITRE ATT&CK)

### Attack #1: Brute Force (T1110 - Credential Access)

**What is it?**
Trying multiple passwords against one account.

**Log:**
```text
<134>Jan 15 09:14:01 DC01 ... 4625 ... Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:03 DC01 ... 4625 ... Account Name: jsmith Status: 0xC0000064
<134>Jan 15 09:14:05 DC01 ... 4625 ... Account Name: jsmith Status: 0xC0000064
[5+ times in 5 minutes, same IP]
```

**Detection (SIEM Rule):**
```yaml
title: Brute Force Against AD Account
logsource:
  product: windows
  service: security
detection:
  selection:
    EventID: 4625
    Status: 'C0000064'  # Wrong password
  timeframe: 5m
  condition: selection | count(TargetUserName) by SourceIp >= 5
level: high
tags:
  - attack.credential_access
  - attack.t1110_001
```

---

### Attack #2: Golden Ticket (T1558_001 - Privilege Escalation)

**What is it?**
An attacker who has dumped the `krbtgt` hash can forge a valid Kerberos ticket (TGT) and access any server without a password.

**Why it's dangerous:**
- Doesn't need to change passwords
- Valid for 10 hours by default (no re-authentication)
- **VERY hard to detect** in normal logs

**Indicators (difficult, but they exist):**
```text
EventID 4768/4769 with abnormal timestamps
EventID 4768 without a corresponding EventID 4624
Multiple Service Tickets in seconds (unusual pattern)
```

---

### Attack #3: Kerberoasting (T1558_003 - Credential Access)

**What is it?**
A user can request a Service Ticket for any service without special privileges. The ticket contains a hash that can be cracked offline.

**Log:**
```text
<134>Jan 15 14:32:01 DC01 ... 4769 ...
  Service Name: MSSQLSvc/database.corp.local
  Client: jsmith@CORP
  Ticket Encryption Type: 0x17 (RC4 - easy to crack)
  Failure Code: 0x0 (successful)
```

**Detection:**
Look for EventID 4769 with multiple Service Tickets in short time, especially with RC4 encryption type.

---

### Attack #4: Credential Dumping (T1003 - Credential Access)

**What is it?**
Attacker runs tools (mimikatz, ntdsutil) to dump passwords and hashes from AD.

**Indicators:**
```text
EventID 4656: Access to NTDS.dit (AD database)
EventID 5136: Changes in AD objects (may indicate post-dump)
EventID 4662: Access to sensitive objects
```

---

### Attack #5: ACL Abuse (T1098_003 - Persistence)

**What is it?**
Modifying permissions (ACLs) on AD objects so the attacker can:
- Change other users' passwords
- Add members to groups
- Create accounts
- Modify GPOs

**Log:**
```text
<134>Jan 15 15:45:22 DC01 ... 5136 ...
  Object: CN=Admins,CN=Groups,DC=corp,DC=local
  Attribute: nTSecurityDescriptor
  Operation: Modify
  Modification: Added GenericAll permission to S-1-5-21-...-2157
```

**Danger:** Attacker can make changes without needing to be in the Administrators group.

---

## 🛡️ Defense and Detection in AD

### Tier 1: Basic Protection

```
✅ Strong password policies:
   ├─ Minimum 14 characters
   ├─ Complexity (uppercase, lowercase, numbers, symbols)
   ├─ Change every 90 days
   └─ History: don't reuse last 24 passwords

✅ Account lockout:
   ├─ After 5 failed attempts
   └─ Lock for 30 minutes

✅ Auditing enabled:
   ├─ Success + Failure on all sensitive events
   └─ Retention: minimum 90 days
```

### Tier 2: AD Hardening

```
✅ Administrator accounts:
   ├─ Create dedicated "Tier 0" account
   ├─ Don't use for daily tasks
   ├─ Use PAM (Privileged Access Management)
   └─ MFA required

✅ Sensitive groups (24/7 monitoring):
   ├─ Domain Admins
   ├─ Enterprise Admins
   ├─ Schema Admins
   ├─ Backup Operators
   └─ Account Operators

✅ Domain Controller protection:
   ├─ Restrictive firewall
   ├─ Only RPC on port 135
   ├─ No SMB from workstations
   └─ No direct internet access
```

### Tier 3: Detection in SIEM

```
Rule: Account creation + Addition to admin group within 5 minutes

title: Likely Persistence - Account Creation + Admin Group Addition
logsource:
  product: windows
  service: security
detection:
  account_created:
    EventID: 4720
  added_to_admin:
    EventID: 4728
    Group: 'Administrators'
  timeframe: 5m
  condition: account_created AND added_to_admin
level: critical
tags:
  - attack.persistence
  - attack.t1098
```

---

## ❌ Common Mistakes Analysts Make

### Mistake #1: Ignoring "normal password changes"

```
❌ WRONG:
"User changed their password, it's normal"

✅ RIGHT:
"Did the same user change their password?
Or was it an admin? What time? From which IP?
If admin changed another admin's password at 3 AM = INVESTIGATE"
```

---

### Mistake #2: Confusing "Account" with "Object"

```
❌ WRONG:
"I monitor all accounts in AD"

✅ RIGHT:
"I monitor CRITICAL accounts (admins) AND all OBJECTS (groups, GPOs)
because attackers maintain persistence via groups"
```

---

### Mistake #3: Not correlating AD with other logs

```
❌ WRONG:
"I see EventID 4720 (account created), close as admin mistake"

✅ RIGHT:
"I see EventID 4720 + search within 10 minutes for:
├─ EventID 4728 (added to admin group)?
├─ EventID 4624 (successful login from rare IP)?
├─ EventID 4723 (password changed)?
└─ If I find these, it's an ATTACK PATTERN = ESCALATE"
```

---

### Mistake #4: Trusting only "account name"

```
❌ WRONG:
Rule: "Alert if account named 'admin%' is created"

✅ RIGHT:
"Alert if ANY account is created at 2-5 AM,
regardless of name. Attacker might call it 'john_backup'
to look legitimate"
```

---

## 🧪 Practical Exercises

### Exercise #1: Analyze an Event Correlation

**Logs presented in order:**

```
[2024-01-15 03:14:22] DC01 EventID 4720
  New Account: temporary_admin
  Creator: Administrator

[2024-01-15 03:16:45] DC01 EventID 4728
  Member Added: temporary_admin
  Group: Domain Admins

[2024-01-15 03:20:00] DC01 EventID 4624
  Successful Login: temporary_admin
  Source IP: 203.0.113.45
  Logon Type: 3 (Network)

[2024-01-15 03:21:30] FW-EDGE-01 Firewall Log
  Outbound Connection: 203.0.113.45 -> 185.220.100.50:443
  Protocol: HTTPS
```

**Questions:**
1. What is the attack pattern?
2. Which EventID is most critical?
3. How would you write a SIEM rule to detect this?

**Expected Answer:**
1. **Pattern:** Persistence. Attacker creates admin account, enables it, logs in, exfiltrates data.
2. **Most critical EventID:** 4728 (adding to Domain Admins = maximum access).
3. **Rule:**
```yaml
title: Likely Attacker Persistence
detection:
  account_created:
    EventID: 4720
  added_to_domain_admins:
    EventID: 4728
    Group: 'Domain Admins'
  successful_login:
    EventID: 4624
    TargetUserName: # the created account
  timeframe: 10m
  condition: account_created AND added_to_domain_admins AND successful_login
level: critical
```

---

### Exercise #2: Golden Ticket or Normal Login?

**Scenario:** Your SIEM detects:
```
User: admin_main
Login Time: 2024-01-15 14:32:00
From IP: 192.168.1.100 (internal workstation)
EventID: 4624 (successful login)
Logon Type: 3 (Network)
```

**Question:** Is it suspicious?

**Analysis:**
```
✓ Time normal (14:32 during business hours)
✓ IP internal (workstation, not internet)
✓ EventID 4624 is normal

BUT is there a Golden Ticket indicator?
├─ Is there EventID 4768 without prior 4624? (TGT without normal login)
├─ Are there multiple Service Tickets in seconds? (4769 abnormal pattern)
└─ Is login "normal" but then accesses resources they never accessed?

Conclusion: NO Golden Ticket based on this alone.
You'd need: Abnormal EventID 4768 OR EventID 5136 ACL changes.
```

---

### Exercise #3: Design a Secure AD Policy

**Scenario:** You're Security Architect at a company with 5,000 users.

**Task:** Define:
1. OU structure
2. Admin groups
3. Password policies
4. Auditing

**Expected Answer:**

```
OUs:
├─ OU=Tier0 (Domain Admins only)
│  └─ Access: MFA, PAM, max 2 admins
│
├─ OU=Tier1 (Server Admins)
│  ├─ OU=Database-Admins
│  ├─ OU=Infrastructure-Admins
│  └─ Access: MFA, password change every 60 days
│
├─ OU=Tier2 (Regular Admins)
│  ├─ OU=Team-Leads
│  ├─ OU=IT-Support
│  └─ Access: MFA, change every 90 days
│
└─ OU=Users (regular employees)
   ├─ OU=Finance
   ├─ OU=Sales
   └─ OU=Engineering

Auditing:
├─ EventID 4720, 4722, 4723, 4728, 4729, 4730: 100% logging
├─ EventID 4624 (successful login): Only unusual IP OR admin
├─ EventID 5136 (changes): 100% logging of ACL changes
└─ Retention: 180 days (compliance)
```

---

## 🎯 Interview Questions

### Level 1 (Entry-Level SOC Analyst)

**Q1: "What is Active Directory?"**
> "A Microsoft service managing identities (users, computers, groups), policies (GPO), and auditing in a Windows corporate network."

**Q2: "What's the difference between an Account and an Object in AD?"**
> "An Account is a user with login credentials. An Object is any entity in AD: user, group, computer, printer, etc."

**Q3: "What EventID indicates an attacker created an admin account?"**
> "EventID 4720 (Account created). Especially if at 2-5 AM and followed by EventID 4728 (added to Administrators)."

### Level 2 (Mid-Level SOC Analyst)

**Q1: "How would you detect a Golden Ticket?"**
> "Look for EventID 4768 (TGT issued) without prior EventID 4624, or multiple EventID 4769 (Service Tickets) in seconds. Also EventID 5136 with sensitive object changes but no normal login (unusual)"

**Q2: "Why is it important to monitor EventID 4728 (Member added to group)?"**
> "It shows someone added a user to a group. If it's an admin group, the attacker just escalated privileges. If it's a critical group, it could affect multiple servers."

**Q3: "What's the difference between Kerberoasting and brute force against AD?"**
> "Brute force: multiple password attempts against one account online (EventID 4625). Kerberoasting: request Service Tickets (EventID 4769), which contain hashes that can be cracked offline without direct detection."

### Level 3 (Senior Analyst / Detection Engineer)

**Q1: "How would you design an AD detection program in a 10,000-user SOC?"**
> "1) Define 'Tier 0' (Domain Admins, Enterprise Admins) with 24/7 monitoring. 2) Create SIEM rules for persistence patterns (4720+4728+4723). 3) Establish baselines of normal login per user. 4) Correlate EventID 4625 (failed) with 4624 (successful) = possible attack. 5) Monitor GPO changes (EventID 5136). 6) Use threat intelligence for dumped credentials."

**Q2: "What's the most dangerous attack against AD and how would you detect it?"**
> "Golden Ticket, because it allows access without changing passwords. Detection is hard: need EventID 4768 without 4624 prior, multiple abnormal 4769, or correlate with ACL/GPO changes (EventID 5136). Alternative: use EDR to detect mimikatz/ntdsutil execution."

**Q3: "How would you mitigate a Domain Admin compromise?"**
> "1) Change password immediately (but attacker might have Golden Ticket). 2) Reset krbtgt password (invalidates all existing Golden Tickets). 3) Force logout of all sessions. 4) Audit EventID 5136 for ACL changes. 5) Investigate what was accessed during compromise. 6) Consider DC reset if very severe."

---

## 🔗 Connections to Other Topics

- 🔥 **SIEM & Detection**: SIEM rules to detect AD attacks (EventID 4720, 4728, etc.)
- 🛡️ **Incident Response**: How to respond to an AD compromise
- 📊 **MITRE ATT&CK**: Attack techniques against AD (T1098, T1110, T1558, T1003)
- 🔐 **Kerberos**: Authentication protocol in AD
- 🌐 **Windows Security Logs**: Source of AD EventIDs
- 🔍 **Threat Hunting**: Searching for compromise indicators in AD
- 🛠️ **Forensics**: Analyzing breaches related to AD

---

## 💾 TL;DR - Quick Reference

### AD Hierarchy
```
FOREST → DOMAIN → OUs → OBJECTS (Users, Groups, Computers)
```

### Critical EventIDs (memorize these)
| EventID | Event | Action |
|---------|-------|--------|
| 4720 | Account created | Investigate if 2-5 AM |
| 4728 | Added to group | Critical if admin group |
| 4723 | Password changed | Critical if admin |
| 4625 | Login failed | Alert if 5+ in 5 min |
| 4624 | Login successful | Correlate with 4625 + firewall |
| 5136 | Object modified | Any sensitive change |

### Attack Techniques
```
T1110 → Brute Force
T1558 → Golden Ticket / Kerberoasting
T1003 → Credential Dumping
T1098 → Account Manipulation
T1484 → Domain Policy Modification (GPO changes)
```

### Compromise Indicators (IOCs)
```
✓ Account created + added to admin group within 5 min
✓ Admin login from external IP at 3 AM
✓ Bulk password changes for multiple admins
✓ GPO changes disabling antivirus/audit
✓ EventID 4768 without prior EventID 4624 (possible Golden Ticket)
```

### Basic Defense
```
1. Strong password policies (14+ chars, 90 days)
2. MFA for all admins
3. 24/7 monitoring of Tier 0 (Domain Admins)
4. 100% auditing of critical EventIDs
5. PAM (Privileged Access Management)
```

---

## 📌 Production Reality

### Real Case #1: AD Compromise detected by SIEM

```
Timeline:
- Friday 18:00 → Last successful backup
- Friday 22:14 → EventID 4720: Account "admin_backup" created
- Friday 22:16 → EventID 4728: Added to Domain Admins
- Saturday 03:45 → EventID 5136: GPO change (antivirus disabled)
- Saturday 04:20 → EventID 4769: Kerberoasting detected
- Sunday 14:00 → Discovery: 2 TB of data exfiltrated

Lesson: EventID correlations detected the attack
5+ hours before manual discovery.
```

### Real Case #2: Common false positive

```
❌ FALSE ALARM:
Rule: "EventID 4728 + Administrators group = CRITICAL"

Reality: New admin being onboarded by IT manager.
Occurred at 09:30 AM during business hours, with valid justification.

✅ SOLUTION:
Change rule:
"EventID 4728 + Domain Admins + (time < 6 AM OR time > 22 PM) + external IP = CRITICAL"
```

---

## 📚 Next Steps

1. **Memorize the critical EventIDs** (4720, 4728, 4723, 4625, 5136)
2. **Understand Kerberos** — it's the foundation of all AD authentication
3. **Practice writing SIEM rules** to detect attack patterns
4. **Study MITRE ATT&CK** specifically AD attack techniques
5. **Lab work**: Create a small AD, simulate attacks, detect in SIEM

---

## 📌 Remember

```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  Active Directory = The heart of the corporate network    │
│                                                            │
│  If AD is compromised, the ENTIRE network is compromised │
│                                                            │
│  Attackers aim for:                                        │
│  1) Initial access (Brute Force, Kerberoasting)          │
│  2) Persistence (Create account, Golden Ticket)          │
│  3) Escalation (Add to Domain Admins)                    │
│  4) Exfiltration (Change GPO, dump credentials)          │
│                                                            │
│  Your job: Detect every step.                            │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

---

**Last updated**: 2024
**Version**: 1.0
**For**: Blue Team Students - TripleTen Bootcamp
