# ☁️ Cloud Security: A Blue Team Deep Dive

## 📖 What Is This? (30-Second Definition)

Cloud Security is the practice of protecting data, applications, and infrastructure hosted on cloud platforms (AWS, Azure, Google Cloud, etc.) from unauthorized access, theft, and compromise. It combines network security, identity & access management, data protection, and incident response—adapted for a shared responsibility model where the cloud provider secures infrastructure while customers secure their configurations, applications, and access controls. Blue Teams must detect, investigate, and respond to cloud-based incidents while maintaining hybrid security (on-premise + cloud).

---

## 🎯 Why Does A SOC/Blue Team Professional Need This?

### Real Job Scenarios:
- **Hybrid Infrastructure:** Your organization runs servers on-premise AND in AWS. An incident occurs in the cloud. Your SOC must detect it, but AWS logs are different from your on-premise logs.
- **Shared Responsibility:** AWS is compromised. Is it AWS's fault or yours? You need to understand the boundary.
- **IAM Breach:** An attacker gains AWS credentials. What can they access? How do you detect unauthorized access? How do you contain it?
- **Data Exfiltration from Cloud:** A malicious insider or compromised app in your cloud environment is downloading sensitive data. Your SIEM needs cloud-specific detection rules.
- **Cost Exploitation:** Attackers launch cryptocurrency miners on your cloud infrastructure. Your cloud bill triples overnight. How do you detect this?

### Interview Questions You'll Face:
- "Explain the shared responsibility model"
- "How is cloud incident response different from on-premise?"
- "What would you look for to detect an AWS compromise?"
- "What's the difference between IAM, RBAC, and ABAC in cloud?"

---

## 🔍 The Concept Broken Down

### **Part 1: Foundation - Cloud vs. On-Premise (The Core Difference)**

#### **On-Premise Security (Traditional)**
```
Your Organization
├─ Physical Servers (you own)
├─ Network Infrastructure (you manage)
├─ Operating System (you patch)
├─ Applications (you secure)
├─ Data (you protect)
└─ Everything = YOUR responsibility
```

**Security Approach:**
- Firewall at network perimeter
- Physical access controls to data center
- Direct OS patching
- Direct application security
- Full visibility into everything

**Incident Response:**
- Access all logs (on your servers)
- Physical isolation possible
- Full forensic control

---

#### **Cloud Security (AWS/Azure/GCP)**
```
Your Organization (Cloud Customer)
├─ Applications (you build)
├─ Data (you upload)
├─ User Access Control (you configure IAM)
└─ Configuration (you manage)

Cloud Provider (AWS/Azure/GCP)
├─ Physical Infrastructure (they secure)
├─ Network (they manage)
├─ Operating System (they patch)
├─ Virtualization Layer (they maintain)
└─ Physical Security (they enforce)
```

**Key Difference:** You share responsibility. This changes everything about incident response.

---

### **Part 2: The Shared Responsibility Model (This Is Critical)**

#### **What AWS/Azure/GCP Is Responsible For:**
- Physical data center security
- Network infrastructure
- Hypervisor (virtualization layer)
- Storage systems
- **They protect the cloud**

#### **What YOU (Customer) Are Responsible For:**
- EC2/VM instance patching
- Application code security
- Database configuration
- Identity & Access Management (IAM)
- Encryption keys (if you manage them)
- Firewall rules you create
- Monitoring your own resources
- **You protect your stuff IN the cloud**

---

#### **Real Example: WannaCry Ransomware**

**Scenario 1: On-Premise**
```
WannaCry attacks your server
↓
You directly access server logs
↓
You patch immediately
↓
You see attack in progress (full visibility)
↓
You block network access
↓
You recover from backup
```

**Scenario 2: Cloud (AWS)**
```
WannaCry attacks your EC2 instance
↓
You need to check AWS CloudTrail (AWS's logging service)
↓
AWS infrastructure protected, but YOUR instance needs patching
↓
You must patch YOUR AMI (Amazon Machine Image)
↓
Visibility depends on what you configured (CloudTrail, VPC Flow Logs)
↓
You isolate by modifying security groups
↓
You recover from EBS snapshots you created
```

