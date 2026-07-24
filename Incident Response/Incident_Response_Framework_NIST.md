# Incident Response Framework (NIST SP 800-61)

## 📖 What Is This? (In 30 Seconds)

The Incident Response Framework is the structured methodology that organizations follow when a security incident occurs. NIST SP 800-61 defines **four phases** that guide investigators from the moment an incident is detected until lessons are learned and systems are restored. Think of it as the "playbook" that tells you what to do, in what order, and when to escalate.

---

## 🎯 Why Does A SOC Analyst Need This?

### In Job Interviews, You'll Hear:
- "Walk me through the incident response process from detection to recovery"
- "What's the difference between containment and eradication?"
- "When would you escalate? To whom?"
- "How do you preserve evidence while stopping an attack?"
- "What's your methodology for investigating incidents?"

### In Your First Month at a SOC:
- You'll receive an alert and think: "Now what?"
- Your SIEM fires. You follow the framework.
- You'll handle containment decisions that affect the business
- You'll preserve evidence while responding to attacks
- You'll report findings to management using this structure
- You'll participate in post-incident reviews
- **Getting this wrong = losing evidence, spreading infection, or missing the attacker**

---

## 🔍 The Four Phases (Expanded)

### **PHASE 1: PREPARATION**

#### What It Is:
Before an incident happens, you prepare. This is the "peacetime" work that determines if you'll succeed when crisis hits.

#### What You Do:
- **Build the team:** Train analysts, define roles, create escalation paths
- **Deploy tools:** SIEM, endpoint detection, logging infrastructure, firewalls
- **Create playbooks:** Step-by-step procedures for common attacks
- **Establish policies:** How to preserve evidence, who has authority to make decisions
- **Test everything:** Run incident response drills, test backups, test communication channels
- **Document baselines:** Know what "normal" looks like in your network

#### Why It Matters:
If you're not prepared, when an incident happens you'll be improvising. Attackers don't wait while you figure out your process.

#### Example in Real SOC:
- Monday: You install a new SIEM and configure alerting rules
- Tuesday: You create a playbook: "If we see 1000+ failed logins from one source, do X"
- Thursday: Alert fires. You follow your playbook. It works.

#### Red Flag:
If your organization hasn't prepared, incidents take 3x longer to resolve.

---

### **PHASE 2: DETECTION & ANALYSIS**

#### What It Is:
An incident is detected (alert, user report, security tool) and you determine what actually happened.

#### What You Do:

**DETECTION (Someone notices something is wrong):**
- Security tool generates alert
- System administrator reports suspicious behavior
- User reports they can't access their account
- Firewall blocks suspicious traffic
- Antivirus quarantines malware

**ANALYSIS (You investigate to confirm):**

1. **Triage the alert**
   - Is this a true positive or false positive?
   - How critical is it? (Scale: Low/Medium/High/Critical)
   - Does it match a known attack pattern?

2. **Gather initial evidence**
   - Collect logs from the source system
   - Get network traffic captures
   - Document timeline (when did this start?)
   - Identify affected assets (what got hit?)
   - Identify affected users (who was involved?)

3. **Determine scope**
   - Is this isolated to one machine or network-wide?
   - How many users are affected?
   - How many systems are compromised?
   - How long has this been happening?

4. **Classify the incident**
   - **Malware?** Use antivirus signatures and behavior analysis
   - **Unauthorized access?** Check access logs and privilege escalations
   - **Data exfiltration?** Monitor network traffic volumes to external IPs
   - **Denial of Service?** Analyze traffic patterns
   - **Insider threat?** Review user behavior and data access

5. **Map to MITRE ATT&CK**
   - Which techniques is the attacker using?
   - This helps you predict what they'll do next

#### Critical Decisions in This Phase:
- **Should we escalate?** (Yes, if confirmed malicious)
- **Should we preserve evidence?** (Yes, always)
- **Should we isolate the system?** (Not yet—wait for Phase 3)

#### Common Mistakes:
- ❌ Acting too fast without confirming it's real
- ❌ Not documenting what you find
- ❌ Deleting logs before reviewing them
- ❌ Assuming one alert = complete picture

#### Example in Real SOC:

**10:35 AM** — SIEM Alert: "20 failed login attempts on CORP-SQL-SERVER from 192.168.1.105"

