# Bounty Hacker - Write Up

## **Introduction**

📊 Difficulty : Easy

Bounty Hacker is a beginner-friendly, hands-on [TryHackMe](http://tryhackme.com) room that simulates a classic box-hunting scenario: deploy a vulnerable machine, enumerate open services, exploit a weak service (including a bruteforce/credential-recovery step), and capture both `user.txt` and `root.txt`.

Link to the room : https://tryhackme.com/room/cowboyhacker

--- 
### **Context**

> *You were boasting on and on about your elite hacker skills in the bar and a few Bounty Hunters decided they'd take you up on claims! Prove your status is more than just a few glasses at the bar. I sense bell peppers & beef in your future!*

As usual, let's start with an `nmap` scan :

<img width="1914" height="286" alt="1" src="https://github.com/user-attachments/assets/3b3253c8-3824-41ea-8b5a-9ee48e0cf55b" />

As expected with this type of rooms, we find ``port 80`` open.

Accessing the web server running on the target machine, we are greeted with the following page : 

<img width="1919" height="1031" alt="2" src="https://github.com/user-attachments/assets/acf75a91-8419-4d23-ae5a-9d411dbf01ab" />

I checked out the page and looked through its content, but didn’t come across anything useful. Running `gobuster` for further enumeration didn’t turn up anything either.

Ok so it seems `port 80` is a dead end. Let's look at the other open ports, beginning with FTP.

Trying to connect to FTP, I found that the server was anonymous only. Therefore, I logged in as `anonymous`. Once logged in to the FTP server, I discovered two files `locks.txt` and `task.txt`.

I went ahead and downloaded the two files to my local machine.

<img width="1917" height="519" alt="3" src="https://github.com/user-attachments/assets/3e2610bf-698b-4e2a-877a-c34559ff0cf4" />

After examining the contents of the two text files, I came to two conclusions:

<img width="856" height="108" alt="4" src="https://github.com/user-attachments/assets/8e488a33-f85e-4250-9290-b0048f1364a6" />
<img width="931" height="466" alt="5" src="https://github.com/user-attachments/assets/bc32b0bd-9937-4106-a82d-5a54f9418f96" />

1. `lin` is the username I’ll use to log in via SSH.
2. `locks.txt` contains a list of potential passwords, one of which is the password for `lin` that I’ll use to access SSH.

I used `hydra` to brute force the password using `lin` as the username and the passwords contained in the `locks.txt` and found the correct password : 

<img width="1741" height="244" alt="6" src="https://github.com/user-attachments/assets/8b767370-0d1d-4507-a261-f871270b48db" />

Using the newly found password, I was able to log in via SSH as `lin` and immediately find the user flag contained in `user.txt`.

<img width="1308" height="527" alt="7" src="https://github.com/user-attachments/assets/1de14798-4bed-4f5b-b483-2aa0dc142a81" />

Now, onto privilege escalation. 

By runnning `sudo -l` to find commands that I could run as sudo, I found the following :  

<img width="1047" height="145" alt="8" src="https://github.com/user-attachments/assets/ea0bae2a-d7da-41da-ab36-d791263f41a3" />

Doing a little research on GTFObins, I found that `tar` could be exploited if ran with root privileges with the following command, which enables us to find the root flag and complete the room : 

<img width="942" height="169" alt="9" src="https://github.com/user-attachments/assets/9315a913-270f-454d-99cb-282069623767" />

---
# Conclusion 

Bounty Hacker was a very basic and straightforward room with nothing too tricky to solve it but it was still great practice for recon, credential hunting, and privilege escalation. Even though it was easy, it reinforced the core methodology to use on more complex boxes.
