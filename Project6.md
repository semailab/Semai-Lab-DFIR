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
