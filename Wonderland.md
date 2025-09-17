# Wonderland - Write Up

## **Introduction**

📊 Difficulty : Medium

Continuing with my training in web penetration, I decided to tackle _Wonderland_, another [TryHackMe](https://tryhackme.com) room. Wonderland is a compact but cleverly designed challenge that walks you through enumeration, exploitation, and post-exploitation of a whimsical web application. In this write-up I’ll show my methodology, the tools I used, the stumbling blocks I hit, and the final root flag.

Link to the room : https://tryhackme.com/room/wonderland

--- 

### **Context**

> *Enter Wonderland and capture the flags.*

As with most TryHackMe rooms, I began by scanning the target for open ports.  

<img width="766" height="218" alt="20250916-Wonderland" src="https://github.com/user-attachments/assets/38edc90d-f744-4f35-bd85-e8c293b87470" />


As expected, port 80 was open, so I accessed the server in a browser. While the web service was the obvious entry point, I also ran a directory brute-force to look for hidden paths.

<img width="665" height="385" alt="20250916-Wonderland-1" src="https://github.com/user-attachments/assets/854f8495-a39b-44a6-9daa-8f813cb6ca96" />

We can look at each discovered directory. I started with `/img` : 

<img width="777" height="181" alt="20250916-Wonderland-2" src="https://github.com/user-attachments/assets/45305c4d-8933-43ce-8adf-15ee9d17c861" />

My initial hypothesis was that the images could conceal data. I considered steganographic techniques as a potential vector. I kept that idea in the back of my head and went with the next directory, `/r` : 

<img width="756" height="223" alt="20250916-Wonderland-3" src="https://github.com/user-attachments/assets/b717dbb6-bf43-4246-b553-52916811694e" />

The cryptic “Keep Going” prompt was interpreted as an indicator to proceed with deeper directory enumeration. And that's precisely what I did.

<img width="632" height="302" alt="20250916-Wonderland-4" src="https://github.com/user-attachments/assets/14b7e2c3-0e74-458d-833c-348fbccf5b8a" />

We find the directory `/a` : 

<img width="806" height="213" alt="20250916-Wonderland-5" src="https://github.com/user-attachments/assets/85f36ed9-93ac-4ce3-b843-058d5d3d7070" />

I quickly understood that by continuing to enumerate directories, I would eventually end up finding the final path : `/r/a/b/b/i/t/`, quite fitting for the Alice in Wonderland's theme of the room.

<img width="1884" height="955" alt="20250916-Wonderland-6" src="https://github.com/user-attachments/assets/8d215b0c-0471-42a1-bfe7-f86a5fa9730b" />

When inspecting the page, we find what looks to be some credentials.

<img width="1917" height="931" alt="20250916-Wonderland-7" src="https://github.com/user-attachments/assets/6123c708-8f57-4c91-82a1-9f2b82c6b389" />

From our `nmap` enumeration, we know that the only other open port on the machine is port 22, corresponding to `ssh`.

And so I tried connecting to `ssh` with the newly found credentials : 

<img width="1371" height="498" alt="20250916-Wonderland-8" src="https://github.com/user-attachments/assets/41b8facd-957d-426b-a6e6-822fdaa7bb45" />

We're in.

The hunt for flags can finally begin. Expecting to uncover `user.txt`, I was surprised to find `root.txt` lying in a home folder instead. Of course I couldn't read its content. I proceeded to search for `user.txt`, suspecting the challenge author may have swapped locations to throw off the usual assumptions.

And my hypothesis was correct. 

<img width="677" height="60" alt="20250916-Wonderland-10" src="https://github.com/user-attachments/assets/58e1fc8f-a059-4f37-9612-7c5964a788a8" />

Now onto the second flag, safely sealed in `root.txt`. As usual, we need to escalate our privileges in order to be able to access its content.

The first thing we can look at is the other file in the `/home/alice` directory : a python script named `walrus_and_the_carpenter.py` : 

<img width="509" height="60" alt="20250916-Wonderland-12" src="https://github.com/user-attachments/assets/3160d801-575b-47c6-a920-d07ce6c3d3d8" />

Not much to see here. 

Looking for privilege escalation vectors, I ran `sudo -l` and found that I could run the python script as the user `rabbit`.

<img width="992" height="146" alt="20250916-Wonderland-14" src="https://github.com/user-attachments/assets/c2b116aa-1438-48cc-86f8-2a3ccb4dcc8d" />

It took me some time to understand that Python libraries aren’t embedded in the interpreter by default. They’re separate scripts or modules that Python loads dynamically from the system’s import path. Knowing that, I noticed the script used a simple `import random` instead of a fully qualified import, meaning Python looked for `random` in its import path. I created a file named `import.py` with my own code so the program imported my file instead of the standard library. That let me execute a payload and obtain a shell as the user who ran the script.

<img width="523" height="67" alt="20250916-Wonderland-13" src="https://github.com/user-attachments/assets/2bdb4090-8437-4dd3-9bcf-6aecc8e97294" />

<img width="813" height="78" alt="20250916-Wonderland-15" src="https://github.com/user-attachments/assets/06f28dc0-d045-4a3b-b259-b8902778c59d" />

In the home directory of rabbit, we find an ELF file named ``teaTime``. My first reflex was to run strings on the file hoping to find useful information about the executable. Unfortunately, `strings` was not available on the target machine. 

<img width="1891" height="171" alt="20250916-Wonderland-17" src="https://github.com/user-attachments/assets/94e2a69c-a415-4920-b4ae-9b4d1cee4472" />

I could however transfer the file to my local machine by hosting an HTTP server with :

```
python3 http.server
```

and

```
wget http://TARGET_IP:8000/teaParty
```

Once on my machine, I ran ``strings`` on the file : 

<img width="645" height="403" alt="20250916-Wonderland-18" src="https://github.com/user-attachments/assets/e3d4ab43-0cef-464b-b508-5214461b13dd" />

I noticed that `date` was being called without an absolute path. Like the earlier Python script, that allows attackers to run their own commands by creating a file named `date` in the current directory and adding it to the PATH.

<img width="1121" height="231" alt="20250916-Wonderland-19" src="https://github.com/user-attachments/assets/1e70c1c1-6edd-4ae5-88e6-56ccd18c9c72" />

Now that we have a shell as `hatter`, we can access his home directory, which contains a file `password.txt`, which we can only guess is the password to `hatter`'s account.

<img width="954" height="95" alt="20250916-Wonderland-20" src="https://github.com/user-attachments/assets/c1dec81a-8f35-4f86-b73b-e0bf50fc069c" />

It seems we have hit a dead end. In fact, I didn't find any vulnerability by hand, and decided to use `linpeas.sh` to find more. 

<img width="930" height="890" alt="20250916-Wonderland-21" src="https://github.com/user-attachments/assets/e7dac288-a1ed-44dd-b443-3aec13068883" />

With the help of `linpeas`, I discovered that `perl` had capabilities set. Searching on GTFObins, I found that this could be exploited to escalate privilege.

<img width="836" height="253" alt="20250916-Wonderland-22" src="https://github.com/user-attachments/assets/05bf1f87-b68a-4db2-95de-77fcfecee2aa" />

Unfortunately, the shell I currently had did not allow me to run the command. So I exited ssh, and logged in as `hatter` with the password previously found. Then, I was able to run the command and escalate privileges to `root` and open `root.txt`.

<img width="1095" height="605" alt="20250916-Wonderland-23" src="https://github.com/user-attachments/assets/63802e03-b818-4365-86e3-750bac08b40d" />


---
# Conclusion 

Wonderland was a satisfying chain of small "aha" moments: web enum → hidden creds → SSH → abusing Python imports → PATH hijack on an unqualified `date` → `perl` capability to root. It reminded me that tiny oversights compound into full compromises and that combining manual detective work with targeted tools pays off. 
This room also taught me to always look for relative paths, which can easily be exploited to run arbitrary commands. 
