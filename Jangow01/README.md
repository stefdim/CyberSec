 Jangow01 - VulnHub Walkthrough

## Description
Jangow01 is an easy-level VulnHub machine focused on enumeration and basic exploitation techniques.  
The goal is to gain user and root access.

---

## Initial Enumeration

Usually jangow01 vulnhub provides the ip when we power up the machine, although we could find it by using 
<br><img width="555" height="112" alt="jango" src="https://github.com/user-attachments/assets/3a178875-8379-466b-bef8-0c0ba3f1f8f3" />

```bash 
netdiscover -i eth0
```
# STEP 1.
First, I performed a basic Nmap scan to identify open ports and services:
```bash
nmap -sV 10.0.2.15
```
<br><img width="492" height="98" alt="Screenshot 2026-03-23 121055" src="https://github.com/user-attachments/assets/1fc624b5-fd12-4742-b9d7-f45cefab2931" />
<br>Open Ports : 
<li>21 → FTP (vsftpd 3.0.3)
<li>80 → HTTP (Apache 2.4.18)

Flag used -> -sV to detect services. <br>We could proceed with more detailed scan but at this point is not necessary since we already found that a web service is available (80 → HTTP (Apache 2.4.18)) so we may start with web enumeration.

# STEP 2.
<B>Web Exploration

Visit the website:
<br><img width="757" height="435" alt="browser_jangow01" src="https://github.com/user-attachments/assets/355610a0-d65f-416f-bcc5-b1db7204502f" />

Navigate to discover potential vulnerabilities.<br>
<b>Discoveries<br>
<br><img width="1056" height="70" alt="parameter_url" src="https://github.com/user-attachments/assets/c2e760af-9072-4787-9f79-b187825ec1e8" />
<br>when visited the tab Buscar i saw that the url provide a potential entry for parameters/commands so i tested basic input.<br>
# STEP 3.
<b>Command Injection<br><li>Testing with: echo test
<br><img width="1288" height="437" alt="echo test" src="https://github.com/user-attachments/assets/1ba797a1-de6e-4b79-b747-fa10229e5a17" />
The output was reflected back, confirming command execution.<br>
<li> After that i proceed with some basic listing commands</li><br><img width="1173" height="537" alt="ls -a" src="https://github.com/user-attachments/assets/ec163397-da9e-4734-87f7-9b4ded297e6d" /><img width="1182" height="543" alt="Screenshot 2026-03-23 153111" src="https://github.com/user-attachments/assets/64fec3d2-5e5f-4319-a65d-659f0017da0e" />

<U><strong>Commands used:
<li>echo test</li>
<li>ls -a</li>
<li>whoami</li> 
<li>pwd</li>
<li>uname -a</li> --> <img width="1154" height="331" alt="Screenshot 2026-03-23 131652" src="https://github.com/user-attachments/assets/a5720466-ebab-4d1c-9f13-54bf9d335a03" />


At this point, I had remote command execution.

# STEP 4
<B>Credential discovery

With command execution available, I started exploring the system:
<br>
<img width="968" height="249" alt="list files www" src="https://github.com/user-attachments/assets/bc8389e0-5f28-43e3-88a0-3c428888f7ae" /> <img width="1311" height="595" alt="ls wordpress" src="https://github.com/user-attachments/assets/3ef68e24-cecf-41d9-b293-9dbcb82bcb06" /><img width="1135" height="475" alt="cat config" src="https://github.com/user-attachments/assets/b25fe678-3c31-467c-bbce-46353332fdf9" />


I discovered a hidden file:

.backup

Reading it revealed credentials:<br><img width="1264" height="210" alt="backup info" src="https://github.com/user-attachments/assets/476104cd-8d8d-419f-aa8b-e965b18513e6" />
<img width="1063" height="416" alt="cat backup" src="https://github.com/user-attachments/assets/42939099-069c-4649-a5f6-a772f33c6276" />

<U><strong>Commands used:
<li>ls -la /var/www/html</li>

# STEP 5
<b>User Access

Using the credentials:




su jangow01

Verification:

whoami

Output:

jangow01

Now I had a more stable shell.

Privilege Escalation

Checking the system:

uname -a

Output:

Linux 4.4.0-31-generic

This is an old kernel, so I searched for local privilege escalation exploits.

First Attempt

I tried exploit 40871, but it kept retrying and did not succeed.

Since it was unstable, I moved on.

Working Exploit

I found another exploit:

Exploit-DB 45010

Uploaded it to the target and compiled it:

gcc 45010.c -o exploit
chmod +x exploit
./exploit
Root Access

After execution:

#

Verification:

whoami

Output:

root
Proof
cd /root
cat proof.txt
Summary

This machine was compromised through:

Command Injection → Initial access
Credential discovery → User access
Kernel exploit → Root access
Key Takeaways
Always test input fields for command injection
Hidden files can expose credentials
Enumeration is critical
Not all exploits work — adapt and try alternatives
