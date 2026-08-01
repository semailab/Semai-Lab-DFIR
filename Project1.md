# Semai Lab DFIR: Cluster Setup

## Objective

This DFIR-based home lab series of projects aims to establish a controlled environment for simulating and detecting various types of cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system and simulate scenarios from basic learning scenarios to imitating real world scenarios. This hands-on experience was designed to deepen understanding of network security, attack patterns, defensive strategies, and also basic malware analysis.

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Ability to ingest and generate network logs.
- Enhanced knowledge of Certificate Authorities and the Windows Active Directory Environment
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
- Type-2 Hypervisor for Virtual Machine Cluster setup
- Windows and Linux machines for various DFIR environments
- Security Information and Event Management (SIEM) system for log ingestion and analysis.



## Steps
**Network Diagram**

To better visualize how my home lab will look like during basic setup, I created a network diagram that will showcase my environment upon completion. 

<img src="https://i.imgur.com/ClQwiXj.png" width="500" height="1000" />

**pfSense Setup**

Once my network diagram was finished, I grabbed a pfSense ISO file and started installing the firewall onto my VirtualBox installation. Once it finished installing, I then configured the virtual network adapter interfaces that I will be using in this portion of the homelab setup. Later on there will be more ports that I will be adding, but I will only need these interfaces for now.

<img src="https://i.imgur.com/StHJ0oT.png" width="500" height="100" />

**Kali Linux, pfSense Firewall, and Cyber Range Setup**

I then set up my main environment, which I chose to be a Kali Linux environment as it is the linux distro I am more familiar with, and it also has a bunch of nice tools that I could use to conduct my own vulnerability assessments of the other machines that I will be creating throughout this project. Once my main environment was setup, I was able to access the pfSense dashboard through pfSense's provided DNS server. From here I set up my own domain and hostname for the lab, as well as changing the password of the admin user. 
<img src = "https://i.imgur.com/4QJh9qh.png" width="700" height="500" />

Once setup was completed, I set up the firewall rules for the LAN, CYBER_RANGE, and AD_LAB subnets. These rules ensure that traffic flows to the subnets they are supposed to flow through, and that traffic does not flow to subnets they are not supposed to flow through.

<img src = "https://i.imgur.com/47PT5xm.png" width="1000" height="200" />
<img src = "https://i.imgur.com/R5kZBEL.png" width="1000" height="250" />
<img src = "https://i.imgur.com/eAvMFbH.png" width="1000" height="200" />

After all the firewall rules were setup, I then set up Metasploitable 2 and Chronos instances, which were super simple, as all I had to do was download the ISOs, drop them in VirtualBox, and they were ready to go immediately (other than chronos, still figuring out how to get the login for chronos as of writing this...).

**Active Directory Setup**

My Active Directory environment consists of 3 Windows VMs, one of them being the Domain Controller (running Windows Server 2019), and the other two serving as clients for the Domain Controller (both running Windows 10 Enterprise). After setting up my Domain Controller VM, I then installed all of the essentials required for the machine to serve as the Domain Controller, such as the Active Directory Certificate Service and the Active Directory Domain Service. Once those were installed and running, I then configured the DNS and DHCP services so that the clients are able to properly connect to the domain controller. The DNS service will be configured with a forwarder that will forward DNS queries to pfSense if the domain controller cannot resolve them. The DHCP will assign random IP addresses to the two clients in order for them to connect to the domain controller's DNS. The Active Directory Certificate Service is the "Certificate Authority" of the domain controller, which will be responsible for signing, issuing, and storing digital certificates.

<img src = "https://i.imgur.com/U6cys6Y.png" width="1000" height="500" />

In this active directory environment, the other two machines should serve as clients, not administrators, so in order to do this, I created three users in the directory. One user, which will be under my name and was used during the AD setup, and two other users which will be under different names. These two users will be the logins for the other two machines.

<img src = "https://i.imgur.com/lHVUUtL.png" width="200" height="100" />

And since this entire lab was supposed to be for DFIR purposes, it will be a lot more fun to have at least one vulnerable machine in the entire environment, thus, I executed a script that created multiple vulnerabilities up to and including the addition of multiple random users into the AD. This, in turn, is why my AD user list currently looks like this:

<img src = "https://i.imgur.com/nJwJ35t.png" width="500" height="350" />

After this, all that was left to do was setup the other 2 VMs, add them to the AD domain under their correct users, and the AD environment is complete.