**Your actions:**
1. Check: Is 192.168.1.105 a known server? (No = suspicious)
2. Collect: What user was being targeted? (SA_Admin = very suspicious)
3. Timeline: When did this start? (9:47 AM = 48 minutes of attacks)
4. Scope: Are other servers under attack? (Check logs = Yes, 3 more servers)
5. Classify: This looks like credential brute force or password spraying
6. Map to MITRE: T1110 (Brute Force) or T1110.003 (Password Spraying)
7. Decision: **ESCALATE TO SENIOR ANALYST** — This is confirmed malicious

---

### **PHASE 3: CONTAINMENT, ERADICATION, & RECOVERY**

This phase has **three sub-phases** that happen in order. Understanding the differences is critical.

#### SUB-PHASE 3A: CONTAINMENT

#### What It Is:
**STOP THE BLEEDING.** Prevent the attacker from doing more damage while preserving evidence.

#### The Containment Strategy:
```
Short-term Containment (0-30 minutes):
↓
Prevent spread, maintain evidence, keep system running if possible

Medium-term Containment (30 minutes - hours):
↓
Isolate compromised systems, block attacker communication

Long-term Containment (hours - days):
↓
Strengthen defenses, monitor for persistence
```

#### How Containment Works:

**Step 1: Isolate the affected system**
- Disconnect from network (but keep powered on to collect live memory)
- OR change network policies to restrict movement
- Goal: Attacker can't spread to other systems

**Step 2: Preserve evidence**
- Collect memory dump (RAM contains active connections, processes)
- Copy hard drive before making changes
- Document what you see (screenshots, logs, timestamps)
- Chain of custody: Log who touched what and when

**Step 3: Block attacker communication**
- If attacker has shell access, kill the connection
- Block C2 server IPs at firewall
- Reset credentials so attacker can't log back in
- BUT: Don't do this yet if you're still investigating

**Step 4: Maintain system integrity**
- Don't restart the system yet (might lose evidence)
- Don't delete files or logs
- Don't patch vulnerabilities yet (might hide how they got in)
- **Keep the system as-is for forensics**

#### Critical Decision in Containment:
**"How much damage can the attacker do RIGHT NOW?"**
- High damage risk? → Aggressive containment (disconnect system)
- Low damage risk? → Gentle containment (restrict network, keep running)

#### Example in Real SOC:

**11:15 AM** — Confirmed: CORP-LAPTOP-203 is infected with trojan

**Your containment actions:**
1. ✅ Collect memory dump (while system is still running)
2. ✅ Disconnect from network (but keep powered on)
3. ✅ Block C2 server IP at firewall (stops new commands)
4. ✅ Force password reset for affected user
5. ✅ Document everything in incident ticket
6. ⏸️ Don't restart the system yet
7. ⏸️ Don't run antivirus cleanup yet

**Result:** Attacker is contained. Evidence is preserved. Investigation continues.

---

#### SUB-PHASE 3B: ERADICATION

#### What It Is:
**KILL THE INFECTION.** Remove the attacker's presence completely.

#### How Eradication Works:

**Step 1: Remove the malware**
- Run antivirus/malware scans
- Delete infected files
- Remove scheduled tasks or registry entries created by malware
- Uninstall backdoors or remote access tools

**Step 2: Close the vulnerability**
- Patch the software/OS where the attacker got in
- Change firewall rules to prevent re-entry
- Update access controls
- Example: If they exploited unpatched SQL Server, patch it NOW

**Step 3: Eliminate persistence mechanisms**
- Malware often creates backup ways to return
- Remove all persistence (backdoors, scheduled tasks, etc.)
- Check for rootkits or bootkit installations
- Verify all malware is gone (not just the main payload)

**Step 4: Verify eradication**
- Scan system again to confirm no traces remain
- Check logs for any remaining attacker activity
- Test that vulnerability is closed

#### Critical Difference: Containment vs. Eradication

| Containment | Eradication |
|-------------|------------|
| **Stops** the attack | **Removes** the attack |
| Preserves evidence | May destroy evidence (acceptable) |
| System stays compromised | System becomes clean |
| Reversible | Not reversible |
| Can happen quickly | Takes longer |
| Example: Disconnect laptop | Example: Run antivirus, patch, restart |

#### Common Mistakes:
- ❌ Erasing logs before understanding how they got in
- ❌ Patching before completing forensics
- ❌ Not checking for persistence (malware returns later)
- ❌ Assuming one reboot = fully cleaned

#### Example in Real SOC:

**3:30 PM** — Containment complete. Now eradicate.

**Your eradication actions:**
1. ✅ Run full antivirus scan → Detects and removes trojan
2. ✅ Search for persistence → Find scheduled task (malware.exe runs daily)
3. ✅ Delete the scheduled task
4. ✅ Check Windows registry → Remove malicious registry entries
5. ✅ Patch SQL Server vulnerability that was exploited
6. ✅ Scan again → No infections found
7. ✅ Document: "Eradication complete on [timestamp]"

**Result:** System is clean. Attacker's tools are gone. Vulnerability is patched.

---

#### SUB-PHASE 3C: RECOVERY

#### What It Is:
**BRING SYSTEMS BACK TO LIFE.** Restore systems to normal operations safely.

#### How Recovery Works:

**Step 1: Restore clean backups**
- Use backup from BEFORE the infection
- Restore user files, configurations, applications
- Verify backup integrity (no malware in backup)

**Step 2: Rebuild systems (if necessary)**
- If backup is too old or unavailable, rebuild from scratch
- Reinstall OS, patches, applications
- Restore data from clean backups only
- Take time—speed isn't more important than security

**Step 3: Restore access and connectivity**
- Reconnect system to network
- Restore network rules and firewall policies
- Restore user access and permissions
- BUT: Keep monitoring for suspicious activity

**Step 4: Validate full functionality**
- Does the system work normally?
- Can users access their files?
- Are all applications functional?
- Are there no error messages or performance issues?

**Step 5: Monitor closely post-recovery**
- Watch for signs of re-infection
- Monitor network traffic from recovered system
- Watch for unusual login attempts
- This monitoring lasts for days/weeks

#### Decision in Recovery:
**"Do we restore from backup or rebuild from scratch?"**
- Risk: Backup might contain malware (too risky, rebuild)
- Safe: Rebuild system, then restore only user data (slower but safer)

#### Example in Real SOC:

**5:00 PM** — Eradication confirmed. Time to recover.

**Your recovery actions:**
1. ✅ Check backup from March 15 (before infection on March 17)
2. ✅ Verify backup has no malware signatures
3. ✅ Restore system from clean backup
4. ✅ Reinstall latest patches (March security updates)
5. ✅ Restore user files from backup
6. ✅ Reconnect to network
7. ✅ User tests system → Everything works
8. ✅ Enable enhanced monitoring on this system for 30 days

**Result:** System is recovered. User is back to work. Security is maintained.

---

#### Why The Three Sub-Phases Matter:

```
Containment    →    Eradication    →    Recovery
(Stop it)           (Remove it)         (Restore it)
(Keep evidence)     (Destroy evidence)  (Normal operations)
(Urgent)            (Methodical)        (Careful)
```

If you skip or mess up a sub-phase:
- ❌ Skip containment? Attacker spreads to more systems
- ❌ Skip eradication? Malware returns after recovery
- ❌ Skip recovery? Systems don't work, users can't be productive

---

### **PHASE 4: POST-INCIDENT ACTIVITY (REVIEW & LESSONS LEARNED)**

#### What It Is:
After the incident is resolved, you pause to learn. What happened? Why? How do we prevent it next time?

#### What You Do:

**Step 1: Collect status reports from infrastructure team**
- Document all changes made during response
- Verify all systems are fully operational
- Confirm no systems are still compromised
- Ensure backups are running normally

**Step 2: Review new applications/tools deployed**
- Did we deploy any new security tools during response?
- Are they functioning correctly?
- Are they generating useful alerts?
- Document their performance

**Step 3: Conduct threat hunting (if necessary)**
- Are there signs this attacker was in the network longer than we thought?
- Did they access other systems we missed?
- Are there other compromises we haven't found yet?
- This is proactive searching, not reactive investigation

**Step 4: Hold post-incident meeting (Lessons Learned)**
- **What happened?** Factual timeline
- **Why did it happen?** Root cause
- **How did we respond?** What worked? What didn't?
- **What will we improve?** Specific action items
- **Who needs to know?** Management, compliance, other teams

**Step 5: Document recommendations**
- Policy changes needed
- New tools or upgrades
- Training needed for staff
- Firewall rules to add
- Monitoring rules to enhance
- Playbook updates

**Step 6: Update your playbooks**
- If this is a known attack type, improve the playbook
- What took too long? Streamline it
- What was unclear? Document it better
- What worked well? Make it standard