**Key Insight:** In cloud, YOU are responsible for knowing you were attacked. AWS tells you WHAT happened in their infrastructure, but YOU must configure logging for your applications.

---

### **Part 3: Cloud Threats vs. On-Premise Threats**

#### **Traditional On-Premise Threats:**
- Malware on servers
- Network intrusions
- Physical theft
- Insider threats with physical access

#### **Cloud-Specific Threats:**

| Threat Type | Definition | Example | Detection |
|---|---|---|---|
| **Credential Compromise** | Attacker obtains AWS access keys or passwords | Developer commits AWS keys to GitHub | AWS CloudTrail shows access from unusual IP |
| **Misconfiguration** | S3 bucket left public, security groups too open | S3 bucket containing backups publicly readable | AWS Config detects public bucket |
| **Privilege Escalation** | Low-privilege user gains admin access via IAM flaw | User with "ec2:*" permission escalates to modify IAM | CloudTrail shows unexpected IAM changes |
| **Data Exfiltration** | Attacker downloads sensitive data from cloud | Compromised EC2 instance downloads DB backups | VPC Flow Logs show large outbound transfer |
| **Resource Hijacking** | Attacker uses your cloud resources for mining/DDoS | Cryptominer deployed on your instance | Abnormally high CPU, high AWS costs |
| **Supply Chain Attack** | Compromised container image or third-party code | Malicious Docker image used in ECS | Container image scan detects vulnerabilities |
| **Unauthorized Deployment** | Attacker creates new resources in your account | Attacker launches EC2 instance in your account | CloudTrail shows EC2:RunInstances from unknown user |

---

### **Part 4: The Shared Responsibility Model - Detailed**

#### **Security Controls by Layer:**

```
Application Layer (YOU)
├─ Code security
├─ Authentication/authorization logic
└─ API security

Data Layer (SHARED)
├─ Data encryption (you provide keys, provider encrypts)
├─ Database backups (you decide, provider stores)
└─ Data classification (you decide, provider enforces)

Operating System Layer (YOU for instances, PROVIDER for managed services)
├─ Instance patching (you for EC2, AWS for RDS)
├─ Hardening (you for EC2, AWS for Lambda)
└─ Configuration management (you for EC2, AWS for containers)

Network Layer (YOU configure, PROVIDER provides)
├─ Security groups (firewall rules - YOU create)
├─ Network ACLs (network-level rules - YOU configure)
├─ VPC setup (network isolation - YOU design)
└─ AWS network security (DDoS protection - AWS provides)

Infrastructure Layer (AWS)
├─ Physical servers
├─ Hypervisor
├─ Physical access controls
└─ Supply chain security
```

---

### **Part 5: IAM - The Most Common Attack Vector**

#### **What Is IAM (Identity & Access Management)?**
IAM controls **WHO** can do **WHAT** in your cloud account.

#### **Components:**

**Users:** Individual people (developers, admins, service accounts)
```
Example: john@company.com has developer permissions
```

**Roles:** Groups of permissions applied to users or services
```
Example: "EC2-Admin" role = can start/stop/terminate EC2 instances
```

**Policies:** Specific permissions
```
Example: "Allow ec2:StartInstances on resource arn:aws:ec2:*:*:instance/*"
```

**Access Keys:** Programmatic credentials (like passwords for code)
```
Example: AKIAIOSFODNN7EXAMPLE (access key ID)
         wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY (secret key)
```

#### **The IAM Breach Scenario:**

```
Step 1: Developer commits AWS access keys to GitHub
        (Public visibility)
        ↓
Step 2: Attacker finds keys via GitHub scanning
        ↓
Step 3: Attacker uses keys to access your AWS account
        ↓
Step 4: Attacker enumerates permissions
        Runs: aws iam get-user
        ↓
Step 5: Attacker abuses permissions
        Example: If "ec2:*" permission exists, attacker can:
        - Launch new EC2 instances
        - Access data
        - Create backdoor access
        ↓
Step 6: Attacker covers tracks
        Deletes CloudTrail logs (if overly permissive)
        ↓
Step 7: You discover weeks later when AWS sends bill for $50K
```