Before moving forward, I added 2 new interfaces to pfSense through VBoxManage and configured their firewall rules in order to continue with the project, as VirtualBox only allows 4 interfaces to be created by default.

<img src = "https://i.imgur.com/WhV9iXb.png" width="1000" height="200" />
<img src = "https://i.imgur.com/XeU7cm6.png" width="1000" height="100" />

**Tsurugi Linux Setup**
Setting up this system was supposed to be as easy as setting up Metasploitable, Kali, and Chronos, however, because of the .iso file on the download page being broken for some reason, I ended up just downloading the .ova file instead, as I was already using VirtualBox for this project, which allowed me to use .ova files for installation purposes. 

**ELK Stack Setup**
In order to fully understand what exactly is happening whenever malware is deployed onto my vulnerable machines, I'll need to find some way to capture data, particularly logs and security events. This is where the ELK Stack (Elasticsearch, Logstash, Kibana) comes into play, as all three play major roles in my DFIR environment, which can be seen when I eventually start deploying malware samples into my vulnerable machines.

Firstly, however, in order to get ELK stack working, they must be configured in its own virual machine, which I ended up using Ubuntu Server for, as the setup is pretty simple and has everything I need to get the ELK stack working, such as OpenSSH server.

<img src = "https://i.imgur.com/yh48fOC.png" width="500" height="350" />

After setting up apt to allow TLS/SSL downloads from external repositories, I was able to use APT to install the first of the stack, Elasticsearch by downloading its GPG (GnuPrivacyGuard) key and its repository to the machine.

<img src = "https://i.imgur.com/mXC5m1C.png" width="1000" height="200" />

In order to get elasticsearch running, I'll need to configure a couple of things in the YAML configuration file that came with the service using vim. I'll need to set up a name for the cluster, which I ended up naming after the entire lab's name, and also set the ip and port of the network host and http port (NOTE: discovery.type is not included in the YAML file by default and must be manually added.

<img src = "https://i.imgur.com/HVAdaWP.png" width="500" height="350" />

<img src = "https://i.imgur.com/0Dg3Hnj.png" width="500" height="350" />

I am now able to start and enable Elasticsearch using systemctl, browsing to our set network host address over our set http port and ensuring it is accessible.

<img src = "https://i.imgur.com/F4Ksb6y.png" width="500" height="350" />

Setting up Kibana is pretty similar. Simply install the repository, configure the YAML file with the proper server port, host, and name, and starting and enabling the service, while also restarting Elasticsearch in order to allow the service to recognize that Kibana is installed.

<img src = "https://i.imgur.com/ldAsg0r.png" width="500" height="350" />

Both services should now be running.

<img src = "https://i.imgur.com/182QBha.png" width="500" height="350" />

**NOTE**: Logstash is technically not required for this part of the setup, but I ended up installing it anyways in case I need it for any future activities that may require it, which in that case logstash setup will likely be included in that project's page instead of here, and this part of the writeup should contain a direct link to that project's page.


Next, I'll install Filebeat, which will play a role in collecting and stashing my necessary logs, being sure to configure filebeat's YAML file in vim to ensure that it contains the Elasticsearch host address in order to connect to it.

<img src = "https://i.imgur.com/o6f88B9.png" width="500" height="350" />

I also need to tell Filebeat that I'm currently not using Logstash logging and that the host in the configuration file is the right one.

<img src = "https://i.imgur.com/9dZnHCD.png" width="500" height="350" />

Once filebeat is up and running, it's best to check that all indices are working properly using a curl command so that logs can be ingested properly.

<img src = "https://i.imgur.com/7Y5ex9G.png" width="500" height="350" />

**Creating a Certificate Authrority**

Now that the ELK stack has been mostly setup, before I do anything with the SIEM, there is something very important that I'm missing, and that important thing is security. Technically since this is a home lab it would be okay to run everything on HTTP and without a certificate authority, but if I'm going to be getting my hands dirty with malware in the lab, it would be best to secure the environment a bit more. Besides, it would also restrict a lot of features in the ELK stack if I don't implement a certificate authority anyways.

To start things off, in the usr/share/elasticsearch directory, I created an "instances.yml" file and listed every instance with the IP of the machine.

<img src = "https://i.imgur.com/BaoyELH.png" width="500" height="350" />