#### Critical Decisions in Post-Incident:
- **Do we need compliance reporting?** (PCI DSS, HIPAA, GDPR, etc.)
- **Do we need legal review?** (Potential data breach)
- **Do we need to notify users?** (Data was exposed)
- **What do we tell management?** (Business impact)

#### Example in Real SOC:

**March 18, 10:00 AM** — Incident is resolved. Time for lessons learned.

**Your post-incident actions:**
1. ✅ Collect infrastructure status → All systems operational
2. ✅ Review new firewall rules deployed → Working correctly, blocking C2 attempts
3. ✅ Threat hunting → Found 2 additional lateral movement attempts (contained them)
4. ✅ Post-incident meeting with team:
   - What happened: Trojan via phishing email
   - Why: User clicked link, downloaded malware
   - How we responded: Contained, eradicated, recovered (2 hours total)
   - What worked: SIEM alert caught it, containment was fast
   - What failed: Email filtering didn't catch it
5. ✅ Recommendations:
   - Strengthen email filtering rules
   - Mandatory phishing awareness training for all staff
   - Deploy additional endpoint protection
6. ✅ Update playbook: "Trojan Response" now includes email filtering bypass procedures

**Result:** Team learns, processes improve, next incident is handled faster.

---

## ⚙️ What You MUST Memorize

**The Four Phases (In Order):**
1. **PREPARATION** — Before incident (build tools, train team, create playbooks)
2. **DETECTION & ANALYSIS** — Incident happens (detect, confirm, investigate)
3. **CONTAINMENT, ERADICATION, RECOVERY** — Stop, remove, restore (in that order)
4. **POST-INCIDENT ACTIVITY** — After resolution (learn, improve, document)

**Key Memory Device:**
```
PREPARATION → DETECT → CONTAIN → ERADICATE → RECOVER → REVIEW

P → D → C → E → R → R

"Plan, Detect, Contain, Eradicate, Recover, Review"
```

**Sub-Phases (Important):**
- **Containment** = Stop the bleeding (preserve evidence)
- **Eradication** = Kill the infection (destroy evidence is OK)
- **Recovery** = Restore to normal (verify functionality)

---

## 📚 What You MUST Understand

- [ ] **Why order matters** — You can't eradicate before containing (attacker spreads)
- [ ] **Containment vs. Eradication** — Different goals, different actions, different timing
- [ ] **Evidence preservation** — Where and when it matters most (Containment phase)
- [ ] **Escalation points** — When to call senior analyst, manager, CEO, lawyers
- [ ] **Business impact** — Each phase affects how long users are without systems
- [ ] **Recovery planning** — Backups, rebuild procedures, testing protocols
- [ ] **Continuous improvement** — Post-incident review is how you get better

---

## 🚨 Real-World Application: How Phases Connect

### Scenario: Ransomware Infection

**PREPARATION (Happened last month):**
- Deployed ransomware detection tool
- Created ransomware response playbook
- Trained team on recovery procedures
- Tested backup restoration process

**DETECTION & ANALYSIS (Today 8:00 AM):**
- Antivirus detects ransomware on CORP-FINANCE-SVR
- Analyst confirms: 150 files encrypted, ransom note present
- Scope: Only 1 server affected (for now)
- Classification: Critical — Finance data is encrypted

**CONTAINMENT (Today 8:30 AM):**
- Disconnect server from network (stop ransomware spreading)
- Collect memory and disk for forensics
- Block attacker C2 communication at firewall
- Server stays isolated but powered on
- **Decision: Do NOT pay ransom**

**ERADICATION (Today 10:00 AM):**
- Run malware scans → Find and remove ransomware
- Search for persistence mechanisms → Find none
- Patch vulnerability that was exploited
- Verify infection is gone

**RECOVERY (Today 1:00 PM):**
- Restore server from backup (March 15, before infection)
- Verify backup has no ransomware
- Restore user files
- Run antivirus on entire server
- Reconnect to network
- Test functionality with finance team

**POST-INCIDENT (Tomorrow 9:00 AM):**
- Collect infrastructure status → All systems normal
- Threat hunting → No other ransomware found
- Post-incident meeting:
  - What worked: Fast detection, quick containment, successful recovery
  - What failed: Vulnerability wasn't patched (IT had it on to-do list)
  - How: Ransomware exploited unpatched Exchange Server
