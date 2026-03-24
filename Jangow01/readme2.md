<h1>Jangow01 - VulnHub Walkthrough</h1>

<h2>Description</h2>
<p>
Jangow01 is an easy-level VulnHub machine focused on enumeration, web exploitation,
credential discovery, and local privilege escalation.
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
I started with an Nmap scan to identify exposed ports and services on the target.
</p>

<pre><code>nmap -sV 10.0.2.15</code></pre>

<p>
The scan revealed the following open ports:
</p>

<ul>
  <li><strong>21/tcp</strong> - FTP</li>
  <li><strong>80/tcp</strong> - HTTP</li>
</ul>

<p>
Since a web service was available, I decided to focus on HTTP first.
</p>

<br><img width="484" height="90" alt="Screenshot 2026-03-23 121055" src="https://github.com/user-attachments/assets/a7a2445e-2682-4af7-afa3-952aa7ebcabb" />


<hr>

<h2>2. Web Enumeration</h2>
<p>
After visiting the target website, I found a parameter that looked interesting:
</p>

<pre><code>/site/busque.php?buscar=</code></pre>

<p>
This parameter appeared to accept user-controlled input directly through the URL,
so it became the main attack surface.
</p>

<p><strong>φωτο απο browser με το parameter</strong></p>

<hr>

<h2>3. Command Injection Discovery</h2>
<p>
To understand how the parameter behaved, I tested it with a simple command:
</p>

<pre><code>echo test</code></pre>

<p>
The output was reflected back in the page, which suggested command execution.
</p>

<p><strong>φωτο απο echo test</strong></p>

<p>
To verify the execution context, I used:
</p>

<pre><code>whoami</code></pre>

<p>
The result was:
</p>

<pre><code>www-data</code></pre>

<p>
This confirmed that I had remote command execution as the web server user.
</p>

<p><strong>φωτο απο whoami</strong></p>

<hr>

<h2>4. Credential Discovery</h2>
<p>
With command execution available, I began enumerating the file system.
One of the first directories I checked was the web root:
</p>

<pre><code>ls -la /var/www/html</code></pre>

<p><strong>φωτο απο ls -la /var/www/html</strong></p>

<p>
During enumeration, I discovered a hidden file:
</p>

<pre><code>.backup</code></pre>

<p>
Inspecting this file revealed valid credentials:
</p>

<pre><code>jangow01 : abygurl69</code></pre>

<p>
This was an important finding because it provided a path to move from the web user
to a real local user account.
</p>

<p><strong>φωτο απο backup file / credentials</strong></p>

<hr>

<h2>5. User Access</h2>
<p>
Using the discovered credentials, I switched to the local user:
</p>

<pre><code>su jangow01</code></pre>

<p><strong>φωτο απο su jangow01</strong></p>

<p>
To confirm the user context, I ran:
</p>

<pre><code>whoami</code></pre>

<p>
The result was:
</p>

<pre><code>jangow01</code></pre>

<p>
At this point, I had a more stable shell and a better position for local enumeration.
</p>

<p><strong>φωτο απο whoami jangow01</strong></p>

<hr>

<h2>6. Local Enumeration</h2>
<p>
Next, I checked the kernel version to identify possible privilege escalation paths:
</p>

<pre><code>uname -a</code></pre>

<p><strong>φωτο απο uname -a</strong></p>

<p>
The output showed an old Linux kernel:
</p>

<pre><code>Linux 4.4.0-31-generic</code></pre>

<p>
Because the system was running an outdated kernel, local privilege escalation through
a kernel exploit became a realistic option.
</p>

<hr>

<h2>7. Privilege Escalation</h2>
<p>
I initially tested one exploit, but it proved unstable and kept retrying without success.
Since reliability matters, I moved on to a different exploit that was more suitable for the target kernel.
</p>

<p>
I then compiled and executed the working exploit:
</p>

<pre><code>gcc 45010.c -o exploit
chmod +x exploit
./exploit</code></pre>

<p><strong>φωτο απο compile και run exploit</strong></p>

<p>
After execution, the exploit patched the current process credentials and spawned a root shell.
</p>

<hr>

<h2>8. Root Access</h2>
<p>
To verify that privilege escalation was successful, I ran:
</p>

<pre><code>whoami</code></pre>

<p>
The output was:
</p>

<pre><code>root</code></pre>

<p>
This confirmed full system compromise.
</p>

<p><strong>φωτο απο root shell</strong></p>

<hr>

<h2>9. Proof</h2>
<p>
Finally, I navigated to the root directory and read the proof file:
</p>

<pre><code>cd /root
cat proof.txt</code></pre>

<p><strong>φωτο απο proof.txt</strong></p>

<hr>

<h2>Summary</h2>
<p>
The machine was compromised through the following attack chain:
</p>

<ul>
  <li>Service enumeration with Nmap</li>
  <li>Web parameter analysis</li>
  <li>Command injection</li>
  <li>Credential discovery through a hidden backup file</li>
  <li>User pivot to <code>jangow01</code></li>
  <li>Kernel-based privilege escalation</li>
  <li>Root access and proof retrieval</li>
</ul>

<hr>

<h2>Key Takeaways</h2>
<ul>
  <li>Simple web parameters can hide critical vulnerabilities.</li>
  <li>Hidden files inside web directories are high-value targets.</li>
  <li>Good enumeration makes exploitation much easier.</li>
  <li>If one exploit is unstable, adapt and move to another approach.</li>
</ul>

<hr>

<h2>Disclaimer</h2>
<p>
This walkthrough was created for educational purposes in a controlled lab environment.
</p>
