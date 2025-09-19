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

**Browsers: DecryptChromium and DecryptV10**

The method decrypts data from the Chromium-based browser, including its stored v10 cookies.

<img src="https://i.imgur.com/1o5BTOZ.png" width="500" height="1000" />

The rest are methods used to find the exact roaming directory needed for the attack, which were less interesting than my current findings.

**Browsers: Gecko**

A "GeckoDatabase" method converting bit data to string data.

<img src="https://i.imgur.com/KY8FoaD.png" width="500" height="1000" />

The methods in the "GeckoEngine" method were relatively similar to the Chromium methods, however instead of easily grabbing the data, the program was only able to grab keys, which had to be decrypted using various methods, like the method "GetPrivate3Key".

<img src="https://i.imgur.com/W6TZskg.png" width="500" height="1000" />

<img src="https://i.imgur.com/xyLG3La.png" width="500" height="1000" />

"GeckoPasswordBasedEncryption" method specifically for decrypting SHA1 hashes, likely what Gecko uses for password hashes.

<img src="https://i.imgur.com/egkLAEe.png" width="500" height="1000" />

**FTP Services**

Unfortunately, it seems to me like my Security+ study guide was true about FTP servers not being so secure...who knew it would be that easy to use a couple of variables to exfiltrate "plain-text" data from FileZilla?

<img src="https://i.imgur.com/AvrSGWr.png" width="500" height="1000" />

Seems like WinSCP was a little better. The program was able to only harvest registry key data.....but unfortunately the encyrption algorithms for this data seem weak.

<img src="https://i.imgur.com/jzZxD4g.png" width="500" height="1000" />

<img src="https://i.imgur.com/AzJ4pkq.png" width="500" height="1000" />

Looking at the rest of the files, I couldn't mind anything interesting other than the previous methods, however, I did notice two things that could contribute to some of the malware's behavior.

VM Detector, particularly targetting VirtualBox and VMWare instances:

<img src="https://i.imgur.com/kM8iWmx.png" width="500" height="1000" />

UAC Admin prompt permissions changed

<img src="https://i.imgur.com/OnP0CdJ.png" width="500" height="1000" />

**Summary**

Based on my findings, I can conclude that this malware is a credential harvester that builds an executable file that can be disguised as a normal executable file, like an installer, detects if it was ran in a VM, modifies UAC admin prompt permissions to bypass UAC, and extracts credentials from FTP services, web browsers, and crypto wallets. The main focus seems to be on web browsers, where session cookies and login information stored on the browser would be sent to the attacker through the malware itself. For any information that was encrypted, multiple decryptors, such as GCM, AES, V10, and SHA1 decryptors were used to decrypt such information and relay it to the attacker. I was a little stumped when trying to figure out where credit card information was being saved, but my best assumption is that since credit card information can be saved on web browsers, the attacker is able to extract credit card details from web browser data, but I unfortunately don't exactly have evidence to support this claim.

**Conclusion**

Malware analysis and reverse-engineering is a pretty powerful skill when trying to evaluate malware behavior. Decompiling malware and evaluating its code provides lots of clues as to what exactly the malware is trying to do, what type of attack its utilizing, and potential hints for mitigating the malware. Of course, this was just a basic analysis, and something like this could easily be avoided by simply using an antivirus or Endpoint Detection and Response (EDR) tool, but I would like to find more clever ways to mitigate malware in the event that antiviruses or EDRs simply aren't enough to get the job done, but that is for another malware analysis scenario. 
