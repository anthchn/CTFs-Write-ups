# Boogeyman 2

## Introduction 

**📊 Difficulty**: Medium

Link to the room : https://tryhackme.com/room/boogeyman2

Boogeyman 2 is the second installment in the Boogeyman trilogy, a series of rooms on [TryHackMe](https://tryhackme.com) part of the SOC Level 1 Capstone Challenges module and is designed to test and demonstrate the various skills developed throughout the [SOC Level 1](https://tryhackme.com/path/outline/soclevel1) Learning Path.

In this room, an employee opens a malicious attachment from a spearphishing email.

We are tasked with analysing the TTPs employed by the attacker to carry out their attack.

We are provided with :
- A copy of the phishing email.
- A memory dump of the victim's workstation.

In this room, we will be using the following tools : 
- _Volatility_ - an open-source framework for extracting digital artefacts from volatile memory (RAM) samples.
- _Olevba_ - a tool for analysing and extracting VBA macros from Microsoft Office documents. This tool is also a part of the Oletools suite.

---

## Context 

>_”Maxine, a Human Resource Specialist working for Quick Logistics LLC, received an >application from one of the open positions in the company._
>
>*Unbeknownst to her, the >attached resume was malicious and compromised her workstation.*
>
>*The security team was able to flag some suspicious commands executed on the >workstation of Maxine, which prompted the investigation. Given this, you are tasked to >analyse and assess the impact of the compromise.”*

Let’s start by opening the phishing e-mail provided in the Artifacts folder.

<img width="1919" height="809" alt="Capture d'écran 2025-08-12 150427" src="https://github.com/user-attachments/assets/3b3b127b-ea3f-4e06-9438-9eae2173a6fd" />
<img width="801" height="229" alt="Capture d'écran 2025-08-12 150627" src="https://github.com/user-attachments/assets/43528901-ed65-4888-9c0c-6ba80691e20d" />

As we can see, maxine.beck@quicklogisticsorg.onmicrosoft.com received an e-mail ‘Resume - Application for Junior IT Analyst Role’ from westaylor23@outlook.com with a file named Resume_WesleyTaylor.doc attached.

### Q. What email was used to send the phishing email?

_Answer_ : westaylor23@outlook.com

### Q. What is the email of the victim employee?

_Answer_ : maxine.beck@quicklogisticsorg.onmicrosoft.com

### Q. What is the name of the attached malicious document?

_Answer_ : Resume_WesleyTaylor.doc

### Q. What is the MD5 hash of the malicious attachment?

When investigating a potentially malicious file, it’s always good practice to look up the hash of said file. Plus, this can be easily achieved with a simple command : 

```bash
md5sum Resume_WesleyTaylor.doc
```
<img width="795" height="49" alt="Capture d'écran 2025-08-12 150716" src="https://github.com/user-attachments/assets/50dd8345-9007-4afd-b5c7-1fdffb52f98c" />

_Answer_ : 52c4384a0b9e248b95804352ebec6c5b

Looking up the obtained hash on VirusTotal, we can see that 37 Antivirus vendors flagged the file as malicious. Additionally, the site gives us insight concerning the capabilities of the malware.

<img width="1707" height="842" alt="Capture d'écran 2025-08-12 150906" src="https://github.com/user-attachments/assets/4756f773-c099-43ff-a3be-6b01b3f3b53d" />

### Q. What URL is used to download the stage 2 payload based on the document's macro?

Reading the Code Insight provided by VirusTotal, we understand that upon execution, the malicious file triggers a macro named AutoOpen, establishes a connection with the server located at https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png and saves the response in a file named update.js located in a ProgramData folder. The downloaded script is then executed. 

### Q. What is the full file path of the malicious stage 2 payload?

The answer can be found in the Code Insight mentioned above.

Other ways we could have found the answer would have been to look at the Behaviour tab in VirusTotal under MITRE ATT&CK Tactics and Techniques.

<img width="982" height="359" alt="Capture d'écran 2025-08-12 151205" src="https://github.com/user-attachments/assets/d8910ea0-2296-4222-9245-df01403d6571" />

Since the file is a `.doc` document, we also could have used `Olevba` to extract the VBA macros as shown below : 

<img width="1265" height="722" alt="Capture d'écran 2025-08-12 151519" src="https://github.com/user-attachments/assets/0f54a66b-bba3-415f-aee2-254fae49cf04" />


_Answer_ : https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.png

### Q. What is the name of the process that executed the newly downloaded stage 2 payload?

_Answer_ : wscript.exe

### Q. What is the full file path of the malicious stage 2 payload ?

_Answer_ : C:\ProgramData\update.js

In the following, I will use several Volatility plugins, which are modules that extract specific types of information from a memory dump. Since the memory dump was taken from a Windows machine, each plugin will be run with the windows. prefix. To identify all available plugins, I referred to the list provided by the volatility --help command.

### Q. What is the PID of the process that executed the stage 2 payload?

From the previous question, we know that `wscript.exe` executed the payload. To find the PID, we have to run Volatility on the provided memory dump. 

Volatility has a very useful plugin that allows us to easily output the PID of all processes.

```bash
vol -f WKSTN-2961.raw windows.pslist | grep wscript.exe
```
<img width="1723" height="159" alt="Capture d'écran 2025-08-12 151848" src="https://github.com/user-attachments/assets/ad81a7ce-9056-490d-bc89-7a817a7b0c8c" />

_Answer_ : 4260

### Q. What is the parent PID of the process that executed the stage 2 payload?

Volatility’s `pstree` plugin is basically the same as the previously used `pslist` except is also outputs the link between processes. 

```bash
vol -f WKSTN-2961.raw windows.pstree | grep -z wscript.exe
```

<img width="1298" height="115" alt="Capture d'écran 2025-08-12 152004" src="https://github.com/user-attachments/assets/dde91798-74b5-4f6a-8680-b67df6bfa53e" />

We could have ran `pstree` to answer both questions.

We find that the parent process of `wscript.exe` was `WINWORD.exe` with the `PID 1124`.

_Answer_ : 1124

### Q. What URL is used to download the malicious binary executed by the stage 2 payload?

I was so confused when trying to answer this question. I though another URL was hidden in the update.js script downloaded upon execution of the malicious attachment. I tried desperately to retrieve the update.js file hoping to read the script and find a URL in it but to no avail. 
 
Instead, I figured that the URL I thought would be in the update.js script was somewhere else. The only file I could test that theory on was the memory dump itself. So I extracted all strings contained in the file and looked for URLs. 

I got a bunch of URLs and decided to filter further by searching for the domain previously obtained : 

```bash
strings WKSTN-2961.raw | grep https://files.boogeymanisback
```
<img width="1002" height="112" alt="Capture d'écran 2025-08-14 193131" src="https://github.com/user-attachments/assets/c87d849d-c458-4233-9cea-efac4de011e5" />


Success !

_Answer_ : https://files.boogeymanisback.lol/aa2a9c53cbb80416d3b47d85538d9971/update.exe

### Q. What is the PID of the malicious process used to establish the C2 connection?

Same as before : to find the PID, all we need to do is to run Volatility with the `pslist` plugin.

```bash
vol -f WKSTN-2961.raw windows.pslist | grep update
```

_Answer_ : 6216

### Q. What is the full file path of the malicious process used to establish the C2 connection?

The only plugin I thought would output the full path of processes/files was the very useful `cmdline` plugin.

And my instincts were right, as we find the answer to the question almost immediately.

```bash
vol -f WKSTN-2961.raw windows.cmdline | grep update
```

<img width="746" height="122" alt="Capture d'écran 2025-08-12 155230" src="https://github.com/user-attachments/assets/3d759ece-6f8c-44ac-8555-059ac6e89ce6" />

_Answer_ : C:\Windows\Tasks\updater.exe

### Q. What is the IP address and port of the C2 connection initiated by the malicious binary? (Format: IP address:port)

In this question, we are asked to find network connections. 

We can use the `netstat` plugin to identify all network connections present at the time of extraction of the memory dump.

Filtering to only output connections involving updater.exe, we find the following :

```bash
vol -f WKSTN-2961.raw windows.netstat | grep updater.exe
```

<img width="1490" height="340" alt="Capture d'écran 2025-08-12 155726" src="https://github.com/user-attachments/assets/9fbbeb3e-c6a2-4f84-9ac9-4c70e42018e4" />

_Answer_ : 128.199.95.189:8080

### Q. What is the full file path of the malicious email attachment based on the memory dump?

Using the `filescan` plugin, which scans the files accessed stored in the memory dump, we find the full path of the .doc file.

<img width="1580" height="75" alt="Capture d'écran 2025-08-12 155853" src="https://github.com/user-attachments/assets/6a5e5eee-5af4-4b89-9043-978415d2fda6" />


_Answer_ : C:\Users\maxine.beck\AppData\Local\Microsoft\Windows\INetCache\Content.Outlook\WQHGZCFI\Resume_WesleyTaylor (002).doc

### Q. The attacker implanted a scheduled task right after establishing the c2 callback. What is the full command used by the attacker to maintain persistent access?
Once again, I hit a dead end. I parsed through the process list, searching for schtasks or anything resembling a task scheduler like autorun but came up empty.
After long minutes, I decided to go back to my earlier approach and extract all strings from the memory dump in the hope of finding something. Luckily, it paid off as multiple instances of the string schtasks appeared, along with what seemed to be a command line linked to task scheduling.

<img width="1919" height="496" alt="Capture d'écran 2025-08-12 163046" src="https://github.com/user-attachments/assets/d300b0e6-4032-4dd8-ab67-0c0915cdad73" />

_Answer_ : schtasks /Create /F /SC DAILY /ST 09:00 /TN Updater /TR 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -NonI -W hidden -c \"IEX ([Text.Encoding]::UNICODE.GetString([Convert]::FromBase64String((gp HKCU:\Software\Microsoft\Windows\CurrentVersion debug).debug)))\"'

## Conclusion

This room definitely wasn’t as challenging as the previous one, but I still found myself struggling with a few questions. 

I am still not used to checking for `strings` when analyzing files, and that definitely showed. 

Aside from that, the room was a great exercise in memory investigation using one of Volatility’s most useful plugins. Memory analysis is one of the first steps forensic analysts take when examining a device, due to its volatile nature, and it’s a crucial stage in threat hunting. 

Overall, this was a valuable and well-designed exercise and I greatly enjoyed going through it.