**Detection:** CloudTrail shows access from unknown IP with exposed credentials

---

## ⚙️ What You MUST Memorize

### **Memory Trick: CIA-SR (CIA + Shared Responsibility)**
- **C** = Confidentiality (only authorized people see data)
- **I** = Integrity (data not modified by attackers)
- **A** = Availability (services are UP and accessible)
- **SR** = Shared Responsibility (YOU + Provider both responsible)

### **Cloud Security Pillars (5 Areas):**

1. **Identity & Access (IAM)**
   - Who can access what
   - Least privilege principle
   - Multi-factor authentication (MFA)

2. **Network Security**
   - VPC (Virtual Private Cloud) isolation
   - Security groups (firewall rules)
   - Network segmentation

3. **Data Protection**
   - Encryption at rest (stored data)
   - Encryption in transit (moving data)
   - Key management

4. **Compliance & Governance**
   - Meet industry standards (HIPAA, PCI-DSS, SOC2)
   - Audit logging (CloudTrail, etc.)
   - Policy enforcement

5. **Threat Detection & Response**
   - Detect unauthorized access
   - Respond to incidents
   - Forensics in cloud environment

### **The SOC's Role in Hybrid (On-Premise + Cloud):**

```
┌─────────────────────────────────────┐
│ Your On-Premise Data Center         │
│  ├─ Servers (Windows, Linux)        │
│  ├─ Firewalls                       │
│  ├─ SIEM (collecting logs)          │
│  └─ Incident Response Team          │
└──────────────┬──────────────────────┘
               │ Secure Connection (VPN/Private Link)
               │
┌──────────────▼──────────────────────┐
│ Cloud Environment (AWS/Azure)       │
│  ├─ EC2 Instances                   │
│  ├─ Databases                       │
│  ├─ Cloud Logging (CloudTrail)      │
│  └─ Cloud-based Monitoring          │
└──────────────────────────────────────┘
               │
┌──────────────▼──────────────────────┐
│ Centralized SOC Dashboard           │
│  ├─ On-Premise logs                 │
│  ├─ Cloud logs (fed into SIEM)      │
│  ├─ Alerts from both                │
│  └─ Unified incident response       │
└──────────────────────────────────────┘
```

---

## 📚 What You MUST Understand

### **Deep Comprehension Points:**

#### **1. Logging is Not Automatic in Cloud**
- **On-Premise:** Server logs exist by default. You see everything locally.
- **Cloud:** You must ENABLE logging.
  - CloudTrail (AWS API calls) - must enable
  - VPC Flow Logs (network traffic) - must enable
  - Application Logs - YOU must configure
  - Database Logs - YOU must enable

**Consequence:** If logging isn't enabled, you won't know you were attacked until data appears on the dark web.

#### **2. Visibility is Limited by Configuration**
```
Scenario 1: You enable CloudTrail
  → You see "root account accessed from IP 200.1.1.1"
  → You know you were compromised
  
Scenario 2: You didn't enable CloudTrail
  → You see $100K AWS bill
  → You know SOMETHING happened but not what
```

#### **3. Forensics in Cloud Is Different**
- **On-Premise:** Kill network cable, preserve evidence
- **Cloud:** If you isolate instance, you lose forensic data (logs still in AWS, but application logs lost)
- **Best Practice:** Create snapshot BEFORE isolating, investigate snapshot later

#### **4. Cost as a Detection Signal**
- **Normal AWS Bill:** $5K/month
- **Cryptominer Added:** $50K/month
- **Cost spike = potential attack** (but delayed detection)

#### **5. The Shared Responsibility Paradox**
- You're responsible for security, but you don't own the hardware
- AWS is responsible for infrastructure, but can't protect your misconfiguration
- **Example:** S3 bucket accidentally set to public
  - AWS secured the infrastructure ✓
  - You misconfigured the bucket ✗
  - **Result:** Your data leaked, but it's YOUR fault

---

## 🚨 Practical Application / Attack & Defense

### **Scenario 1: Credential Exposure via GitHub Commit**

