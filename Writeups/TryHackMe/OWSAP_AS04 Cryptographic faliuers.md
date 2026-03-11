# Overview
Machine : [Application Design Flaws](https://tryhackme.com/)
Platform : [Tryhackme](https://tryhackme.com/)
Difficulty : easy
Category : Web

this machine focus on encryption when use it wrongly 

---
# Scope 
Target IP : 10.49.145.228
In scope services : 

```url
http://10.49.145.228:5004/ 
```
---
# Enumeration
After we deployed the machine `10.49.145.228`
let's investigate `http://10.49.145.228:5004/`

![1](../../images/AS04_Cryptographic_Faliuers_1.png)
#### Misconfiguration
encrypted Document : 
```
Nzd42HZGgUIUlpILZRv0jeIXp1WtCErwR+j/w/lnKbmug31opX0BWy+pwK92rkhjwdf94mgHfLtF26X6B3pe2fhHXzIGnnvVruH7683KwvzZ6+QKybFWaedAEtknYkhe
```

First intercepting **HTTP REQUEST & RESPONSE** for this `http://10.49.145.228:5004/` to view source code if there client-side source code we can use

![3](../images/AS04_Cryptographic_Faliuers_3.png)

there is GET method with interesting file name let's open it

![2](../images/AS04_Cryptographic_Faliuers_2.png)

there is unhidden important variables . Breaking those variables down we can notice that 
```javascript
const ENCRYPTION_MODE = "ECB";
```
ECB is week [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) Cipher mode encryption
Lets decrypt it using "Misconfiguration" i will use this online [tool](https://anycript.com/crypto) . we will just fill those boxes using
![6](../images/AS04_Cryptographic_Faliuers_6.png)
we got the flag!

# Lessons Learned

- Avoid ECB Mode: Never use Electronic Codebook (ECB) mode for sensitive data; always use more secure modes like GCM (Galois/Counter Mode) or CBC (Cipher Block Chaining) with a unique IV.

- Secrets Management: Sensitive cryptographic keys and IVs should never be exposed in client-side source code (JavaScript), as they can be easily intercepted during enumeration.

# Recommendations 

- Use strong modern algorithms such as AES-GCM, ChaCha20-Poly1305, or enforce TLS 1.3 with valid certificates
- Establish and use a secure development lifecycle with AppSec professionals to help evaluate and design security and privacy-related controls
- Establish and use a library of secure design patterns or paved road ready to use components
- Use threat modeling for critical authentication, access control, business logic, and key flows
- Limit resource consumption by user or service
