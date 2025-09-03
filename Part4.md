# Semai Lab DFIR: Simulating and Detecting an Attack - Scenario 3: Introduction to Malware and Malware Analysis

## Objective

This DFIR-based home lab series of projects aims to establish a controlled environment for simulating and detecting various types of cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system and simulate scenarios from basic learning scenarios to imitating real world scenarios. In this part of the project, I will be simulating an attack scenario by infecting a domain controller in an isolated environment and introducing the general concept and process of basic malware analysis, as well as learning the tools used for malware forensics.

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Ability to ingest and generate network logs.
- Enhanced knowledge of digital forensics and attack mitigation.
- Development of knowledge of malware behavior and the tools used in conducting basic malware analysis
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
- Type-2 Hypervisor for Virtual Machine Cluster
- Windows and Linux machines for various DFIR environments
- Security Information and Event Management (SIEM) system for log ingestion and forensic analysis.
- Automated background services for attack detection and mitigation.
- Malware Analysis tools for code decompilation and extensive code analysis (Ghidra)



## Steps
In the previous scenario, we demonstrated an attack scenario involving poor security configurations and poor login credentials. This scenario then led to an exploit attempt that became unsuccessful by the end of it, but also led into our next scenario involving malware, which will lead into the "Malware Analysis" section of the project where we begin to reverse-engineer malware in order to determine what it's exactly doing.

**Malware 1**

For this scenario, I'm using a [malware sample](https://github.com/k-perrino/malware-workshop-acm-w) provided to me by the University of Texas at San Antonio's ACM-W club, which is a pretty fun club to get involved in (they also demonstrated the malware sample at a malware analysis workshop which was pretty cool). 

**NOTE: Before executing the malware, PLEASE be sure to isolate your environment from your entire network, or perhaps even disable internet connectivity from the VM entirely. I'm running a different pfsense setup for this environment as winlogbeat still requires an outbound connection to the ELK stack VM in order to work, so I want to disconnect the domain controller from everything but the ELK stack VM, for now.**

Before we isolate the domain controller, we want to actually download and compile the malware first. Since the malware runs on a C file, we're going to have to compile it in order for it to run, but because of Elastic Security being a potential issue, there is a chance we'll have to find a way to compile and then immediately run the file right after compilation, hoping that Elastic Security won't be a problem once again, but first, let's install the C compiler.

After installing a package manager of our choice, which I decided to use Chocolatey, I installed the MinGW package, which includes the GCC compiler normally used for C compilation.

<img src="https://i.imgur.com/ogyJlFC.png" width="500" height="1000" />

Now we're ready to download the malware sample directly from the github page!

<img src="https://i.imgur.com/3BGAHFK.png" width="500" height="1000" />

**NOTE: If you're following along, PLEASE isolate your OS before continuing further! I take no responsibility for misuse! CONTINUE AT YOUR OWN RISK!**

After isolating my domain controller using pfsense, I then compiled the file, which ended up becoming successful so far! Elastic Security did not alert me about the compiled executable file. Time to have some fun!

<img src="https://i.imgur.com/5szxs64.png" width="500" height="1000" />

After executing the file, the program went straight to action. It didn't even take that long for me to find evidence of the executable file editing registry entries right after execution.

<img src="https://i.imgur.com/rPKQdIt.png" width="500" height="1000" />

Not only were registries changed, while filtering logs by "process.name: dns_rev_shell.exe", I found a log that shows a DNS query to a dns called "c2.malicious.local". What makes this stand out from typical DNS queries is the first two characters in the domain: C2. This DNS query is leading to a Command-And-Control infrastructure, an essential tool used by bad actors to control malware on affected devices. If I planted something like this in Celesta's instance in the last scenario, I probably would've been able to gain easier access to her desktop environment, as well as potentially abuse exploits that would lead to privilege escelation, and this is only one scenario that becomes possible with malware.

**Malware Analysis and Mitigation**

Now comes the question, how would we be able to transport this malware to a safe environment to analyze it? We know some of the things that the malware did just from elastic logs, but that's generally not enough as agents like winlogbeat can only log so much. This is where Tsurugi, FlareVM, and Remnux come into play.

