<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:0F2027,100:00D4FF&height=200&section=header&text=Blue%20Team%20Knowledge%20Base&fontSize=40&fontColor=ffffff&animation=fadeIn&desc=Personal%20Cybersecurity%20Wiki%20%7C%20Blue%20Team%20%7C%20Incident%20Response%20%7C%20DFIR&descAlignY=65&descSize=18" width="100%"/>
</p>

<div align="center">

![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)
![Last Updated](https://img.shields.io/badge/Last%20Updated-2026-00D4FF?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active%20Learning-brightgreen?style=flat-square)

</div>

---

## 📖 About This Knowledge Base

A comprehensive, professionally-organized **personal cybersecurity wiki** documenting Blue Team concepts, incident response methodologies, Windows internals, networking fundamentals, threat detection strategies, and DFIR techniques.

This is not a book. This is a **living resource** built through real investigations, lab experience, and continuous learning. Every concept connects to others. Every section includes practical detection strategies and examples from actual incident response cases.

**Why this exists:** To transform hands-on learning into actionable, interconnected knowledge that bridges the gap between theory and practice.

---

## 🗂️ Knowledge Base Structure

### **[01 - Incident Response](./01%20Incident%20Response)**
Core IR methodologies and NIST framework phases.
- NIST Incident Response Lifecycle
- Detection Phase
- Analysis Phase
- Containment Strategies
- Eradication & Recovery
- Post-Incident Lessons Learned
- Root Cause Analysis Methodology

### **[02 - Windows Internals](./02%20Windows%20Internals)**
Deep dive into Windows architecture and critical components.
- Process Management
- LSASS (Local Security Authority Subsystem Service)
- Winlogon Process
- Registry Structure & Hives
- Windows Services Architecture
- Scheduled Tasks Execution
- Windows Event Logging

### **[03 - Active Directory](./03%20Active%20Directory)**
Enterprise authentication and access control backbone.
- Kerberos Authentication Protocol
- NTLM Legacy Authentication
- Domain Controllers
- Group Policy Objects (GPO)
- LDAP Protocol & Queries
- Service Accounts & Delegation
- Trust Relationships

### **[04 - Networking](./04%20Networking)**
Network protocols and communication fundamentals.
- DNS (Domain Name System)
- DHCP (Dynamic Host Configuration)
- TCP/IP Protocol Suite
- UDP vs TCP
- HTTP/HTTPS Protocols
- TLS/SSL Encryption
- Network Traffic Analysis

### **[05 - Threat Detection](./05%20Threat%20Detection)**
Detection engineering and threat hunting methodologies.
- Detection Fundamentals
- Sigma Rules
- YARA Patterns
- Alert Correlation
- False Positive Reduction
- Hunting Methodologies
- Indicator of Compromise (IOC) Development

### **[06 - DFIR](./06%20DFIR)**
Digital Forensics and Incident Response procedures.
- Memory Analysis with Volatility
- Disk Forensics
- Timeline Construction
- Evidence Handling & Chain of Custody
- Artifact Analysis
- Log Parsing & Correlation
- Forensic Reporting

### **[07 - MITRE ATT&CK](./07%20MITRE%20ATT&CK)**
One document per tactic with detection strategies.
- Reconnaissance
- Resource Development
- Initial Access
- Execution
- Persistence
- Privilege Escalation
- Defense Evasion
- Credential Access
- Discovery
- Lateral Movement
- Collection
- Command & Control
- Exfiltration
- Impact

### **[08 - SIEM](./08%20SIEM)**
Security Information and Event Management platforms.
- Elasticsearch & Kibana
- Splunk Fundamentals
- Log Ingestion & Parsing
- Search & Query Syntax
- Dashboard Design
- Alert Rules & Automation
- SIEM Best Practices

### **[09 - Linux](./09%20Linux)**
Linux administration and hardening for security.
- File System & Permissions
- User & Group Management
- Process Management
- Service Architecture
- SSH Hardening
- Firewall Configuration (iptables/firewalld)
- Log Analysis (/var/log)
- Linux Artifact Forensics

### **[10 - Malware](./10%20Malware)**
Malware behavior, persistence, and techniques.
- Persistence Mechanisms
- Privilege Escalation Techniques
- Lateral Movement Tactics
- Defense Evasion Methods
- Data Exfiltration
- Malware Signatures & Detection
- Reverse Engineering Basics

### **[11 - Cloud Security](./11%20Cloud%20Security)**
Cloud infrastructure security and incident response.
- AWS Security Fundamentals
- Azure Security Model
- Cloud Logging (CloudTrail, etc.)
- IAM & Access Control
- Cloud-Specific Attack Vectors
- Container Security
- Kubernetes Security

### **[12 - Detection Engineering](./12%20Detection%20Engineering)**
Building robust detection capabilities from scratch.
- Detection Philosophy
- Threat Modeling
- Data Requirements
- Detection Logic Design
- Testing Detections
- Tuning & Optimization
- Metrics & KPIs

### **[13 - Cheat Sheets](./13%20Cheat%20Sheets)**
Quick reference guides for common tools and commands.
- PowerShell Commands
- Bash One-Liners
- Wireshark Filters
- Event ID Reference
- MITRE ATT&CK Techniques Quick Index
- Tool Installation & Configuration

### **[14 - Labs Notes](./14%20Labs%20Notes)**
Documentation from hands-on lab exercises and investigations.
- Lab Setup Guides
- Investigation Walkthroughs
- Lessons Learned
- Tool Configuration Examples
- Real-World Scenario Walkthroughs

### **[15 - Interview Questions](./15%20Interview%20Questions)**
Common SOC & Blue Team interview topics with answers.
- Blue Team Fundamentals
- Incident Response Scenarios
- Windows Internals Q&A
- Active Directory Deep Dives
- DFIR Investigations
- Detection Engineering Concepts
- Behavioral Interview Questions

---

## 📋 Document Template

Every document in this knowledge base follows the same professional structure for consistency and ease of navigation:

```markdown
# [Topic Name]

## Definition
Clear, concise explanation of the concept. Written in your own words, not copied.

---

## Why it Matters
Real-world context. Why should a SOC Analyst care about this?

---

## How it Works
Mechanics and flow. Architecture if applicable.

---

## Common Attacks
Threat actors abuse this concept how? Real-world examples.

---

## Detection
How do you catch attacks leveraging this? Event IDs, log sources, indicators.

---

## MITRE ATT&CK
Related techniques from the MITRE framework. Tactics and sub-techniques.

---

## Real Investigation Example
Example from QuantiaPay, NorthBridge, or other actual investigations.

---

## Related Windows Events / Log Sources
Specific Event IDs or log files you should monitor.

---

## Related Topics
[Topic A](link) → [Topic B](link) → [Topic C](link)

This creates mental connections and deeper understanding.

---

## References
- [Microsoft Official Documentation](link)
- [MITRE ATT&CK Framework](link)
- [SANS Pen Testing](link)
```

---

## 🎯 Core Principles

### ✅ **Keep It Concise**
Maximum two pages per document. Quality over quantity. You're writing professional notes, not a textbook.

### ✅ **Write It In Your Own Words**
Read the source. Close the page. Explain it from memory. If you can't, you didn't understand it. This isn't plagiarism—it's mastery.

### ✅ **Connect Everything**
Every topic links to related concepts. Your brain learns through associations, not isolated facts. This builds reasoning, not memorization.

### ✅ **Use One Visual Per Document**
A single diagram, flowchart, or ASCII art beats twenty paragraphs of explanation.

### ✅ **Ground It in Reality**
Every concept includes:
- A real attack example
- Detection strategy
- Event IDs or log sources
- MITRE ATT&CK mapping

---

## 🚀 How to Use This Knowledge Base

### 1. **For Active Learning**
Pick a topic you're studying. Read the document once. Close it.
Teach someone else. If you can't explain it, the document wins.

### 2. **For Interview Prep**
Browse [Interview Questions](./15%20Interview%20Questions). Link back to the full documents for deeper study.

### 3. **For Incident Response**
Quick-reference during active investigations. Example: "What event IDs indicate lateral movement?"
Check [05 - Threat Detection](./05%20Threat%20Detection) or [04 - Networking](./04%20Networking).

### 4. **For Building Detection Logic**
Reference [12 - Detection Engineering](./12%20Detection%20Engineering) + specific threat topics.
Connect MITRE techniques → detection queries → validation.

### 5. **For Certification Prep**
Use [Cheat Sheets](./13%20Cheat%20Sheets) for quick facts. Reference full documents for conceptual depth.

---

## 🔗 Quick Navigation

| Need | Go To |
|------|-------|
| How does Kerberos work? | [03 - Active Directory](./03%20Active%20Directory) |
| What events indicate lateral movement? | [05 - Threat Detection](./05%20Threat%20Detection) |
| Memory analysis walkthrough? | [06 - DFIR](./06%20DFIR) |
| Pass the Ticket attack details? | [07 - MITRE ATT&CK](./07%20MITRE%20ATT&CK) → Lateral Movement |
| Windows forensics? | [02 - Windows Internals](./02%20Windows%20Internals) |
| Elasticsearch query examples? | [08 - SIEM](./08%20SIEM) |
| Linux hardening checklist? | [09 - Linux](./09%20Linux) |

---

## 📚 Learning Methodology

This knowledge base is built on a proven learning framework:

```
Study (1 hour) → Understand (concepts, not facts)
           ↓
Write in Own Words (20 min) → Force comprehension
           ↓
Connect to Other Topics (10 min) → Build mental models
           ↓
Document Detection Method (10 min) → Practical application
           ↓
Real-World Example (10 min) → Behavioral grounding
           ↓
Result: Deeper learning in 70 minutes than 3 hours of passive reading
```

---

## 🎓 Real-World Grounding

Every document in this knowledge base connects to actual investigations:

- **QuantiaPay Incident Response** — Cloud CI/CD compromise
- **NorthBridge Manufacturing** — Lateral movement investigation
- **NorthBay Logistics** — SOC alert correlation
- **Liberty & Co. Assessment** — Enterprise security assessment

When we document Kerberos attacks, we reference how they appeared in these investigations. Theory meets practice.

---

## 🔐 SOC Analyst Outcome

The ultimate goal: When someone asks **"In which NIST phase does Root Cause Analysis occur?"**

Your brain doesn't recall a slide. It traces the connection:

```
NIST Framework
    ↓
Analysis Phase
    ↓
Determine Root Cause
    ↓
Collect Evidence
    ↓
Build Timeline
    ↓
Document Lessons Learned
    ↓
Create Recommendations
```

**That's reasoning, not memorization.**

That's what makes a SOC Analyst.

---

## 📈 Content Roadmap

- [x] Incident Response Fundamentals
- [x] Windows Internals Core Concepts
- [x] Active Directory Authentication
- [x] Networking Protocols
- [ ] Threat Detection Strategies (In Progress)
- [ ] DFIR Procedures (In Progress)
- [ ] MITRE ATT&CK Tactics (In Progress)
- [ ] SIEM Implementation (Planned)
- [ ] Linux Security (Planned)
- [ ] Malware Analysis (Planned)
- [ ] Cloud Security (Planned)
- [ ] Detection Engineering (Planned)
- [ ] Cheat Sheets (Planned)
- [ ] Labs Notes (Ongoing)
- [ ] Interview Q&A (Planned)

---

## 🤝 Contributing

This is a personal knowledge base, but if you find errors or have suggestions:

1. **Found an issue?** Open an issue with corrections.
2. **Have a better explanation?** Submit a pull request.
3. **Missing a topic?** Suggest it in discussions.

---

## 📄 License

MIT License — Feel free to fork, reference, and build upon this knowledge base.

---

## 📞 Connect

- **LinkedIn:** [leonardo-romero-garcia](https://www.linkedin.com/in/leonardo-romero-garcia-086798417)
- **GitHub:** [ravesnoopy](https://github.com/ravesnoopy)
- **Discord:** leonardoyyy

---

## 🎯 Final Note

> "The best way to learn is to teach. The best way to teach is to document.  
> The best documentation is the one you'll actually use."

This knowledge base exists because memorization fades. Understanding persists.

Every concept here is built for retention, application, and mastery.

Welcome to the Blue Team Knowledge Base.

---

<div align="center">

**Last Updated:** 2026  
**Status:** 🟢 Active & Growing  
**Sections Completed:** 2 / 15

Made with 🔵 for the Blue Team

</div>

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:00D4FF,100:0F2027&height=100&section=footer" width="100%"/>
