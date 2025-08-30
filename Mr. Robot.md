{Title}
## **Introduction**

📊 Difficulty : Medium

Mr. Robot is an iconic CTF for web penetration, and challenges users on all aspects and all steps of web application penetration.

Link to the room : https://tryhackme.com/room/mrrobot

--- 
## Context

> *Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them?*

## Reconnaissance

As always, we start by enumerating open ports. 

![[20250829-Mr. Robot.png]]

Let's access the website hosted by the system.

![[20250829-Mr. Robot-1.png]]

The website has a bunch of redirects, but nothing of interest for us. Let's enumerate for hidden directories using gobuster. To achieve this, I used the common.txt wordlist from /dirb, downloaded by default on Kali Linux and on the attack box from TryHackMe.

```
gobuster dir -u MACHINE_IP -w WORDLIST
```

![[20250829-Mr. Robot-6.png]]

As we can see, there are a number of directories we can explore, some more interesting than others. For example, the /readme directory returns the following message : 

![[20250829-Mr. Robot-5.png]]

Other than getting a little chuckle out of me, there isn't a lot we can do with this directory.

One particularly interesting file is **/robots.txt**, which websites use to specify directories they want to exclude from search engine indexing and prevent from appearing in search results. 

Accessing the /robots.txt directory, we find our first flag : 

![[20250829-Mr. Robot-2.png]]

![[20250829-Mr. Robot-3.png]]

The other directory present in the robots.txt list looks like a wordlist. Let's keep that on the side for later.

![[20250829-Mr. Robot-4.png]]

Another directory found by gobuster was ``/wp-login``. Accessing that directory, we find a login form. 

![[20250829-Mr. Robot-7.png]]

Trying random values for username and password, we get the following error message : 

![[20250829-Mr. Robot-8.png|0x0]]

As we can see, the web server notifies us that the username we entered was not associated to an account. This kind of message can help us find existing usernames. For that, let's start by intercepting the POST request sent when trying to log in.

## Brute force

![[20250829-Mr. Robot-9.png]]

The username is stored in the ``log`` variable, while the password is stored in the ``pwd`` variable.

Let's use brute force to find existing usernames. We'll be using ``Hydra`` and the wordlist obtained earlier.

But first, if we look closely at the wordlist, we'll see that some words appear multiple times, which may result in longer processing time. So let's filter the wordlist to only contain unique values : 

```
hydra -L fscocity.dic -p test MACHINE_IP http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username"
```

We find one username :

![[20250829-Mr. Robot-10.png]]

Now that we have a username, it is time to find the password. Let's try a random password with the obtained username : 

![[20250829-Mr. Robot-11.png]]

Just like we did with the username, we use Hydra to find the password from the provided wordlist : 

```
hydra -l Elliot -P fscocity.dic MACHINE_IP http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered"
```

![[20250829-Mr. Robot-12.png]]

We can now log into Elliot's account. Once we are logged in, we are greeted with a dashboard page.

![[20250829-Mr. Robot-13.png]]

## Web shell

Now, we can look around, click on links, use the inspect tool to find potential intrusion vectors, etc. One of the page we can access from this dashboard is Edit Themes page. There, we can insert PHP code to customize the themes of the web page. We can use this to insert our own script and obtain a web shell.

![[20250829-Mr. Robot-14.png]]

We will be using the ``php-reverse-shell.php`` script (link : https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), replacing the IP with ours.

![[20250829-Mr. Robot-15.png]]

Once saved, we now have to find a way to execute this script. After a bit of digging, and from the ``/wp-content`` directory found with gobuster, we find the following path, which executes the `archive.php` script we injected.

![[20250829-Mr. Robot-16.png]]

All we need now is a listener to catch the connection on our machine. We can use netcat for this : 

```
nc -nlvp 1234
```

We have successfully obtained a shell and can now explore the system to find the second flag!

![[20250829-Mr. Robot-17.png]]

We can safely assume that the last flag is located in the `/root` directory, which can be accessed with elevated privileges.  

## Privilege escalation

Let's search for privilege escalation vectors. Let's start by looking at the usual PE vectors, starting with enumerating binaries with the SUID bit set : 

![[20250829-Mr. Robot-18.png]]

Searching on [GTFObins](https://gtfobins.github.io/gtfobins), we find that nmap can be used to escalate privileges with the following command : 

```
./nmap -oG=$LFILE DATA
```

After running the command, specifying the file we want to open with root privileges, we obtain a new shell with root user.

![[20250829-Mr. Robot-19.png]]

Only problem is : we can't change directory. However, now that we have root privileges, we can use the command from [GTFObins](https://gtfobins.github.io/gtfobins) to escalate privileges with Sudo. Once done, we obtain one last shell which allows us to explore the system with root privileges and find the last flag. 

![[20250829-Mr. Robot-20.png]]

---
# Conclusion 

Mr. Robot is a classic when it comes to CTF. It successfully challenges users by making them go through the classic steps of web penetration, from enumeration, injection, getting a shell and privilege escalation and serves as an excellent practice for web penetration. I was a bit nervous when I started this room, but with a solid approach and some help from internet searches, I managed to complete it; though it wasn't without its challenges.