After adding a couple of firewall rules that allow communication ONLY to and from Tsurugi and domain controller, I transfer the malicious file from the domain controller, as well as remove the file from the OS completely.

<img src="https://i.imgur.com/nfQJBWm.png" width="500" height="1000" />

Going to the Tsurugi Linux VM, the file was successfully imported to the VM. Analyzing the file here would not be a great idea, however, as it does have open internet access, so let's go ahead and transport it into FlareVM...

<img src="https://i.imgur.com/xzED9g7.png" width="500" height="1000" />

...and also into Remnux, that way we have 2 Analysis VMs to work with, just incase!

<img src="https://i.imgur.com/USNF85w.png" width="500" height="1000" />

Ghidra is a reverse-engineering tool developed by the United States National Security Agency (NSA) with the primary role of reverse-engineering software, making it an essential tool for malware reverse-engineering, which is what I'll be using to decompile our malware.

<img src="https://i.imgur.com/AGI27Uw.png" width="500" height="1000" />

Right after importing the file into my project, I immediately noticed something that could be extremely useful in learning more about this malware before even analyzing the code: The MD5 and SHA256 hashes. These hashes are used to identify any type of file, including malware. A website like VirusTotal takes note of these hashes and can easily identify types of malware just by analyzing the hashes inside an executible file. VirusTotal can sometimes even provide the exact attack strategies associated with malware in order to tell us more about what the malware is exactly trying to do. Unfortunately for us, however, we get no result when plugging the hashes into VirusTotal, so we'll have to investigate a little further (we could've also dropped the entire file into VirusTotal before even running it in the first place, or we could've also used an online file analyzer like Triage to get more details on the file's behavior, but it's also better to reverse engineer the file itself to get a better understanding of exactly what it's doing).

<img src="https://i.imgur.com/RO6BSwq.png" width="500" height="1000" />

<img src="https://i.imgur.com/E0Tr3i7.png" width="500" height="1000" />

After a little bit of digging through the assembly code, I noticed multiple bits of assembly that ghidra was able to decompile to C code I could understand. I'll go through my findings in little bits as I find them.

In this bit of code, something I actually didn't catch while executing the malware in the domain controller was the file planting a file inside the ProgramData folder.

<img src="https://i.imgur.com/ijeodHy.png" width="500" height="1000" />

Ghidra was also able to pinpoint the part of the program that initiated the reverse shell connection to the c2 domain we spotted earlier. After a little bit of googling, I was able to come with the conclusion that the malware utilized ws2_32.dll to attempt to create a socket that leads to the malicious c2 domain, which created a connection from the domain controller to the malicious c2 domain (which probably didnt actually happen since I did isolate the domain controller from the internet before executing the malware).

<img src="https://i.imgur.com/fai605e.png" width="500" height="1000" />

This main script tells us the process the file used to execute the functions we found earlier. the function calls __main(), which initializes the program, then calls "drop_pwned_file()" to drop a file in the ProgramData folder telling us that we've been pwned, then calls the "reverse_shell()" function that creates the socket that connects the infected machine to the malicious c2 domain.

<img src="https://i.imgur.com/GFG9UpC.png" width="500" height="1000" />

**Remediation**

Now that we exactly know what's the file is doing, now we can take the appropriate steps to mitigate it! I had already deleted the files from the domain controller earlier, but now would probably be a good time to delete the file from all of my machines, including FlareVM, Remnux, and Tsurugi. Additionally, it's also probably best to revert every computer to its previous screenshots to fully ensure that the threat is fully removed from our machines used in this scenario.

**Conclusion**

This is only the start of our journey of how malware behaves, how we can identify and detect the type of malware with its behavior, and how we can mitigate it. There are so many malware samples out there free for us to mess around with, but because of the dangerous nature, there will come a point where it will likely be too dangerous to execute malware on our domain controller, which unfortunately means that we won't be able to exactly use the domain controller and ELK stack to learn more about malware behavior inside the computer itself, but we'll also discover other tools that will bridge that gap for us. Buckle up, because this is only the beginning of our malware analysis journey!
