# Linux for Blue Team & SOC Analysts

## 📖 What Is This? (In 30 Seconds)

Linux is an **open-source, Unix-based operating system** that is the backbone of security infrastructure in modern SOCs. Unlike Windows, Linux is built on principles of file permissions, process isolation, and detailed logging—making it essential for security investigations, running security tools, and securing servers. In Blue Team/SOC work, you WILL encounter Linux systems both as targets to investigate and as platforms where your security tools run.

**Key distinction:** Linux isn't "easy to edit"—rather, it's transparent. You can see exactly what's running, who has access, and what changed. This transparency is what makes it powerful for security.

---

## 🎯 Why Does A SOC/Blue Team Professional Need This?

### In Job Interviews, You'll Hear:
- "Explain how Linux permissions work and why they matter in security investigations"
- "How would you investigate a compromised Linux server? Walk me through the process"
- "What's the difference between a regular user and sudo? Why is this important?"
- "What can you learn from /var/log/auth.log?"
- "If a file has `rwxr-x---` permissions, who can read it? Who can execute it?"
- "How would you detect privilege escalation on a Linux system?"
- "Compare how you'd investigate a breach on Linux vs Windows"

### In Your First Month at a SOC:
- Your SOC will likely have Linux servers (web servers, databases, logging infrastructure)
- You'll need to SSH into these servers to investigate incidents
- You'll examine `/var/log/auth.log`, `/var/log/syslog`, and other logs for evidence
- You'll look for suspicious file permission changes, unexpected sudo usage, or unknown processes
- You'll need to understand process isolation, user permissions, and basic networking
- You'll run tools like `nmap`, `tcpdump`, `ss`, `netstat` to gather evidence
- **Getting this wrong = missing the attack, contaminating evidence, or misinterpreting logs**

---

## 🔍 Core Concepts Broken Down

### **CONCEPT 1: LINUX USERS AND PERMISSIONS (The Foundation)**

