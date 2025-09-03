# Semai Lab DFIR: Simulating and Detecting an Attack - Scenario 1: Password Spraying

## Objective

This DFIR-based home lab series of projects aims to establish a controlled environment for simulating and detecting various types of cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system and simulate scenarios from basic learning scenarios to imitating real world scenarios. In this part of the project, I will be simulating attack scenarios to gain a better understanding of log analysis.

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Ability to ingest and generate network logs.
- Enhanced knowledge of digital forensics and attack mitigation.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
- Type-2 Hypervisor for Virtual Machine Cluster setup
- Windows and Linux machines for various blue and red team environments
- Security Information and Event Management (SIEM) system for log ingestion and forensic analysis.
- Automated background services for attack detection and mitigation.



## Steps
In this first scenario, I will be using my Kali VM that was created in Part 1 of this project series. Kali contains many attack modules that can be utilized to perform various attacks, and in this part of the project, I will demonstrate some of these attack modules, as well as the event logs in elastic that were generated from these attacks.

Before running this malware, I took a snapshot of every machine that I will be using in this scenario, these machines being the Kali VM, the Active Directory VM, and just incase, the FlareVM and the REMnux VM. I probably only need Kali, Kali, and the Domain Controller for this scenario, but I have REMnux and FlareVM at the ready in case I need them.


**Scenario 1: Password Spraying**

Password Spraying is one of the easiest and most common attack techniques as it involves utilizing lists of common usernames and passwords, repeatedly brute-forcing those usernames and passwords throughout an entire server, and analyzing which accounts have successful logins from these attempts. In a very secure environment, this should not be that much of a problem, but even in today's environment it can still be a viable strategy, especially when targeting individuals who might not have much knowledge of internet security.

When scanning the environment with nmap, I noticed that there are a LOT of ports open in the Domain Controller. This is most likely the result of the script I downloaded during the AD setup, which opened the domain controller to many vulnerabilities, such as these open ports that I am now able to exploit.

<img src="https://i.imgur.com/ugMnIjM.png" width="500" height="1000" />

This might make it easy to get Hydra, a password sprayer, up and running in the simplest way possible.

Running Hydra on port 3389, I'm getting a lot of log responses in Elastic, more specifically lots of 4776 event codes. Looking at the logs, it seems that Hyrdra is specifically targetting the "Guest" account, with most of the credentials not seeming to validate at all.

<img src="https://i.imgur.com/MDJ8Qeg.png" width="500" height="1000" />

<img src="https://i.imgur.com/GEsoLYY.png" width="500" height="1000" />

The combination of 4776 and 4634 event codes, which are prevalent in this data visualizer below, is a clear sign that there is some sort of brute force attempt happening to the domain controller, which in this case, is the kali instance brute forcing the RDP port of my domain controller using Hydra.


<img src="https://i.imgur.com/vokCb9I.png" width="500" height="1000" />

In a typical work environment, usually attempts like these would be a warning sign of a brute-force attack, but sometimes it can also just be an employee forgetting their password and repeatedly trying different passwords to try and get back in, which in that case the employee would be contacted to reset their password. In an actual brute force attack however.....

<img src="https://i.imgur.com/ZbfBNPs.png" width="500" height="1000" />

....yeah. I don't think a typical employee would try logging in 25,000+ times....well...unless they're absolutely psychotic.

**Remediation**

In something like this, I experimented with a tool called Fail2ban, which will block an ip with a set amount of consecutive login attempts for a set amount of time in order to stop brute-force attempts from happening. In this scenario, I integrated Fail2ban with Filebeat so that I can ingest logs from Fail2ban through Filebeat, making it easier to track when Fail2ban blocks an IP when it hits the set amount of consecutive login attempts.