**Attack Flow:**
```
1. Developer writes code to connect to AWS RDS database
2. Hard-codes AWS access keys in code: 
   AKIAIOSFODNN7EXAMPLE:wJalrXUtnFEMI/K7...
3. Developer pushes code to GitHub
4. Repo is accidentally public (or attacker has access)
5. Attacker finds credentials via GitHub scanning tools
6. Attacker configures AWS CLI with stolen keys
7. Attacker runs: aws ec2 describe-instances
8. Attacker sees company has database with customer data
9. Attacker runs: aws s3 ls s3://company-backups/
10. Attacker downloads customer database backups
11. Attacker deletes backups to cover tracks
```

**Detection (What Your SOC Should See):**

```
If CloudTrail is enabled:
  → Access to AWS API from foreign IP
  → s3:GetObject on backup bucket
  → s3:DeleteObject on backup bucket
  → All originating from same user (compromised credentials)
  
If CloudTrail is NOT enabled:
  → You won't know until:
     - AWS contacts you about suspicious activity
     - Attackers sell data on dark web
     - Customers report their data leaked
     - Company gets sued
```

**Defense (Blue Team Response):**

```
Detection Phase:
  1. CloudTrail alert: "Unauthorized user accessed s3 bucket from 200.1.1.1"
  2. Verify: Check source IP, is it company-owned? No → suspicious
  3. Investigate: Check what was accessed/downloaded
  
Containment Phase:
  1. Immediately disable exposed access keys
  2. Revoke temporary session tokens
  3. If data downloaded, start data breach investigation
  
Investigation Phase:
  1. Review CloudTrail logs for timeline
  2. What data was accessed?
  3. For how long did attacker have access?
  4. What else did they access?
  5. Check if this specific IAM user has other environments
  
Eradication Phase:
  1. Rotate all credentials
  2. Scan source code repository for exposed keys
  3. Implement secret management (AWS Secrets Manager)
  4. Enable MFA for all AWS accounts
  5. Implement credential scanning in CI/CD pipeline
  
Recovery Phase:
  1. Notify customers of potential breach
  2. Force password reset for affected accounts
  3. Implement monitoring on that S3 bucket
  4. Review all IAM user permissions (least privilege)
```

---

### **Scenario 2: Overly Permissive IAM Policy**

**Misconfiguration:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "ec2:*",
      "Resource": "*"
    }
  ]
}
```

This policy says: "This user can do ANYTHING with EC2 instances, anywhere."

**Attack Flow:**
```
1. Junior developer compromised (malware on laptop)
2. Attacker gets developer's AWS credentials
3. Attacker checks permissions: aws iam get-user-policy
4. Attacker sees "ec2:*" permission
5. Attacker launches 1000 EC2 instances for cryptomining
   Command: aws ec2 run-instances --image-id ami-12345 --count 1000
6. Each instance mines cryptocurrency for 1 week
7. You receive $100K AWS bill
```

**Detection:**
```
What should trigger alerts:
  → CloudTrail shows ec2:RunInstances x 1000 from one user
  → EC2 CPU utilization spike
  → High outbound network traffic (mining pool communication)
  → Unusual user behavior (developer running production commands)
  
If alerts are configured:
  → SOC catches within 1-5 minutes
  
If alerts are NOT configured:
  → You discover at end of month when bill arrives
```

**Defense:**
```
Prevention:
  1. Implement least privilege:
     Only allow specific actions needed
  2. Resource tagging + limits
     Limit EC2 instances created per day
  3. MFA required for sensitive actions
  
Detection:
  1. Monitor ec2:RunInstances for unusual volume
  2. Alert on CPU > 90% for extended time
  3. Monitor outbound traffic to mining pools (threat intel)
  
Response:
  1. Terminate all malicious instances immediately
  2. Disable compromised credentials
  3. Force password reset
  4. Forensic analysis of developer's laptop
  5. Implement better access controls
```

---

### **Scenario 3: S3 Bucket Left Public (Misconfiguration)**

**Setup:**
```
Company stores customer PII in S3 bucket: 
s3://company-customers/

A developer configures bucket with:
  - Block Public Access = OFF
  - Bucket Policy allows public access
  
