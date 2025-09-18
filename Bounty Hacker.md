# Bounty Hacker - Write Up
## **Introduction**

📊 Difficulty : Easy

Bounty Hacker is a beginner-friendly, hands-on [TryHackMe](http://tryhackme.com) room that simulates a classic box-hunting scenario: deploy a vulnerable machine, enumerate open services, exploit a weak service (including a bruteforce/credential-recovery step), and capture both `user.txt` and `root.txt`.

Link to the room : https://tryhackme.com/room/cowboyhacker

--- 
### **Context**

> *You were boasting on and on about your elite hacker skills in the bar and a few Bounty Hunters decided they'd take you up on claims! Prove your status is more than just a few glasses at the bar. I sense bell peppers & beef in your future!*

As usual, let's start with an `nmap` scan :

![[20250918-Bounty Hunters.png]]

As expected with this type of rooms, we find ``port 80`` open.

Accessing the web server running on the target machine, we are greeted with the following page : 

![[20250918-Bounty Hunters-1.png]]

I checked out the page and looked through its content, but didn’t come across anything useful. Running `gobuster` for further enumeration didn’t turn up anything either.

Ok so it seems `port 80` is a dead end. Let's look at the other open ports, beginning with FTP.

Trying to connect to FTP, I found that the server was anonymous only. Therefore, I logged in as `anonymous`. Once logged in to the FTP server, I discovered two files `locks.txt` and `task.txt`.

I went ahead and downloaded the two files to my local machine.

![[20250918-Bounty Hunters-2.png]]

After examining the contents of the two text files, I came to two conclusions:

![[20250918-Bounty Hunters-3.png]]

![[20250918-Bounty Hunters-4.png]]

1. `lin` is the username I’ll use to log in via SSH.
2. `locks.txt` contains a list of potential passwords, one of which is the password for `lin` that I’ll use to access SSH.

I used `hydra` to brute force the password using `lin` as the username and the passwords contained in the `locks.txt` and found the correct password : 
![[20250918-Bounty Hunters-5.png]]

Using the newly found password, I was able to log in via SSH as `lin` and immediately find the user flag contained in `user.txt`.

![[20250918-Bounty Hunters-6.png]]
Now, onto privilege escalation. 

By runnning `sudo -l` to find commands that I could run as sudo, I found the following :  

![[20250918-Bounty Hunters-7.png]]
Doing a little research on GTFObins, I found that `tar` could be exploited if ran with root privileges with the following command, which enables us to find the root flag and complete the room : 

![[20250918-Bounty Hunters-8.png]]

---
# Conclusion 

Bounty Hacker was a very basic and straightforward room with nothing too tricky to solve it but it was still great practice for recon, credential hunting, and privilege escalation. Even though it was easy, it reinforced the core methodology you’ll use on more complex boxes.