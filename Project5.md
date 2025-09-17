# Semai Lab DFIR: Simulating and Detecting an Attack - Scenario 4: Malware Analysis w/ Redline Stealer

## Objective

This DFIR-based home lab series of projects aims to establish a controlled environment for simulating and detecting various types of cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system and simulate scenarios from basic learning scenarios to imitating real world scenarios. In this part of the project, I will be conducting an analysis on actual malware that has been utilized in many sophisticated attacks to determine exactly what the malware is doing, including possibly locating any methods of mitigating the malware just from the code itself.

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

In the last scenario, I introduced malware analysis and how a machine could be infected with malware and how malware analysis tools can be utilized to establish a better understanding of how the malware behaves and how to exactly mitigate any threats the malware poses. This time, I will be conducting a malware analysis on malware that has been used to conduct actual attacks on business infrastructure with the goal of completely understanding what the malware is exactly doing and how I can mitigate the threat. **IMPORTANT: Before performing any sort of malware detonation, completely isolate your machine from your network. I am not responsible for any damages caused by any malware misuse!**


**Acquiring the Sample**
Before even conducting an analysis, I'll need to figure out a way to actually grab the malware sample. Luckily for me, [Vxunderground](https://vx-underground.org/) is a public database with the sole purpose of archiving malware samples for educational use, which is where I grabbed the sample for Redline Stealer, a windows-based malware with the purpose of extracting credentials from browsers and other various areas of the system. Even though I have a description of what the malware does, I want to know exactly what the malware is doing in a practical scenario, that way I know exactly what the malware compromised, and potentially find ways to mitigate the threat in a timely manner.



