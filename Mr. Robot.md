# Mr. Robot - Write Up

## **Introduction**

📊 Difficulty : Medium

Mr. Robot is an iconic CTF for web penetration, and challenges users on all aspects and all steps of web application penetration.

Link to the room : https://tryhackme.com/room/mrrobot

--- 
## Context

> *Can you root this Mr. Robot styled machine? This is a virtual machine meant for beginners/intermediate users. There are 3 hidden keys located on the machine, can you find them?*

## Reconnaissance

As always, we start by enumerating open ports. 

<img width="805" height="217" alt="20250829-Mr  Robot" src="https://github.com/user-attachments/assets/c55d7a42-b2a4-45e5-a9a9-99e7430261e1" />


Let's access the website hosted by the system.

<img width="1353" height="615" alt="20250829-Mr  Robot-1" src="https://github.com/user-attachments/assets/1445d629-253d-462e-af54-4a063867ac89" />

The website has a bunch of redirects, but nothing of interest for us. Let's enumerate for hidden directories using gobuster. To achieve this, I used the common.txt wordlist from /dirb, downloaded by default on Kali Linux and on the attack box from TryHackMe.

```
gobuster dir -u MACHINE_IP -w WORDLIST
```

<img width="680" height="729" alt="20250829-Mr  Robot-6" src="https://github.com/user-attachments/assets/9e447119-60da-4548-815e-c01fb26345a8" />

As we can see, there are a number of directories we can explore, some more interesting than others. For example, the /readme directory returns the following message : 

<img width="843" height="135" alt="20250829-Mr  Robot-5" src="https://github.com/user-attachments/assets/de9cc1c4-7c77-4a79-97ae-a409e4f2c5a0" />

Other than getting a little chuckle out of me, there isn't a lot we can do with this directory.

One particularly interesting file is **/robots.txt**, which websites use to specify directories they want to exclude from search engine indexing and prevent from appearing in search results. 

Accessing the /robots.txt directory, we find our first flag : 

<img width="655" height="169" alt="20250829-Mr  Robot-2" src="https://github.com/user-attachments/assets/7b7dd0d7-5d3c-406e-9410-bf3d35b940b7" />

<img width="797" height="164" alt="20250829-Mr  Robot-3" src="https://github.com/user-attachments/assets/9063a2df-9231-4f9d-b143-e91795507222" />

The other directory present in the robots.txt list looks like a wordlist. Let's keep that on the side for later.

<img width="834" height="495" alt="20250829-Mr  Robot-4" src="https://github.com/user-attachments/assets/41944dbb-fcfd-4cc9-a551-6cf641149b9a" />


Another directory found by gobuster was ``/wp-login``. Accessing that directory, we find a login form. 

<img width="1147" height="635" alt="20250829-Mr  Robot-7" src="https://github.com/user-attachments/assets/d05e4ff8-4f30-4635-99ce-e6a43acff994" />

Trying random values for username and password, we get the following error message : 

<img width="438" height="522" alt="20250829-Mr  Robot-8" src="https://github.com/user-attachments/assets/53ad188a-1686-47ca-80fa-d3fd6d573053" />


## Brute force

As we can see, the web server notifies us that the username we entered was not associated to an account. This kind of message can help us find existing usernames. For that, let's start by intercepting the POST request sent when trying to log in.

<img width="997" height="443" alt="20250829-Mr  Robot-9" src="https://github.com/user-attachments/assets/821ad29c-d7b8-456f-90aa-9b1d103d1fb4" />

The username is stored in the ``log`` variable, while the password is stored in the ``pwd`` variable.

Let's use brute force to find existing usernames. We'll be using ``Hydra`` and the wordlist obtained earlier.

But first, if we look closely at the wordlist, we'll see that some words appear multiple times, which may result in longer processing time. So let's filter the wordlist to only contain unique values : 

```
hydra -L fscocity.dic -p test MACHINE_IP http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:Invalid username"
```

We find one username :

<img width="1383" height="204" alt="20250829-Mr  Robot-10" src="https://github.com/user-attachments/assets/55bf4be4-59dd-43f1-9388-f17adfa315a8" />

