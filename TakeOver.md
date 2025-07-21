# TakeOver - TryHackMe Write-up


## **Introduction**
**📊 Difficulty**: Easy

TakeOver is a room on TryHackMe which focuses on subdomain enumeration and URL path injection. The goal of this CTF is to identify vulnerable subdomains that could potentially be taken over by attackers.

Link to the room : https://tryhackme.com/room/takeover

---

## **Context**

> _"Hello there,  
> I am the CEO and one of the co-founders of futurevera.thm. At Futurevera, we believe the future is in space. We conduct space research and write blogs about it. We used to help students with space questions, but we are rebuilding our support.  
> Recently, a group of blackhat hackers approached us claiming they could take over our website and are asking for a large ransom. Please help us identify what they could potentially take over.  
> Our website is located at https://futurevera.thm  
> Hint: Don't forget to add the MACHINE_IP to /etc/hosts for futurevera.thm ; )"_

## **Approach**

Based on the room's description, the challenge appeared to focus on subdomain enumeration. My first instinct was to use Gobuster, a tool I became familiar with during Section 9 (Offensive Security Tooling) of the **Cybersecurity 101** path on TryHackMe.

Per TryHack Me, *"Gobuster is an open-source tool used for web directory, DNS subdomain, virtual host, Amazon S3 bucket, and Google Cloud Storage enumeration. It works by brute-forcing with a given wordlist and handling incoming HTTP responses."*

Before starting, I followed the hint from the room's prompt and added `futurevera.thm` to my `/etc/hosts` file, ensuring that my machine could correctly resolve the domain.

```bash
echo MACHINE_IP futurevera.thm >> /etc/hosts
```

Only then did I run GoBuster to enumerate the subdomains.

## **Subdomain Enumeration**

Initially, I considered using the dns mode for brute-forcing subdomains. Here’s a breakdown of the command:

```bash
gobuster dns -d futurevera.thm -w /path/to/wordlist.txt
```
`-d`precise the domain we want to brute force.
`-w` precise the path to the wordlist we want to use.

Fortunately for us, the AttackBox of TryHackMe came with a bunch of pre-downloaded wordlists.
However, the dns mode didn't return any results.

Let's try the whost option : 

```bash
gobuster vhost -u futurevera.thm -w /usr/share/wordlists/Seclists/Discovery/DNS/subdomains-top1million-5000.txt --append-subdomain
```
<img width="739" height="219" alt="GoBuster" src="https://github.com/user-attachments/assets/ff4fa4dc-d715-4096-9382-8d2881325c19" />

**Success !** The scan returned a valid subdomain : `portal.futurevera.thm`

After adding the newly found domain into the `/etc/hosts file`, I tried accessing it via the web browser : 
```bash
echo MACHINE_IP portal.futurevera.thm >> /etc/hosts
```
<img width="1242" height="224" alt="portal.futurevera.thm" src="https://github.com/user-attachments/assets/4b0c4ce3-4810-47b8-9728-f8177b485753" />

Unfortunately, it looks like a dead end. I was really bummed at that moment. Not knowing where to go next, I tried looking at the given context once more...

> _"[...] At Futurevera, we believe the future is in space. We conduct space research and write <mark>blogs</mark> about it. We used to help students with space questions, but we are rebuilding our <mark>support</mark> [...]."_

Two of the words employed here are commonly used in subdomains. And so I tried accessing `blog.futurevera.thm` and `support.futurevera.thm`.

But first, I had to add the two domains into `/etc/hosts` : 
```bash
echo MACHINE_IP blog.futurevera.thm support.futurevera.thm >> /etc/hosts
```
<img width="1918" height="763" alt="support.futurevera.thm" src="https://github.com/user-attachments/assets/d14eef1c-082d-49cf-987c-5fddb9f03d32" />

The two webpages themselves didn't have much to offer. However, looking at the certificates, we stumble on an interesting DNS name.

<img width="1918" height="765" alt="Security risk" src="https://github.com/user-attachments/assets/beac6fec-5012-46ec-b060-b4307b5bfd51" />

Clicking on **Advanced** and on View Certificates, we obtain : 

<img width="987" height="336" alt="Advanced" src="https://github.com/user-attachments/assets/821ea124-95b4-4d84-a6f0-3007f3caa988" />
<img width="981" height="124" alt="flag" src="https://github.com/user-attachments/assets/92ccde0e-7f77-4077-baba-b77a7bf80c00" />

Entering the DNS name into the URL search bar, we are redirected to another URL, which contains the flag to complete the room.

<img width="893" height="39" alt="Capture d'écran 2025-07-15 204435" src="https://github.com/user-attachments/assets/cc00ca21-9ae9-49ac-9c5e-4ec7d901be33" />

## Conclusion

This was my very first CTF challenge. I dove in right after finishing the Cybersecurity 101 module on TryHackMe. 

As my first CTF, I found this challenge to be quite educational. I was a bit disappointed that Gobuster didn’t yield any useful results, especially since I was eager to apply what I had learned during the Cybersecurity 101 path on TryHackMe. In the end, no tools were actually needed to complete the room but instead only careful observation and logical thinking, which are key skills to develop in cybersecurity. 

The key lesson from "TakeOver" is that the solution is often right in front of you, and sometimes the most effective attacks are based on simple, overlooked details like how URLs are structured or how subdomains are configured.