#### What It Is:
Linux is a **multi-user operating system**. Unlike your personal Windows PC (where you're the only user), Linux servers have:
- Root user (like Administrator in Windows)
- Regular users (with limited permissions)
- System users (for services like database engines, web servers)

Each user has a unique **UID (User ID)** and belongs to **groups** (like "admin", "sudo", "www-data").

#### Why This Matters in Security:

**Scenario 1: Attacker gains access**
- If attacker compromises a regular user account = limited damage (can't modify system files)
- If attacker gains root access = total compromise (can do anything)
- Your job in SOC = determine which user was compromised and what they could access

**Scenario 2: Privilege escalation**
- Attacker starts as regular user, exploits vulnerability to become root
- Evidence: `/var/log/auth.log` shows `sudo` usage or failed sudo attempts
- Your job = detect this escalation pattern

#### Real Example:

```
# User "john" tries to view system files (fails because no permission)
john@server:~$ cat /etc/shadow
cat: /etc/shadow: Permission denied

# User tries to use sudo (which allows temporary root access)
john@server:~$ sudo cat /etc/shadow
[sudo] password for john: 
root:*:19000:0:99999:7:::
mysql:!:19000:0:99999:7:::
```

In this example:
- `/etc/shadow` = file only root can read (stores password hashes)
- `john` can't read it as regular user
- `john` CAN read it using `sudo` (if john is in sudoers file)

**In a SOC investigation:**
- If you see "john" accessing `/etc/shadow` without sudo, that's a major red flag (compromise)
- If you see "john" using `sudo` 50 times at 3 AM, that's suspicious (lateral movement or escalation)

---

### **CONCEPT 2: FILE PERMISSIONS (How Security Works)**

#### What It Is:
Every file in Linux has permissions that define:
- **Who** can access it (owner, group, others)
- **What** they can do (read, write, execute)

#### How to Read Permissions:

```
-rwxr-x---  1  root  admin  4096  Mar 15 10:00  /etc/passwd

Breaking it down:
- = regular file (d = directory, l = symbolic link)
rwx = owner (root) can read, write, execute
r-x = group (admin) can read, execute (NOT write)
--- = others can do NOTHING
```

**The three permission types:**
- **r (read)** = can view file contents
- **w (write)** = can modify or delete file
- **x (execute)** = can run file as program (for directories: can enter directory)

#### Real Security Example:

```
-rw-r--r--  important_file.txt
= Owner can read/write, group can read, others can read
= SECURITY RISK: Anyone on the system can read this

-rw-------  password.txt
= Only owner can read/write, nobody else can access
= SECURE: Private file protected from other users
```

#### In SOC Investigations:

**Red Flag 1: Suspicious Permission Changes**
```
Normal:    -rw-r--r--  /var/www/html/index.php
Suspicious: -rwxrwxrwx  /var/www/html/index.php
            ^ Everyone can now read, write, execute!
```
This indicates an attacker likely modified the file to add a backdoor.

**Red Flag 2: Writable System Files**
```
Normal:    -rw-r--r--  /etc/passwd
Suspicious: -rw-rw-rw-  /etc/passwd
            ^ Non-root users can now modify user database!
```
This indicates privilege escalation—attacker is modifying system files.

---

### **CONCEPT 3: SUDO (Temporary Root Access)**

#### What It Is:
`sudo` = "superuser do" = allows regular users to run commands as root, but only if they're authorized (in `/etc/sudoers` file).

#### How It Works:

```bash
# Regular user john tries to restart apache
john@server:~$ service apache2 restart
Permission denied (you must be root)

# john uses sudo (if authorized)
john@server:~$ sudo service apache2 restart
[sudo] password for john: 
apache2 restarted
```

**In `/etc/sudoers` file:**
```
john ALL=(ALL) ALL
^ john can use sudo for ANY command as ANY user
^ This is dangerous! Usually more restrictive like:

john ALL=(ALL) /usr/sbin/service
^ john can only use sudo for /usr/sbin/service
```

#### Why This Matters in Security:

**Scenario: Attacker escalates privileges**
1. Attacker compromises regular user "john"
2. Attacker discovers john is in sudoers file
3. Attacker runs: `sudo -i` (becomes root)
4. Evidence in logs: `/var/log/auth.log` shows "john" used sudo

**Your SOC job:**
- Check who can use sudo: `cat /etc/sudoers`
- Review sudo usage logs: `grep sudo /var/log/auth.log`
- Flag suspicious patterns (e.g., failed sudo attempts = attacker guessing password)

#### Red Flags in Sudo Usage:

```
Mar 15 03:47:22 server sudo: john : TTY=pts/0 ; PWD=/home/john ; USER=root ; COMMAND=/bin/bash
^ 3 AM is suspicious for regular user
^ john ran bash as root (fully interactive shell)
^ MAJOR RED FLAG: Possible compromise

Mar 15 03:47:45 server sudo: john : 1 incorrect password attempt
^ Failed sudo password = attacker trying to escalate
^ 10+ failed attempts = dictionary attack on sudo
```

---

### **CONCEPT 4: LINUX LOGS (Where The Evidence Lives)**

#### Key Logs You'll Investigate:

**1. `/var/log/auth.log` (Authentication Log)**
- Every login, logout, sudo usage
- Failed login attempts (brute force detection)
- SSH connections, password changes

Example entry:
```
Mar 15 09:23:45 server sshd[5432]: Failed password for john from 192.168.1.100 port 22 ssh2
^ User "john" failed to login from IP 192.168.1.100
^ 10+ entries like this = brute force attack

Mar 15 09:25:12 server sshd[5433]: Accepted password for john from 192.168.1.100 port 22 ssh2
^ Same IP finally succeeded = compromised credentials
```

**2. `/var/log/syslog` or `/var/log/messages` (System Log)**
- System events, errors, warnings
- Service starts/stops
- Kernel messages

**3. `/var/log/secure` (RHEL/CentOS version of auth.log)**
- Similar to auth.log on Red Hat systems

**4. Application-Specific Logs:**
```
/var/log/apache2/access.log    (web server access)
/var/log/apache2/error.log     (web server errors)
/var/log/mysql/error.log       (database errors)
/var/log/nginx/access.log      (nginx web server)
```

#### In SOC Investigations:

```bash
# Find all failed login attempts
grep "Failed password" /var/log/auth.log | wc -l
# Result: 1247 failed attempts = brute force attack

# Find all sudo usage
grep "sudo" /var/log/auth.log
# Review to find suspicious sudo commands

# Find all SSH logins
grep "sshd" /var/log/auth.log | grep "Accepted"
# Identify all successful logins, check times/IPs for anomalies

# Find logins from specific IP
grep "192.168.1.100" /var/log/auth.log
# Track all activity from that source IP
```

---

### **CONCEPT 5: PROCESSES AND RUNNING SERVICES**

#### What It Is:
A **process** is a running program. Every process has:
- **PID** (Process ID)
- **Owner** (which user started it)
- **Parent process** (what started this process)
- **Command line** (arguments and flags)

#### Why This Matters in Security:

**Normal Process:**
```
www-data  5432  0.1  0.2  65536 8192  ?  S  10:00  0:01  /usr/sbin/apache2 -k start
^ Process owned by www-data user (not root)
^ Running legitimate apache2 web server
```

**Suspicious Process:**
```
root  6789  15.2  45.1  524288 450000  ?  S  03:47  1:23  /tmp/miner.sh
^ Running from /tmp (unusual location)
^ Using 45% of system memory (resource hogging)
^ Owned by root (compromised root privileges)
^ At 3:47 AM (suspicious timing)
^ LIKELY MALWARE (cryptocurrency miner)
```

#### Commands to Investigate Processes:

```bash
ps aux
# List all running processes with details

ps aux | grep apache
# Find all apache processes

netstat -tlnp
# Show all listening ports and which process owns them

ss -tlnp
# Modern version of netstat (more reliable)

top
# Real-time process monitor (see CPU, memory usage)

lsof -i :80
# Find what process is listening on port 80
```

---

## ⚙️ What You MUST Memorize

### File Permissions (chmod)

```
rwxrwxrwx
│││││││└─ Others can execute
││││││└── Others can write
│││││└─── Others can read
││││└──── Group can execute
│││└───── Group can write
││└────── Group can read
│└─────── Owner can execute
└──────── Owner can write and read
```

**Common Permissions:**
- `755` = rwxr-xr-x (owner: rwx, group+others: rx) - typical for executable scripts
- `644` = rw-r--r-- (owner: rw, group+others: r) - typical for files
- `700` = rwx------ (owner: rwx only) - private files
- `777` = rwxrwxrwx (everyone can do everything) - SECURITY RISK

### Sudo Escalation Pattern

```
1. Attacker compromises regular user
2. Attacker checks if user is in sudoers: sudo -l
3. If allowed, attacker runs: sudo -i (becomes root)
4. Evidence: auth.log shows sudo usage and commands
```

### Key Linux Directories

```
/home/          - User home directories (personal files)
/root/          - Root user's home directory
/etc/           - Configuration files (sudoers, passwd, shadow)
/var/log/       - Log files (where investigations start)
/tmp/           - Temporary files (often used for malware)
/usr/bin/       - Standard executable programs
/usr/sbin/      - System administration programs (require sudo)
```

### Critical Commands for SOC

```
# User and permission commands
id                          # Show current user info
whoami                      # Show current username
sudo -l                     # List what current user can sudo
cat /etc/sudoers            # View sudo permissions
cat /etc/passwd             # View all users
cat /etc/shadow             # View password hashes (root only)

# File commands
ls -la /directory           # List files with permissions
chmod 755 filename          # Change file permissions
chown owner:group filename  # Change file owner
find / -perm -4000         # Find setuid files (escalation risk)

# Process commands
ps aux                      # List all processes
netstat -tlnp              # Show listening ports and processes
ss -tlnp                   # Modern netstat alternative
lsof -i :PORT              # Show what's listening on port

# Log commands
grep "pattern" /var/log/auth.log    # Search for patterns
tail -n 100 /var/log/auth.log       # Show last 100 lines
grep "sudo" /var/log/auth.log       # Find sudo usage
grep "Failed" /var/log/auth.log     # Find failed logins

# Network commands
ip addr show                # Show network interfaces
ip route show              # Show routing table
ss -an                     # Show all connections
```

---

## 📚 What You MUST Understand

- [ ] **Linux is transparent by default** — You can see what's running, who owns it, and what changed (unlike Windows which hides things)
- [ ] **Permissions are not optional** — They're the ONLY thing preventing users from accessing each other's files
- [ ] **Sudo is a privilege escalation vector** — If attacker gets sudo, they get root; if they get root, they get everything
- [ ] **Logs are your investigation starting point** — `/var/log/auth.log` tells you who accessed what and when
- [ ] **Users and groups are security boundaries** — www-data user is restricted; root is unrestricted
- [ ] **File ownership matters** — A malicious file owned by root is more dangerous than one owned by regular user
- [ ] **Processes reveal compromise** — Unexpected processes, especially in /tmp or running as root, indicate attack
- [ ] **SSH keys replace passwords in servers** — Many Linux systems use SSH keys, not passwords (makes brute force impossible)

---

## 🚨 Real-World Application: A Linux Compromise Investigation

### Scenario: Webserver Hacked

**Initial Alert (7:00 AM):**
- IDS alerts on suspicious outbound traffic from web server
- Web server's CPU is at 95% (normally 10-15%)
- Disk I/O is extremely high

**Step 1: Investigate Processes (8:00 AM)**
```bash
ps aux | head -20
# You see:
# www-data  5432  87.3  42.1  524288 400000  ?  R  06:47  1:13  /tmp/xmrig
# ^ www-data user running crypto miner in /tmp
# ^ Using 87% CPU and 400MB RAM
# ^ Started at 6:47 AM (correlates with alert)
```

**Step 2: Check Network Connections (8:05 AM)**
```bash
ss -tlnp | grep 5432
# Shows connection to 192.168.1.50:3333
# ^ This is a known mining pool
# ^ Confirms malware is exfiltrating compute resources
```

**Step 3: Investigate How They Got In (8:15 AM)**
```bash
grep "www-data" /var/log/auth.log | tail -50
# You see:
# Mar 15 06:30:00 server sshd: Accepted password for www-data from 203.0.113.50
# ^ But www-data user shouldn't be able to SSH!
# ^ Someone enabled SSH for www-data or changed permissions
```

**Step 4: Check File Permissions (8:20 AM)**
```bash
ls -la /etc/sudoers.d/
# You see:
# -rw-r--r--  www-data
# ^ www-data can now run commands as root!
# ^ This is how attacker escalated privileges

ls -la /tmp/ | grep xmrig
# -rwxr-xr-x  www-data  /tmp/xmrig
# ^ Suspicious executable in /tmp (malware location)
```

**Step 5: Timeline Reconstruction**
```
6:30 AM - Attacker SSH into server as www-data (compromised password)
6:32 AM - Attacker uses sudo to add self to sudoers file
6:35 AM - Attacker downloads xmrig miner to /tmp
6:47 AM - Attacker starts miner (detected by IDS/CPU spike)
```

**Root Cause:** www-data password was weak and brute-forced.

**Your Response:**
1. Kill the miner process
2. Remove www-data from sudoers
3. Reset www-data password
4. Audit all files modified by www-data
5. Check if attacker accessed customer data
6. Patch web application vulnerability that led to password compromise

---

## ❌ Common Mistakes Students Make

### Mistake 1: Confusing "Code is Open" with "System is Easy to Change"
**Wrong:** "Linux is open source so I can just modify any file I want"
**Correct:** Linux source code is visible, but file permissions still prevent you from editing files you don't own
**Real consequence:** You can't modify `/etc/passwd` unless you're root, even though you can read the source code

### Mistake 2: Not Understanding Permission Numbers
**Wrong:** "777 and 755 are basically the same"
**Correct:** 777 = rwxrwxrwx (EVERYONE can write), 755 = rwxr-xr-x (only owner can write)
**Real consequence:** Using 777 on sensitive files = allowing any user to delete or modify them

### Mistake 3: Assuming Sudo = Just "Run as Root"
**Wrong:** "sudo is just for administrators to run commands"
**Correct:** sudo is how privilege escalation happens. Attacker gains regular user → uses sudo → becomes root
**Real consequence:** Missing sudo abuse in logs = missing the entire escalation chain

### Mistake 4: Not Checking Process Ownership
**Wrong:** "I'll kill the malware process and we're done"
**Correct:** Check who owns the process, where it came from, what it's connecting to
**Real consequence:** You kill the visible process but leave the backdoor, attacker re-runs it

### Mistake 5: Ignoring SSH Key Compromise
**Wrong:** "SSH logins are in auth.log, so I can detect them easily"
**Correct:** If attacker has SSH key, no password attempt is logged—they just appear as "Accepted publickey"
**Real consequence:** Missing SSH key compromise in your investigation

### Mistake 6: Thinking Windows Investigation Skills Transfer Directly
**Wrong:** "I investigated Windows, Linux must be similar"
**Correct:** Linux investigation relies on command-line logs, process trees, and permissions; Windows relies on Event Viewer, registry, and ACLs
**Real consequence:** Looking in wrong logs, misinterpreting evidence, wrong conclusions

---

## 🧪 Practice Scenario: Analyze This Compromise

### Scenario Data:

```
# User processes:
ps aux output:
root      1234  0.1  0.1  65536 8192   ?  S  09:00  0:01  /usr/sbin/sshd -D
john      5432  2.3  1.5  524288 12288 ?  S  10:47  0:23  /bin/bash
john      5433  87.4 45.0 1000000 400000 ? R  10:48  3:45  /usr/bin/python3 /tmp/miner.py
root      5434  0.1  0.2  131072 16384 ?  S  10:50  0:02  /usr/bin/scp

# File permissions:
ls -la /etc/sudoers.d/:
-rw-r--r-- root root    john
-rw-r--r-- root root    www-data

# Logs:
grep "john" /var/log/auth.log:
Mar 15 10:45:32 server sshd[4999]: Accepted password for john from 203.0.113.10 port 22 ssh2
Mar 15 10:46:15 server sudo: john : TTY=pts/0 ; PWD=/tmp ; USER=root ; COMMAND=/usr/bin/python3
Mar 15 10:47:23 server sudo: john : 1 incorrect password attempt ; TTY=? ; PWD=/tmp ; USER=root ; COMMAND=/bin/bash
Mar 15 10:47:45 server sudo: john : TTY=pts/0 ; PWD=/tmp ; USER=root ; COMMAND=/bin/bash

# Network:
ss -tlnp output:
tcp  LISTEN  5433   0.0.0.0:4444  * (python3)
tcp  LISTEN  5434   192.168.1.50:3333  * (scp)
```

### Questions:

1. **What happened?** Timeline the attack (when attacker got in, what they did, in what order)
2. **How did they escalate?** What method did attacker use to go from john→root?
3. **What's the malware?** What process should concern you most and why?
4. **What evidence is critical?** Which log entries prove the compromise?
5. **What's missing?** What would you want to investigate further?

### Answers:

**1. Timeline:**
- 10:45:32 - Attacker SSH into server as user "john" from external IP 203.0.113.10
- 10:46:15 - Attacker successfully runs sudo command as root (escalation succeeded)
- 10:47:23 - Attacker attempts to switch to bash as root (failed first attempt)
- 10:47:45 - Attacker successfully becomes root shell
- 10:48:00 - Attacker launches mining process (/tmp/miner.py) as root

**2. Escalation method:**
- john user was in sudoers file (you can see in /etc/sudoers.d/ that john has sudo rights)
- Attacker knew john's password (brute-forced or social engineering)
- Attacker ran `sudo /usr/bin/python3` (no password prompt needed if sudoers allows it)

**3. Malware identification:**
- `/usr/bin/python3 /tmp/miner.py` is the malware
- Running as root (PID 5434, owner root)
- Using 87% CPU, 400MB RAM (resource hogging = cryptocurrency miner)
- Listening on port 4444 and connecting to 192.168.1.50:3333 (mining pool)

**4. Critical evidence:**
```
"Accepted password for john from 203.0.113.10" = attacker login
"sudo: john : USER=root ; COMMAND=" = privilege escalation
"1 incorrect password attempt" = attacker guessing bash password initially
```

**5. Investigation needed:**
- How did attacker know john's password? (Password spray? Phishing? Leaked credentials?)
- Were john's SSH keys modified?
- What did attacker access before launching miner?
- How long was attacker in the system?
- Did attacker access customer data?
- Are there other compromised accounts?

---

## 🎯 Interview Questions You Might Get

### Easy (L1 Knowledge)

**Q:** "What's the main difference between Windows and Linux from a security perspective?"
**A:** "Linux is built on principle of least privilege—each user/process has minimal permissions by default. Permissions are explicit and visible. Windows defaults to more permissive. Also, Linux servers are usually command-line (CLI) while Windows is GUI-heavy, which changes investigation approach."

**Q:** "Explain what sudo does in one sentence."
**A:** "sudo allows authorized users to run commands with elevated privileges (usually as root) without sharing the root password."

**Q:** "What does chmod 755 mean?"
**A:** "Owner can read/write/execute, group can read/execute, others can read/execute. Typically used for executable scripts."

### Medium (L2 Understanding)

**Q:** "You find a Linux server where /etc/sudoers file permissions are `-rw-rw-rw-`. What does this mean for security and what should you investigate?"
**A:** "This is critical vulnerability—anyone can modify the sudoers file to give themselves root access. This indicates:
1. Likely privilege escalation already happened (attacker modified permissions)
2. Need to check sudo logs for unusual usage
3. Need to restore sudoers from backup
4. Need to identify how attacker escalated to modify sudoers in first place"

**Q:** "How would you detect if an attacker escalated privileges using sudo?"
**A:** "Check `/var/log/auth.log` for sudo usage patterns:
- Look for failed sudo attempts (attacker guessing)
- Look for unusual commands via sudo (especially shell access)
- Look for unusual times (3 AM logins)
- Look for unusual sudo source (service account using sudo shouldn't happen)
Example: `grep sudo /var/log/auth.log | grep -v "sudo: root"`"

**Q:** "Walk me through investigating a suspicious process you found running as root."
**A:** "1. Identify the process: `ps aux | grep PROCESS`
2. Check what it's doing: `lsof -p PID` (open files)
3. Check network connections: `ss -tlnp | grep PID`
4. Find when it started: check system logs during that timeframe
5. Check who started it: look at parent process and auth.log
6. Analyze the binary: `strings /path/to/binary` (find hardcoded IPs, URLs)
7. Isolate and kill if malicious"

### Hard (L3/Senior)

**Q:** "You're investigating a compromised Linux server. User 'john' shows sudo usage at 3 AM, but john claims he didn't log in that night. Explain how this is possible and what your next steps would be."
**A:** "Possible explanations:
1. SSH key compromise—attacker has john's SSH key, so they log in without password prompt
2. Brute-forced password—1000+ failed attempts then one success
3. Compromised parent process—something john runs had vulnerability, attacker pivoted
4. john's account was already compromised days ago, attacker just using it now

Next steps:
- Check auth.log for SSH login method (password vs publickey)
- If publickey, examine ~/.ssh/authorized_keys for unauthorized keys
- Check if SSH key has different modification time than expected
- Review full sudo log for COMMAND field—what exactly did attacker run?
- Timeline: when was john's password/key last verified as secure?
- Cross-reference john's legitimate activities—can we exclude those from investigation?
- Check if john's account has legitimate automation that could explain 3 AM activity"

**Q:** "Describe a scenario where file permissions are MORE important for security than on Windows and explain why."
**A:** "Linux scenario: Database server runs as 'mysql' user. Database files have permissions `-rw-r-----` (owned by mysql:mysql). If attacker compromises web application (running as www-data user), they CANNOT read database files even though both are on same server.

Windows comparison: Windows ACLs are more complex but less commonly understood. Many administrators default to 'Everyone: Full Control' which negates the protection.

Why this matters: Linux permissions are simple, visible, and hard to bypass. This principle of least privilege is why Linux is preferred for critical infrastructure. The attacker has to:
1. Compromise www-data account
2. Escalate to mysql user
3. Find credentials
4. Connect to database

vs Windows where misconfigured ACLs might allow direct access."

---

## 🔗 How This Connects To Everything Else

- **Incident Response Framework (NIST):** Linux investigation happens during Detection & Analysis phase. You're gathering evidence from logs and process analysis.
- **MITRE ATT&CK:** Linux-specific techniques like "Privilege Escalation via sudo" (T1548.003), "Process Injection" (T1055), "Create or Modify System Process" (T1543)
- **Windows Event IDs:** Linux has no Event IDs but `/var/log/auth.log` serves similar purpose for authentication events
- **Active Directory:** Often integrated with Linux via LDAP/Kerberos. Linux users can be AD users. Compromise can affect both systems.
- **Networking Concepts:** Understanding ports, connections, network isolation is easier to trace in Linux (every process visible with `ss`)
- **SIEM:** SIEM systems collect Linux logs, parse auth.log, detect patterns (failed logins, sudo abuse)
- **Threat Hunting:** Proactive search for suspicious processes, permission changes, or unusual sudo usage in Linux systems
- **Forensics (DFIR):** Linux memory and disk forensics rely on understanding process structure, file ownership, and log locations

---

## 💾 TL;DR For Busy People

| Concept | What It Is | Why It Matters in SOC |
|---------|-----------|----------------------|
| **Users & Permissions** | Every file has owner and permissions (rwx) | Attacker's access limited by permissions; permissions changes = evidence |
| **Sudo** | Temporary root access for authorized users | Privilege escalation vector; sudo logs prove compromise |
| **File Permissions (chmod)** | Read/Write/Execute for owner/group/others | Malicious files change permissions; suspicious patterns = attack |
| **/var/log/auth.log** | Authentication and sudo usage log | Where you find login attempts, sudo abuse, escalation evidence |
| **Processes** | Running programs with owner, PID, parent | Malware appears as unexpected processes; process ownership indicates privilege level |
| **SSH Keys** | Asymmetric cryptography for authentication | Compromised keys = persistent access without password attempts in logs |
| **Servers** | Linux systems running infrastructure | Most web servers, databases, routers run Linux; investigation requires CLI access |

---

## 📌 Production Reality: Your First Week

**Monday:** Your manager shows you a Linux server that's been hacked.
```bash
ssh john@compromised-server.internal
$ ps aux
root  5432  87.3  45.1  524288 400000  ?  R  03:47  1:23  /tmp/miner
$ grep "sudo" /var/log/auth.log | tail -20
# Discover: attacker used john's account to run sudo commands
```

**Tuesday:** You investigate HOW they got john's password.
```bash
$ grep "john" /var/log/auth.log | grep "Failed"
# 2,347 failed attempts from 203.0.113.50
$ grep "john" /var/log/auth.log | grep "Accepted"
# Finally: "Accepted password for john from 203.0.113.50"
# = Brute force attack succeeded
```

**Wednesday:** You check for privilege escalation.
```bash
$ sudo -l  # What can john do with sudo?
User john may run the following commands on server:
    (ALL) ALL
# OH NO - john can run ANY command as root
$ cat /etc/sudoers.d/john
# When was this added? Who added it?
```

**Thursday:** You contain and recover.
```bash
# 1. Kill malware process
$ sudo kill -9 5432

# 2. Remove john from sudoers
$ sudo visudo -r /etc/sudoers.d/john

# 3. Reset john's password
$ sudo passwd john

# 4. Remove SSH keys (if compromised)
$ rm ~/.ssh/authorized_keys
```

**Friday:** You document and present findings.
```
Root Cause: john's weak password brute-forced via SSH
Attack Timeline: 3:47 AM attacker logged in, immediately escalated to root via sudo
Evidence: 2,347 failed SSH attempts, then successful login, then sudo to root
Impact: Attacker ran cryptocurrency miner for ~3 hours
Remediation: Password reset, SSH key rotation, remove excessive sudo permissions
Prevention: Enable SSH key auth only, 2FA, rate limiting on SSH
```

---

## 📌 Real Job Perspective: What Matters Most

When you're hired as a Junior SOC Analyst:

1. **Your manager will say:** "We have a Linux server that might be compromised, investigate"
2. **You NEED to know:**
   - How to SSH into the server
   - How to check what processes are running
   - How to read `/var/log/auth.log` for attack evidence
   - How to identify privilege escalation via sudo
   - How to recognize malware (unexpected processes, high CPU, outbound connections)
   - How to document findings

3. **You DON'T need to:**
   - Write Linux kernel modules
   - Build a Linux system from scratch
   - Understand every Linux distro
   - Become a Linux system administrator

4. **What will make you look competent:**
   - Finding the attack timeline in logs
   - Identifying how attacker escalated privileges
   - Recognizing suspicious process patterns
   - Knowing what evidence to preserve
   - Explaining your findings clearly

---

## ❌ Common Interview Mistakes

**Mistake 1: Confusing Linux with GNU**
Wrong: "Linux is the entire operating system"
Correct: "Linux is the kernel; GNU tools provide the userspace utilities"
In interviews: Just call it "Linux" and you'll be fine. Pedantic distinction matters for system architects, not SOC analysts.

**Mistake 2: Not Understanding SSH Login vs Shell Access**
Wrong: "If SSH login succeeded, attacker has shell"
Correct: SSH login confirms authentication; attacker still needs shell access
In interviews: Say "SSH allows remote access, but we need to verify what commands they executed"

**Mistake 3: Overthinking Permissions**
Wrong: Trying to calculate octal permissions in head during interview
Correct: Know the common ones (755, 644, 700) and understand the principle
In interviews: "rwxr-xr-x means owner can do everything, others can only read/execute" is enough

---

## 📚 Further Reading & Resources

**Essential Reading:**
- NIST SP 800-53 (Security Controls for Linux systems)
- Ubuntu Security Guide (free online)
- "Linux Basics for Hackers" by OccupyTheWeb (pragmatic for SOC)

**Practice Resources:**
- TryHackMe (free and paid Linux challenges)
- HackTheBox (similar to TryHackMe)
- OverTheWire Wargames (free, excellent for learning)
- Create your own Ubuntu VM locally and practice commands

**Key Man Pages to Read:**
- `man sudo`
- `man chmod`
- `man ls`
- `man ssh`
- `man tail` (for log analysis)

**Real-World Log Analysis:**
- Take security incident reports and identify Linux evidence
- Follow DFIR case studies on blogs (2 SecShop, SANS Pen Test, etc.)
- Read post-mortems on HackerNews comments

---

## ⚠️ Closing Words

Linux isn't "easy to edit" as an operating system. It's **transparent**—you can see what's running, who owns it, and what changed. This transparency is what makes it powerful for security investigations.

In your SOC career, Linux will be everywhere: web servers, databases, load balancers, routers, security appliances. Understanding how Linux security works (users, permissions, logs) is not optional—it's fundamental.

The good news: Linux security is actually simpler than Windows once you understand the core concepts. Everything is a file. Files have permissions. Users have limited access. Logs tell you everything.

Master these, and you'll be significantly ahead of most junior analysts.

---

*Last Updated: 2024*
*Difficulty: L1-L2*
*Interview Relevance: ⭐⭐⭐⭐⭐*
*Job Applicability: Required Knowledge for All SOC Roles*
*Production Applicability: Daily Use in Incident Response*