Now that we have a username, it is time to find the password. Let's try a random password with the obtained username : 

<img width="382" height="555" alt="20250829-Mr  Robot-11" src="https://github.com/user-attachments/assets/b2a671ee-8db2-49a4-adc3-d58fd5f82b14" />

Just like we did with the username, we use Hydra to find the password from the provided wordlist : 

```
hydra -l Elliot -P fscocity.dic MACHINE_IP http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^:The password you entered"
```

<img width="1385" height="225" alt="20250829-Mr  Robot-12" src="https://github.com/user-attachments/assets/85cf7ced-f6b1-4ac4-a391-d46b53ad982b" />


We can now log into Elliot's account. Once we are logged in, we are greeted with a dashboard page.

<img width="1361" height="627" alt="20250829-Mr  Robot-13" src="https://github.com/user-attachments/assets/1fcd180c-8586-43f8-9959-03894ba2dcef" />

## Web shell

Now, we can look around, click on links, use the inspect tool to find potential intrusion vectors, etc. One of the page we can access from this dashboard is Edit Themes page. There, we can insert PHP code to customize the themes of the web page. We can use this to insert our own script and obtain a web shell.

<img width="1150" height="605" alt="20250829-Mr  Robot-14" src="https://github.com/user-attachments/assets/085d17cb-0acf-4c88-88d8-c22afae0d67c" />

We will be using the ``php-reverse-shell.php`` script (link : https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), replacing the IP with ours.

<img width="961" height="606" alt="20250829-Mr  Robot-15" src="https://github.com/user-attachments/assets/5d248281-9ac0-4e94-afa8-6aebeb8d49ce" />


Once saved, we now have to find a way to execute this script. After a bit of digging, and from the ``/wp-content`` directory found with gobuster, we find the following path, which executes the `archive.php` script we injected.

<img width="1165" height="96" alt="20250829-Mr  Robot-16" src="https://github.com/user-attachments/assets/3e11da89-02e7-4671-b16c-17df6896e3ac" />


All we need now is a listener to catch the connection on our machine. We can use netcat for this : 

```
nc -nlvp 1234
```

We have successfully obtained a shell and can now explore the system to find the second flag!

<img width="275" height="128" alt="20250829-Mr  Robot-17" src="https://github.com/user-attachments/assets/dfecd6ef-f532-462c-958d-9a52eba3a2c0" />

We can safely assume that the last flag is located in the `/root` directory, which can be accessed with elevated privileges.  

## Privilege escalation

Let's search for privilege escalation vectors. Let's start by looking at the usual PE vectors, starting with enumerating binaries with the SUID bit set : 

<img width="685" height="401" alt="20250829-Mr  Robot-18" src="https://github.com/user-attachments/assets/8e510216-0430-4b7a-af60-cbb35e3481f2" />

Searching on [GTFObins](https://gtfobins.github.io/gtfobins), we find that nmap can be used to escalate privileges with the following command : 

```
./nmap -oG=$LFILE DATA
```

After running the command, specifying the file we want to open with root privileges, we obtain a new shell with root user.

<img width="595" height="133" alt="20250829-Mr  Robot-19" src="https://github.com/user-attachments/assets/c6450084-161e-40d5-8b51-007043e316c9" />

Only problem is : we can't change directory. However, now that we have root privileges, we can use the command from [GTFObins](https://gtfobins.github.io/gtfobins) to escalate privileges with Sudo. Once done, we obtain one last shell which allows us to explore the system with root privileges and find the last flag. 

<img width="335" height="161" alt="20250829-Mr  Robot-20" src="https://github.com/user-attachments/assets/5b768792-1ebb-4e18-ad8f-c3705d5b4ca6" />

---
# Conclusion 

Mr. Robot is a classic when it comes to CTF. It successfully challenges users by making them go through the classic steps of web penetration, from enumeration, injection, getting a shell and privilege escalation and serves as an excellent practice for web penetration. I was a bit nervous when I started this room, but with a solid approach and some help from internet searches, I managed to complete it; though it wasn't without its challenges.