Result: Anyone can access customer data
```

**Discovery:**
```
Option 1 (Proactive - You catch it):
  AWS Config rule checks bucket permissions
  Alert triggers: "S3 bucket is publicly accessible"
  SOC investigates and fixes
  
Option 2 (Reactive - They catch it):
  Security researcher finds bucket via scanning
  Publishes on Twitter: "Company XYZ exposed customer data"
  Media coverage, regulatory fine, lawsuits
```

**Detection & Response:**
```
Step 1: AWS Config alert or manual audit
  → Identify public S3 bucket
  
Step 2: Determine what's in it
  → AWS Access Analyzer shows: "10 million objects publicly readable"
  
Step 3: Assess exposure
  → Check CloudTrail: Has anyone accessed it?
  → Check S3 access logs (if enabled)
  
Step 4: Contain
  → Block public access immediately
  → Enable versioning to preserve current state
  
Step 5: Investigate
  → Review who created bucket
  → Why was it public?
  → What data was exposed?
  
Step 6: Notify
  → If PII exposed: notify customers
  → If payment card data: notify payment processor
  → If health data: HIPAA notification
  
Step 7: Prevent
  → Enforce bucket policies via SCPs (Service Control Policies)
  → Regular audits
  → Developer training
```

---

## ❌ Common Mistakes Students Make

### **Mistake 1: Thinking Cloud Provider Secures Your Data**
- ❌ **Wrong:** "AWS secures my databases, so they're safe"
- ✅ **Correct:** "AWS secures the RDS infrastructure. I must secure: credentials, network access, encryption keys, backups"
- **Real Consequence:** Unsecured database credentials, anyone in company can access production database

### **Mistake 2: Forgetting Logging Must Be Enabled**
- ❌ **Wrong:** "AWS automatically logs everything"
- ✅ **Correct:** "AWS provides logging services. I must enable CloudTrail, VPC Flow Logs, etc."
- **Real Consequence:** Incident occurs, you have no evidence of what happened

### **Mistake 3: Over-Permissive IAM Policies**
- ❌ **Wrong:** "Give developer ec2:* so they can do their job"
- ✅ **Correct:** "Give developer only ec2:DescribeInstances and ec2:StartInstances"
- **Real Consequence:** Compromised developer credentials = full account compromise

### **Mistake 4: Ignoring Cost as Detection Signal**
- ❌ **Wrong:** "Unexpected $100K bill? Just charge it to the department"
- ✅ **Correct:** "Cost spike = potential unauthorized resource usage. Investigate immediately."
- **Real Consequence:** Cryptominer runs for weeks before discovered

### **Mistake 5: Not Implementing MFA on Cloud Accounts**
- ❌ **Wrong:** "Password is strong enough"
- ✅ **Correct:** "Even strong passwords can be phished. MFA prevents account takeover."
- **Real Consequence:** Single credential compromise = full account compromise

---

## 🧪 Practice/Analysis

### **Scenario to Analyze:**

**Case:** Your SOC receives CloudTrail alert at 2:00 AM

```
Event: iam:AttachUserPolicy
User: dev-user@company.com
Time: 2024-01-15 02:00:15 UTC
Source IP: 203.0.113.45 (known to be in Russia)
Action: Attached "AdministratorAccess" policy to dev-user

Related events (same session):
  1. iam:CreateAccessKey (created new credentials)
  2. s3:GetObject (accessed customer database backup)
  3. s3:GetObject (accessed payment card batch file)
  4. iam:ListUsers (enumerated all users)
  5. ec2:DescribeInstances (enumerated all instances)
