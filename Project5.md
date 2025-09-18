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

<img src="https://i.imgur.com/KvFYPro.png" width="500" height="1000" />

Before even unzipping the downloaded folder, since this malware is oriented towards Windows, it is best that I transfer the file to my FlareVM machine, as not only is it based on Windows 10 Enterprise, but it is also completely isolated from my network, meaning it has no way of escaping my isolated VM through a network connection.

<img src="https://i.imgur.com/0OSmzY2.png" width="500" height="1000" />

<img src="https://i.imgur.com/3yosolZ.png" width="500" height="1000" />

Unzipping the file, I am met with a bunch of contents related to the malware, up to and including the malware's libraries.

<img src="https://i.imgur.com/pFB6G0w.png" width="500" height="1000" />

<img src="https://i.imgur.com/pFB6G0w.png" width="500" height="1000" />

Importing "build.exe" from the libraries folder provides me with a SHA-256 hash. Plugging this hash into VirusTotal gives us information about RedLine.Client.exe. 

<img src="https://i.imgur.com/ZKzXA5V.png" width="500" height="1000" />

<img src="https://i.imgur.com/h8A2Hr4.png" width="500" height="1000" />

When evaluating the executable's assembly in Ghidra, however, I did not find anything notably interesting.

I did notice when analyzing "builder.exe", that a snippet of code exists where the program utilizes the Microsoft Runtime Execution Engine (MSCOREE.DLL) to import a function called "_CorExeMain".

<img src="https://i.imgur.com/0gBZsTB.png" width="500" height="1000" />

I still could not find anything other than that function import, however, so I pivoted to the last executable file in the Libraries folder, "stub.exe". This is where I started seeing very suspicious functions. With just the function name, however, I could not exactly figure out what the functions were performing. I needed to somehow find more evidence to prove that the functions were actually performing what is specified on their names.

<img src="https://i.imgur.com/OdVWIlP.png" width="500" height="1000" />

<img src="https://i.imgur.com/LTm1DdV.png" width="500" height="1000" />

<img src="https://i.imgur.com/LhATgYH.png" width="500" height="1000" />

<img src="https://i.imgur.com/pdSe7ft.png" width="500" height="1000" />

<img src="https://i.imgur.com/QfuAG2d.png" width="500" height="1000" />

There's a lot more very suspicious functions, however, I would like to evaluate each and every one of them once I get information on what they actually do. For now, I would like to evaluate Redline's main panel and see what it can actually do. There's a lot of UIs about browser contents, IP contents, FTP connections, etc in the actual UI itself. I didn't even realize this program had the potential to steal credit card credentials, I didn't see that in the files I evaluated.

<img src="https://i.imgur.com/ydp766Q.png" width="500" height="1000" />

<img src="https://i.imgur.com/p1nJOk3.png" width="500" height="1000" />

Looking into the malware's config file, we can see a lot more about what those functions from earlier really were about. There's settings about browser grabbing, file grabbing, FTP connection grabbing (most likely to intercept file transfers), and IM clients, which is a pretty interesting addition in my opinion. I do wonder if it would be able to crack signal chats, but it's best for me to not toy around with the malware.

<img src="https://i.imgur.com/6HEOK9v.png" width="500" height="1000" />

After safely detonating the "build.exe" file in FlareVM, I was eventually able to evaluate the new execuatble file in ILSpy and come across many functions related to the file's behavior, which I will start listing now.

**Crypto**
BouncyCastle AESFastEngine: An engine within the BouncyCastle cryptography library used for AES algorithm implementation.

<img src="https://i.imgur.com/dnphMkB.png" width="500" height="1000" />

Gcm and IAead Block Cipher Algorithms: Likely used to decipher encryption algorithms in order to break into cryptocurrency wallets.

<img src="https://i.imgur.com/gn5J2BK.png" width="500" height="1000" />

Decrypt method for decrypting GCM block ciphers:

<img src="https://i.imgur.com/2EGVnwy.png" width="500" height="1000" />

**Browsers**

Method specifically for Microsoft Edge, but it seems to have nothing directly inside the method, maybe because Edge's engine relies on Chromium now? Nonetheless, it also returns a variable "Browsers" containing the browsers that the malware found using the "ParseBrowsers" variable.

<img src="https://i.imgur.com/I6pvf2q.png" width="500" height="1000" />

**Broswers: ParseBrowsers Method**

The method checks for any browser folders in the "C:\Users\<user>\AppData\Roaming" directory, calls a bunch of methods, and adds the results of every method to a list "browserProfiles".

<img src="https://i.imgur.com/0dJjSsE.png" width="500" height="1000" />

<img src="https://i.imgur.com/rB70pv1.png" width="500" height="1000" />

**Browsers: GetCredentials Method**

The method searches for a "Profile Data" file and collects data from an SQL connection. These values are parsed from the "ReadData" method.

<img src="https://i.imgur.com/EKt7ZfB.png" width="500" height="1000" />

<img src="https://i.imgur.com/xJcFRHv.png" width="500" height="1000" />