Using elastic's certutil feature, I created a Certificate Authority bundle that provided me with a zip folder. Upon unzipping such folder, I now have a ca/ directory, a certificate file, and a matching key for the Certificate Authority.

<img src = "https://i.imgur.com/Ndjl8b7.png" width="500" height="350" />

I will use the same certutil feature to also generate our certificates that will be signed by the certificate authority.

<img src = "https://i.imgur.com/BeZUawP.png" width="500" height="350" />

After unzipping the bundle, I made a new directory, certs/, that I will copy the files provided to me from the bundle generates by the certutil feature to.

<img src = "https://i.imgur.com/58XKuP7.png" width="500" height="350" />

The bundle provided us with two types of certificates: public certificates and dedicated certificates that lead to each service in instances.yml. To eliminate any confusion and to provide the proper certificates to the certificate authority, I created ca/ and certs/ directories in each service's directories and copied each certificate to their respective directories:

Public certs to the ca/ directory:

<img src = "https://i.imgur.com/NP4pung.png" width="500" height="350" />

Dedicated certificates to the certs/ directory:

<img src = "https://i.imgur.com/KP9e4g5.png" width="500" height="350" />

After a bunch of cleaning up of leftover files, as well as managing a couple of permissions around (so that elk can have access to elasticsearch/ and kibana/ because for some reason it doesn't and i have to do that manually), I printed all of my certificate information to the console, ensuring my certificates were properly generated.

<img src = "https://i.imgur.com/Ajp3TG8.png" width="500" height="350" />

Now I can add security features to the Elasticsearch and Kibana services.

Elasticsearch.yml:

<img src = "https://i.imgur.com/VFqa6WF.png" width="500" height="350" />

Kibana.yml:

<img src = "https://i.imgur.com/I3GUBT8.png" width="500" height="350" />

From here, I can't quite connect to the host using curl -XGET, and this is because A. The machine doesn't trust the certificate authority and B. With our security features enabled, authentication is required to connect to the host.

<img src = "https://i.imgur.com/I03ydH3.png" width="500" height="350" />

(For security reasons, I'm not providing screenshots for these next two steps.)

To grab the credentials I need, I simply generate them using elasticsearch's setup-passwords utility.

The kibana_system credentials are important for the kibana.yml file in order for kibana to interact with elasticsearch, so I went ahead and copied the kibana_system credentials to kibana.yml.

A quick restart of both Elasticsearch and Kibana and I am now able to access Elastic on my Kali machine!

<img src = "https://i.imgur.com/Nqqupr4.png" width="500" height="350" />

**Fleets**

Fleets are agents that can be deployed to any machine and is responsible for data collection and log ingestion. This will be pretty important once I start experimenting with malware on the Active Directory machine.

Once I set my fleet server host to the ELK stack VM's address, I will then recieve a service token, which I'll need later on if I want to access the elasticsearch database.

Going back to my ELK stack machine, I then install the required files in order for the fleet agent to work.

<img src = "https://i.imgur.com/Gtr793d.png" width="500" height="350" />

Returning to the interface and copying the fleet install commands...

<img src = "https://i.imgur.com/HfWccNA.png" width="500" height="350" />

...the fleet service should now be running!

<img src = "https://i.imgur.com/Ja6Hrju.png" width="500" height="350" />

In the fleet settings, all I need to do now is change my elasticsearch host to the host of the ELK stack VM, and now I am ready to install an agent to the vulnerable AD machine!

The nice thing about fleet is that it has a Windows Endpoints policy specifically for Windows machines, which makes it super easy to ingest PowerShell logs as well as Sysmon logs, making it easier to pinpoint suspicious activity in our ingested logs.


<img src = "https://i.imgur.com/FJNPobN.png" width="500" height="350" />

A certificate import from ELK Stack VM to Active Directory VM, as well as a simple Fleet Agent install command later, and my DFIR environment is ready!

<img src = "https://i.imgur.com/ydYfreg.png" width="500" height="350" />


**Conclusion**
Throughout this setup, I learned a lot about the behind-the-scenes of SIEM setup, and although this is only a homelab-oriented setup, it is pretty essential to learn the basics of how to set up clusters in order to get an understanding of how several features, including certificate authorities, are configured and ran in a corporate environment. Not only can these skills be used in a homelab environment, but they can be also used in competitive environments, including competitions like CyberForce and CCDC, providing me with a useful skill for my university's teams.