- Recommendations:
  - Patch management process needs automation
  - Backup testing should happen monthly, not quarterly
  - Ransomware training for end users
- Update playbook with lessons learned

**Total time: 24 hours (not days or weeks)**

---

## ❌ Common Mistakes Students Make

### Mistake 1: Thinking Phases Are Sequential Only
**Wrong:** "I finish Preparation, then move to Detection forever"
**Correct:** You're ALWAYS in Preparation (even while in other phases). Each incident makes Preparation better.

### Mistake 2: Confusing Containment With Eradication
**Wrong:** "Containment and eradication are the same thing"
**Correct:** Containment stops the attack (system stays compromised). Eradication removes the attack (system gets cleaned).
**Why it matters:** If you try to eradicate before containing, the attacker spreads while you're cleaning.

### Mistake 3: Deleting Evidence Too Early
**Wrong:** "After I reboot the system, the attack is gone"
**Correct:** Rebooting destroys evidence (memory is lost). Collect evidence FIRST, then reboot.
**Real consequence:** Compliance violations, can't prove what happened, attacker goes free.

### Mistake 4: Not Documenting During Response
**Wrong:** "I'll document everything after the incident is resolved"
**Correct:** Document WHILE responding. Timestamps, actions, findings matter.
**Why:** Memory fails. Incidents are chaotic. You'll forget details.

### Mistake 5: Thinking Recovery = Back to Work
**Wrong:** "System is restored, we're done"
**Correct:** Recovery means functional operations WITH monitoring. You're not "done" for weeks.
**Why:** Attacker might have persistence (backdoor). You need to watch for re-infection.

### Mistake 6: Skipping Post-Incident Review
**Wrong:** "The incident is over, time to move to the next alert"
**Correct:** Post-incident review is where you improve. Skip this and you'll handle the next incident the same way.
**Why:** Incidents are learning opportunities. Waste that opportunity = make the same mistakes again.

### Mistake 7: Not Understanding When to Escalate
**Wrong:** "I'll handle everything myself"
**Correct:** Some incidents need senior analysts, management, legal, or law enforcement.
**When to escalate:**
- ✅ Suspected data breach → Legal team
- ✅ Ransomware → Management + possibly law enforcement
- ✅ APT activity → Bring in threat intel team
- ✅ Critical infrastructure → Escalate to CISO
- ✅ Anything you're unsure about → Ask senior analyst

---

## 🎯 Interview Questions You Might Get

### Easy (L1 Knowledge)
**Q:** "What are the four phases of incident response?"
**A:** "Preparation, Detection & Analysis, Containment/Eradication/Recovery, and Post-Incident Activity."

**Q:** "What's the difference between containment and eradication?"
**A:** "Containment stops the attack from spreading while preserving evidence. Eradication removes the attacker's presence completely. You contain first, then eradicate."

**Q:** "Why is preparation important if we can't stop all incidents?"
**A:** "Preparation determines how fast and effectively we respond. Without it, we improvise during crises, which is slower and more error-prone."

### Medium (Show Understanding)
**Q:** "Walk me through how you'd respond to a confirmed malware infection on a critical server."
**A:** 
1. Detection & Analysis: Confirm the infection, document scope
2. Containment: Isolate the server, collect forensics, block C2
3. Eradication: Remove malware, patch vulnerability, verify it's gone
4. Recovery: Restore from clean backup, test functionality
5. Post-Incident: Review what happened, update playbooks

**Q:** "What's more important: containing quickly or preserving evidence?"
**A:** "Both matter, but the priority depends on business impact. If the attacker is spreading to critical systems, contain aggressively even if it means losing some evidence. If it's contained to one low-risk system, preserve evidence for thorough investigation."

**Q:** "What would you do if you discovered evidence of a data breach during post-incident review?"
**A:** 
1. Escalate to management immediately
2. Involve legal team for compliance obligations
3. Investigate how much data was accessed
4. Prepare for user notification if required by law
5. Work with legal on disclosure timeline

### Hard (Senior Analyst Level)
**Q:** "You're responding to an incident where you need evidence for potential prosecution. How does that change your containment strategy?"
**A:** "Chain of custody becomes critical. Every action must be documented—who touched what, when, why. I'd involve legal and possibly law enforcement early. Containment might be slower and more deliberate to preserve evidence integrity. I'd avoid actions that destroy evidence even if they'd speed up eradication."

