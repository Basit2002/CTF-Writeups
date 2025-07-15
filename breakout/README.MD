# Breakout CTF – Summary Walkthrough

A full text-based breakdown of the Breakout vulnerable machine from VulnHub.

## Discovery
- Ran `nmap -sV -Pn <ip>` → Found ports: 80 (Apache), 139/445 (SMB), 10000/20000 (Webmin/Usermin)

## Enumeration
- Used `enum4linux` → Discovered valid usernames
- Found Brainfuck-encoded password on port 80 page source → Decoded to get login creds

## Foothold
- Logged into Usermin (port 20000) → Found shell panel → Launched reverse bash shell

## Shell Access
- Used `msfconsole` to catch the shell
- Upgraded to Meterpreter

## Privilege Escalation
- Used `local_exploit_suggester` → DirtyPipe (CVE-2022-0847) exploit used to gain root

---

**Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/empire-breakout-ctf-full-walkthrough-vulnhub-9f9d26f7307d)