```

### **Questions to Answer:**

1. **Is this a real attack or false positive?**
   - Red flags: 2 AM access, Russia IP, developer doesn't typically work at this time
   - Clues: Attacker is enumerating permissions (ListUsers, DescribeInstances)
   - Answer: **High probability of real attack**

2. **What's the attacker's objective?**
   - Attached AdminAccess = wants full account access
   - Downloaded sensitive data = data theft
   - Created access key = persistence
   - Answer: **Initial access to infrastructure + data exfiltration**

3. **How do you respond?**
   ```
   Immediate (0-5 minutes):
     1. Disable dev-user credentials
     2. Invalidate all active sessions
     3. Alert incident response
     4. Check if data was downloaded (S3 access logs)
   
   Short-term (5-30 minutes):
     1. Review all CloudTrail events for this user over past 24 hours
     2. Check if other users were compromised
     3. Secure all access keys
     4. Implement deny-all policy on customer data bucket
   
   Medium-term (30 min-4 hours):
     1. Full forensic analysis of CloudTrail
     2. Determine if data exfiltration occurred
     3. Contact AWS security team
     4. Prepare breach notification if necessary
   ```

4. **What persistence did the attacker create?**
   - New access key (can access account even if password reset)
   - AdminAccess policy (high-privilege access)
   - If not caught: Backdoor account for future access

5. **How do you prevent this next time?**
   - MFA on all IAM users (this attack requires password theft)
   - Alert on any policy attachment outside business hours
   - Alert on access from unusual geographies
   - Disable unused access keys
   - Regular access reviews

### **Solutions:**

**Immediate Containment:**
```bash
# AWS CLI commands to respond
aws iam delete-access-key --access-key-id AKIAIOSFODNN7EXAMPLE --user-name dev-user
aws iam detach-user-policy --user-name dev-user --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
aws iam create-login-profile --user-name dev-user --password <new-temp-password> --password-reset-required
```

**Investigation:**
```
Review CloudTrail for:
  - Timeline: When did attacker first access?
  - Scope: How many users/resources affected?
  - Data: What was downloaded?
  - Persistence: What backdoors created?
