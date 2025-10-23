# Semai Lab DFIR: Simulating and Detecting an Attack - Scenario 4: Malware Analysis w/ HackTheBox Sherlock - Safecracker

## Objective

This DFIR-based home lab series of projects aims to establish a controlled environment for simulating and detecting various types of cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system and simulate scenarios from basic learning scenarios to imitating real world scenarios. In this part of the project, I will be conducting an analysis on actual malware that has been utilized in many sophisticated attacks to analyze the behavior of the malware, as well as any other notable information regarding it.

### Skills Learned

- Advanced understanding of malware analysis and threat hunting
- Development of knowledge of malware behavior and the tools used in conducting basic malware analysis
- Development of critical thinking and problem-solving skills in cybersecurity.
- Advanced understanding of threat mitigation

### Tools Used

- Type-2 Hypervisor for Virtual Machine Cluster
- Windows and Linux machines for various DFIR environments
- Malware Analysis tools for code decompilation and extensive code analysis



## Steps

In the last project, we conducted a malware analysis on the Redline Stealer malware. This project has overlap with HackTheBox Sherlock, as I will be going through their "Safecracker" sherlock to learn how to identify various notable areas of malware samples, and how to document them. This project will also be available in the HTBSherlockWriteups section of semailab! (WARNING: If you are following along, isolate a virtual environment, disabling any and all internet access to the machine first before executing any kind of malware. I am not responsible for any damages caused by malware misuse!)

# Task 1: Which user account was utilised for initial access to our company server?

After installing the "safecracker.zip" file provided for us in the challenge, we're met with multiple files pertaining to the sherlock.

<img src="https://i.imgur.com/KvFYPro.png" width="500" height="1000" />

For this question, I immediately navigated to "log.json", and I was met with multiple logs pertaining to the incident. Querying a search for "Users" along with a little bit of scrolling and we can find the first user account the attacker targets. 

<img src="https://i.imgur.com/S1Enmml.png" width="500" height="1000" />

# Task 2: Which command did the TA utilise to escalate to SYSTEM after the initial compromise?

Since we know the account name that the attacker first targeted, let's take a deeper look into that specific user's files in Autopsy! The first place that I decided to look at, which is pretty common for command execution, was the user's PowerShell folder, and to no surprise, a PowerShell log history was found. Opening the PowerShell log gives us the exact command used to escalate to SYSTEM (PSExec is commonly known for Lateral Movement according to [MITRE](https://attack.mitre.org/software/S0029/)).

<img src="https://i.imgur.com/HKtNptv.png" width="500" height="1000" />