Fail2ban installation is a little easier than the past ELK service installations as it comes included in Debian/Ubuntu's APT library. Just sudo apt install file2beat and it's pretty much installed, although in order for Fail2ban to work properly, I need to allow SSH access through UFW and then enable the firewall from there. (NOTE: once UFW is enabled, ports have to be manually added to the UFW whitelist in order for them to function properly.

<img src="https://i.imgur.com/7pJvcwb.png" width="500" height="1000" />

Fail2ban comes with a default configuration file, just like other services in the ELK stack, but this default config file should be kept in the event that issues arise. For now, I'll copy this file to a different file instead. 

<img src="https://i.imgur.com/Bw4ZcCP.png" width="500" height="1000" />

In the config file, I can adjust various factors, including bantime, ban duration multipliers for multiple failed attempts, etc. I'll leave everything at default for now, but this is where Filebeat comes in handy as I'll be utilizing it to store any logs that Fail2ban creates.

Previously, I did not really have a use for the filestream feature in Filebeat as I used the service's default configuration to ingest logs, but since I'm adding fail2ban as a configuration to the service, it's a good time to enable this as well as add fail2ban's fields to the configuration file.

<img src="https://i.imgur.com/mvU0dmK.png" width="500" height="1000" />

Checking fail2ban's state using "fail2ban-client -h" and it seems to be working on the machine as it prints out fail2ban's documentation.

<img src="https://i.imgur.com/QeKPWX1.png" width="500" height="1000" />

Just incase, since I'm likely also going to add more than just Filebeat in the future, especially to ingest more Windows logs, this is probably a great time to setup Logstash.

Using nano and vim, I created a fail2ban.conf file inside the Logstash directory in order for Logstash to recognize Fail2ban.

<img src="https://i.imgur.com/R7XTyIb.png" width="500" height="1000" />

Once that was done, I then created a patterns/ directory and added the following:

<img src="https://i.imgur.com/VMOGELl.png" width="500" height="1000" />

After these expression patterns are added, I then force reloaded logstash. To make things cleaner and easier for Sysmon log ingestion, I also went ahead and installed Winlogbeat onto my domain controller.

Running an nmap scan from my Kali machine now that Winlogbeat is working, I can now see that RDP is definitely the most vulnerable port in the domain controller as my Kali machine got a direct connection established to the RDP port, and Winlogbeat logged that exact event with the IP of my Kali machine.

<img src="https://i.imgur.com/T4cxxRN.png" width="500" height="1000" />

Conducting another password attack on the domain controller and.....it does not seem like file2ban is stopping the attack.

<img src="https://i.imgur.com/hTAl9EY.png" width="500" height="1000" />

Doing a little bit of research and it seems like fail2ban only stops attacks on Linux machines, which means technically i got it working to wear it will block IP addresses that attack the ELK machine, but I wanted to stop IP addresses that attack my Domain Controller. Luckily enough, there is a Windows version of fail2ban called "Fail2Ban4Win" which does the same thing as Fail2Ban, however Fail2Ban4Win only processes logs to its own folder, which would originally be able to be passed through Filebeat....but Filebeat is on my ELK stack VM, not my domain controller VM.

Realistically the only thing I could do here was create an event log source for Fail2Beat4win, forward the log file to the event log using a powershell script, and then make the script run automatically so that Winlogbeat could pass the logs through that way.


<img src="https://i.imgur.com/vWDbiWp.png" width="500" height="1000" />

<img src="https://i.imgur.com/d3cqaX8.png" width="500" height="1000" />

<img src="https://i.imgur.com/qQpaSwN.png" width="500" height="1000" />

After this, Fail2Ban4Win is now displaying logs in the Event log, and now it is time to start the Hydra attack once again from my Kali machine.

<img src="https://i.imgur.com/dukMLVa.png" width="500" height="1000" />

Wait...that's new! The entire connection just dropped after a couple of failed attempts!

<img src="https://i.imgur.com/1b2VU5l.png" width="500" height="1000" />

Going back into the winlogbeat logs, and just like that, our Kali machine got banned from accessing my domain controller.

<img src="https://i.imgur.com/h7sKCTE.png" width="500" height="1000" />

**Conclusion**

Fail2ban is one of the easier and more beginner-friendly ways to secure your instance from simple password attacks like Hydra. Usually when programs like these aren't used in a stealthy way, it's pretty easy to secure systems from simple attacks like these. It does get more difficult when attacks become more sophisticated, but with proper configurations, and maybe some automation thrown into the mix, even those attacks can be stopped. With proper knowledge of both how to attack the system and how to secure it, it gets a lot easier to know how to properly secure an environment.