```

**Communication:**
- Notify: CTO, Security team, Legal, Customer support
- If data breach: Prepare customer notifications
- If compliance data (PII, HIPAA, PCI): Notify regulators

---

## 🎯 Interview Questions You Might Get

### **Level 1 (Entry-Level):**

**Q1:** "Explain the shared responsibility model in cloud security"
- **Expected Answer:**
  - Cloud provider secures infrastructure (data centers, hypervisors, networks)
  - Customer secures configurations, IAM, data protection, applications
  - Example: AWS secures EC2 hardware, you secure EC2 OS patching
  - Different for managed services: AWS RDS = AWS handles OS; you handle database configuration

**Q2:** "What's CloudTrail and why does a SOC need it?"
- **Expected Answer:**
  - CloudTrail = AWS audit logging service
  - Logs ALL API calls made to AWS account
  - Essential for: Detecting unauthorized access, forensics, compliance
  - Without it: No visibility into who did what in AWS

**Q3:** "You discovered $50K in unexpected AWS charges. What do you do?"
- **Expected Answer:**
  - Don't ignore it—could be attack (cryptominer, resource hijacking)
  - Check CloudTrail for unusual API calls
  - Review EC2 instances (are there unknown ones?)
  - Review S3/RDS usage (unusual data transfer?)
  - Investigate and terminate malicious resources

---

### **Level 2 (Mid-Level):**

**Q1:** "Walk me through detecting an IAM credential compromise in cloud"
- **Expected Answer:**
  - Enables: CloudTrail, VPC Flow Logs, GuardDuty (AWS threat detection)
  - Looks for: Access from unusual IP, unusual API calls, privilege escalation attempts
  - Timeline: When were credentials exposed? How long was attacker active?
  - Impact: What resources accessed? What data touched?
  - Response: Disable credentials immediately, audit all activity, implement MFA

**Q2:** "What's the difference between detecting an attack on-premise vs. cloud?"
- **Expected Answer:**
  - On-Premise: Direct server access, firewall logs, endpoint logs
  - Cloud: API logs (CloudTrail), network logs (VPC Flow Logs), depends on configuration
  - Challenge: Less visibility by default—must enable logging
  - Advantage: Centralized cloud provider logs easier to correlate
  - Forensics: On-premise = memory dump; Cloud = preserve snapshots

**Q3:** "Design a detection rule for credential exposure in GitHub"
- **Expected Answer:**
  - Monitor: GitHub repository pushes for credential patterns (regex for AWS keys)
  - Alert: If AWS keys detected in commit
  - Response: Immediately revoke exposed keys
  - Integrate: GitHub secret scanning + AWS CloudTrail monitoring
  - Prevention: Secret management (AWS Secrets Manager, HashiCorp Vault)

---

### **Level 3 (Senior / Advanced Interview):**

**Q1:** "A developer's credentials were compromised. Walk me through how to determine the scope of the breach in a cloud environment"
- **Expected Answer:**
  - CloudTrail analysis: All API calls made with compromised credentials
  - Timeline reconstruction: When exposed → when used → when revoked
  - Resource inventory: What EC2, S3, RDS, databases accessed?
  - Data exposure: Analyze CloudTrail to see what data was downloaded
  - Persistence check: Did attacker create backdoors? New IAM users? Access keys?
  - Network analysis: VPC Flow Logs to see unusual traffic
  - Multi-cloud analysis: If hybrid, check all cloud environments
  - Forensic imaging: Snapshot affected instances for deep analysis

**Q2:** "Your organization is moving from on-premise to cloud. How do your incident response processes change?"
- **Expected Answer:**
  - Log sources change: SIEM still central, but feed from CloudTrail, VPC Flow Logs
  - Visibility: Logging isn't automatic—must enable
  - Forensics: Snapshot-based rather than direct disk access
  - Isolation: Network isolation via security groups (not physical isolation)
  - Recovery: AMI snapshots rather than tape backups
  - Shared responsibility: Some incidents are now AWS's problem, not yours
  - Compliance: Different audit requirements (SOC 2, FedRAMP, etc.)
  - Communication: Need cloud provider's incident response contacts

**Q3:** "Explain how you'd detect and respond to an advanced persistent threat (APT) in a hybrid infrastructure"
- **Expected Answer:**
  - Detection layers:
    - Endpoint EDR (on-premise + cloud VMs)
    - Network monitoring (internal + cloud network)
    - Cloud logs (CloudTrail, VPC Flow Logs)
    - Application logs
    - Behavioral analytics (unusual access patterns)
  - APT characteristics to hunt:
    - Lateral movement across hybrid environment
    - Living off the Land (using legitimate tools)
    - Persistence mechanisms (cloud backdoors + on-premise)
    - Data staging for exfiltration
  - Response:
    - Incident response must coordinate on-premise + cloud teams
    - Containment: Isolate both on-premise AND cloud systems
    - Eradication: Remove persistence from all environments
    - Forensics: Collect evidence from both on-premise and cloud

---

## 🔗 How This Connects To Everything Else

- **Incident Response:** Most incidents now involve cloud. IR playbooks must cover cloud-specific steps.
- **SIEM & Log Analysis:** CloudTrail logs must be ingested into your SIEM for correlation with on-premise logs.
- **Forensics:** Cloud forensics different (snapshots vs. direct access). Chain of custody matters.
- **Compliance:** Different standards for cloud (SOC 2, AWS compliance certifications).
- **IAM/Access Control:** Cloud IAM is foundation. Weak IAM = entire account compromise.
- **Network Security:** VPC isolation, security groups, WAF (Web Application Firewall) for cloud apps.
- **Data Protection:** Encryption at rest + in transit in cloud is YOUR responsibility (not AWS's by default).
- **Malware Analysis:** Malware in cloud = different analysis (can't directly access infected instance).
- **Threat Hunting:** Threat hunt in cloud using CloudTrail, not file system analysis.

---

## 💾 TL;DR For Busy People

| Concept | Definition | Detection Method | Response |
|---|---|---|---|
| **Shared Responsibility** | AWS secures infrastructure, you secure configuration | Understand boundary of who's responsible | Different incident response playbooks |
| **IAM Compromise** | Attacker obtains AWS credentials | CloudTrail shows API calls from unknown IP | Disable credentials, audit all activity |
| **CloudTrail** | AWS audit logging service | Enable CloudTrail in all regions | Correlate logs in SIEM |
| **VPC Flow Logs** | Network traffic logging in cloud | Enable on all subnets | Detect data exfiltration, unusual connections |
| **S3 Misconfiguration** | Bucket left publicly accessible | AWS Config checks bucket policies | Restrict access, audit what was exposed |
| **Credential Exposure** | Access keys found in source code | GitHub secret scanning + CloudTrail | Revoke keys immediately, rotate |
| **Cost Spike** | Unexpected AWS bill increase | Monitor monthly cost vs. baseline | Investigate for cryptominer, resource hijacking |
| **Logging Not Enabled** | Cloud events not recorded | Check if CloudTrail/Flow Logs enabled | Enable immediately, can't forensics without logs |
| **Privilege Escalation** | Attacker gains admin access | CloudTrail shows iam:AttachUserPolicy | Revoke permissions, audit all users |

---

## 📌 Production Reality

### **In a Real SOC with Hybrid Infrastructure:**

**Day 1: Initial Setup**
1. **On-Premise SIEM:** Receives Windows, Linux, firewall logs
2. **AWS Account Created:** Development team deploys application
3. **First Problem:** SOC doesn't know about AWS yet (communication gap)
4. **Action Needed:** Set up CloudTrail, feed to SIEM

**Day 30: First Cloud Alert**
1. **Detection:** CloudTrail shows unusual API calls
2. **Investigation:** Was this authorized? Check with dev team.
3. **Often:** "Oh yeah, we deployed that at 3 AM"
4. **Lesson:** Need better communication between SOC + dev teams

**Day 90: First Cloud Incident**
1. **Alert:** S3 bucket left public with sensitive data
2. **Discovery:** Via security researcher, not your SOC
3. **Media Coverage:** "Company Exposes Customer Data"
4. **Consequence:** Implement AWS Config rules, regular audits

**Day 180: Mature Hybrid Security**
1. **Centralized Logging:** All on-premise + cloud logs in SIEM
2. **Automated Detection:** Rules for cloud-specific threats
3. **Regular Audits:** CloudTrail analysis, IAM reviews, S3 bucket checks
4. **Incident Response:** Playbooks for cloud incidents
5. **Compliance:** Audit trails for SOC2, etc.

### **Real Challenges Your SOC Will Face:**

1. **Log Volume:** CloudTrail = millions of events. SIEM must scale.
2. **Cost:** AWS bills increase if lots of logging. SIEM license increases.
3. **Skill Gap:** Not all SOC analysts understand cloud-specific threats.
4. **Shared Responsibility Confusion:** Developers think AWS will protect them.
5. **Detection Lag:** Cloud logs sometimes delayed minutes. Need real-time alternatives.

---

## 📚 Further Reading

### **AWS-Specific:**
- AWS CloudTrail Documentation: https://docs.aws.amazon.com/cloudtrail/
- AWS IAM Best Practices: https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html
- AWS Security Best Practices: https://aws.amazon.com/architecture/security-identity-compliance/

### **Azure-Specific:**
- Azure Activity Log: https://learn.microsoft.com/en-us/azure/azure-monitor/essentials/activity-log
- Azure Security Best Practices: https://learn.microsoft.com/en-us/security/

### **Google Cloud:**
- Google Cloud Audit Logs: https://cloud.google.com/logging/docs/audit
- Google Cloud Security: https://cloud.google.com/security

### **General Cloud Security:**
- Cloud Security Alliance: https://cloudsecurityalliance.org/
- NIST Cloud Computing Security: https://csrc.nist.gov/projects/cloud-computing/
- OWASP Cloud-Native Security: https://owasp.org/

### **Tools to Learn:**
- AWS CLI (command-line access to AWS)
- CloudTrail Insights (AWS-built threat detection)
- GuardDuty (AWS threat detection)
- AWS Config (compliance monitoring)
- VPC Flow Logs (network monitoring)
- Terraform (Infrastructure as Code - cloud)

### **Your Next Steps:**
1. Request AWS sandbox environment from your organization
2. Enable CloudTrail on all AWS accounts
3. Create detection rules for: IAM compromise, resource hijacking, data exfiltration
4. Practice: AWS incident response playbooks
5. Study: Shared responsibility model for each cloud provider
6. Implement: Least privilege IAM policies
7. Monitor: Cost anomalies as detection signal

---

**Document Generated:** July 2026  
**For:** Cybersecurity Blue Team Portfolio (GitHub)  
**Level:** Entry to Mid-Level Analyst  
**Status:** Ready for Portfolio Use ✅