**Q:** "Describe a time when you had to choose between fast recovery and thorough investigation. What did you do?"
**A:** "If this happened: [Specific example from your experience or lab]. Here's my decision-making process: [Explain priority, who I consulted, why you chose that approach]. This taught me that escalation to management matters—they decide business vs. security tradeoffs, not the analyst."

**Q:** "How do you know when an incident is truly eradicated?"
**A:** "Never 100%, but close: Multiple clean scans with current signatures, no persistence mechanisms found, vulnerability patched, monitoring for re-infection shows nothing suspicious for 48-72 hours. Even then, enhanced monitoring continues for weeks to catch late-stage persistence."

---

## 🔗 How This Connects To Everything Else

- **MITRE ATT&CK:** Techniques map to Detection & Analysis phase. You classify what happened.
- **Windows Event IDs:** Evidence you collect in Detection & Analysis and Containment phases.
- **Networking Concepts:** Understanding lateral movement helps in Containment (where to block).
- **Active Directory:** Most enterprise incidents involve AD. AD attacks span all four phases.
- **DFIR:** Forensics happens during Containment/Eradication (collect evidence) and Post-Incident (detailed analysis).
- **Threat Hunting:** Proactive hunting in Post-Incident phase finds threats before they alert.
- **SIEM:** Detects incidents (Phase 2), triggers playbooks automatically (Phase 3), feeds data to review (Phase 4).

---

## 💾 TL;DR For Busy People

| Aspect | Answer |
|--------|--------|
| **What is it?** | Four-phase framework for responding to security incidents (NIST SP 800-61) |
| **Why care?** | Every SOC job uses this. Every incident follows this order. |
| **The four phases:** | Preparation → Detection & Analysis → Containment/Eradication/Recovery → Post-Incident |
| **Remember this:** | Contain BEFORE eradicate. Preserve evidence during containment. Eradicate after contained. |
| **If you mess up:** | Attacker spreads, evidence is lost, incident takes weeks instead of hours |
| **How to use in job:** | When alert fires, ask "Which phase are we in?" and follow the playbook. |

---

## 📌 Production Reality: What It Actually Looks Like

**Monday 9:00 AM** — Alert fires. You're in Phase 2 (Detection & Analysis)
- SIEM says: "Possible credential stuffing attack"
- You investigate: "Yes, confirmed. 2,000 failed logins, 1 successful"
- Decision: Escalate to senior analyst

**Monday 9:30 AM** — Senior analyst joins. Now Phase 3 (Containment)
- "Reset the compromised password"
- "Monitor for further access"
- "Block the attacker IP at firewall"
- Evidence is collected but system stays running

**Monday 2:00 PM** — Phase 3 (Eradication)
- "Scan for persistence" → Nothing found
- "Patch the vulnerability they exploited"
- "Update firewall rule"

**Monday 5:00 PM** — Phase 3 (Recovery)
- User changed password (already did at 9:30)
- User confirmed they can access systems
- "Done"

**Tuesday 10:00 AM** — Phase 4 (Post-Incident)
- Team meeting
- "What was the root cause?" → Weak password policy
- "What do we improve?" → Require complex passwords, enable MFA
- Update password policy documentation

**Total time: ~25 hours (most of it is waiting + monitoring)**

---

## 📌 One More Thing: Your Role in Each Phase

**As an L1 SOC Analyst, you will:**

| Phase | Your Role | What You Don't Do |
|-------|-----------|------------------|
| **Preparation** | Test playbooks, learn tools | Create policy or hire |
| **Detection & Analysis** | Investigate, confirm, escalate | Make containment decisions alone |
| **Containment/Eradication/Recovery** | Execute procedures, document | Make major decisions (escalate to L3/management) |
| **Post-Incident** | Contribute to review, suggest improvements | Make policy changes alone |

---

## 📚 Further Reading & Resources

- **NIST SP 800-61 Rev. 2** — Official framework (Google it, it's free)
- **Related Windows Event IDs:** 4624, 4625, 4672, 4768, 4769, 4771
- **MITRE ATT&CK Techniques:** Impacts (Service Stop, Resource Hijacking, etc.)
- **Real incident writeups:** Check HackerNews, Bleeping Computer for detailed post-mortems
- **Practice:** Build a home lab and simulate incidents following this framework

---

*Last Updated: 2024*
*Difficulty: L1*
*Interview Relevance: ⭐⭐⭐⭐⭐*
*Job Applicability: Required Knowledge for All SOC Roles*
