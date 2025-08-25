# Intrusion Detection System (IDS) Homelab Project 
## Project Goal and Description
This project details the setup, configuration, and validation of a virtualized Intrusion Detection System (IDS) within a home lab environment. The primary objective was to deploy Suricata on an Ubuntu server to actively monitor network traffic for suspicious activities and potential threats between simulated attack (Kali Linux) and vulnerable (Metasploitable2) machines. This project demonstrates proficiency in network security principles, system administration, and methodical troubleshooting.
## Technologies Used
Virtualization: VirtualBox
Operating Systems: 
- Kali Linux (Attacker VM - 192.168.5.10)
- Metasploitable2 (Vulnerable Target VM - 192.168.5.20)
- Ubuntu Server (IDS/Monitor VM - 192.168.5.30)
- IDS Software: Suricata
- Network Scanning: Nmap, Ping
- Documentation and Version Control: Git, Github, and Markdown
## Project Architecture
The homelab consists of three virtual machines connected on a dedicated Internal Network within VirtualBox (vlab_net), allowing them to communicate with each other while being isolated from the host machine's network. The Ubuntu-IDS VM is configured with a second network adapter (NAT) for internet access, crucial for time synchronization and software updates.
![VirtualBox network settings for Attack Machine]("\Network-config\Kali-Network-Config.png") 
![VirtualBox network settings for Vulnerable Machine]("\Network-config\Metasploitable-Network-Config.png") 
![VirtualBox internal network settings for IDS Machine]("\Network-config\IDS-Network-Config-1.png") 
![VirtualBox NAT network settings for IDS Machine]("\Network-config\IDS-Network-Config-2.png") 
## Implimentation and Challenges Faced
Building this homelab was a hands-on learning experience that involved navigating several common and complex system administration and network security challenges. Each obstacle provided an opportunity to deepen my understanding and refine my troubleshooting methodology.
### 1. Initial Network Connectivity and Static IP Configuration
- **Challenge:** Initially, the virtual machines were unable to communicate reliably due to incorrect network settings (e.g., DHCP not providing consistent IPs, or VMs not being on the same logical network segment). This blocked all subsequent steps, including software installation and inter-VM communication.
- **Solution:** I troubleshot and resolved network connectivity issues by ensuring all three VMs were configured on the same Internal Network (My IDS required internet for syncing its date/time and dowloading the IDS software, so it has a second adapter connected to a NAT Network). I then implemented static IP addressing on each VM using Netplan (/etc/netplan/*.yaml) for predictable and stable network communication. This foundational step was critical for the lab's functionality.
![Attack Machine IP config]("\IP-config-screenshots\Kali-IP.png") 
![Vulnerable Machine IP config]("\IP-config-screenshots\Metasploitable-IP.png") 
![IDS Machine IP config]("\IP-config-screenshots\IDS-IP.png")
### 2. Suricata IDS Core Configuration and Startup Issues
- **Challenge:** After installing Suricata, the service failed to start, returning a critical error: E: runmodes: no output module named alert-fast. This indicated a fundamental problem within the IDS's primary configuration file.
- **Solution:** I diagnosed and mitigated this service failure by meticulously reviewing the suricata.yaml file. I identified that an incorrectly named and conflicting output module (alert-fast) was causing the parsing error. I remediated this by manually editing the suricata.yaml to remove the invalid entry, ensuring Suricata could initialize correctly.
![Corrected outputs module in suricata.yaml]("\IDS-file-configuration\IDS-suricata.yaml-outputs.png") 
### 3. Persistent Permissions and Temporary File Management
- **Challenge:** Even after addressing configuration syntax, sudo systemctl status suricata reported "Permission denied" errors related to creating a socket directory (/var/run/suricata/) and an inability to write to it. These temporary directories are reset on reboot, making manual fixes non-persistent.
- **Solution:** I implemented a robust, persistent solution by configuring a tmpfiles.d entry (/etc/tmpfiles.d/suricata.conf). This configuration automatically creates the /var/run/suricata directory with the correct suricata:suricata ownership and 0750 permissions at every system boot, resolving the permission denied errors permanently and ensuring Suricata's reliable startup.
![Corrected permissions in /var/log/suricata]("IDS-chown-config") 
### 4. IDS Traffic Visiility (Promiscuous Mode)
- **Challenge:** Despite Suricata running without startup errors, the fast.log file remained empty even after running nmap scans from Kali to Metasploitable2. This suggested the IDS was not "seeing" the network traffic intended for other machines.
- **Solution:** I identified the critical omission of promiscuous mode. By default, VMs only see traffic addressed to them. I configured the Ubuntu-IDS VM's network adapter in VirtualBox to "Allow All" promiscuous mode. This enabled the IDS to monitor all traffic passing through the Internal Network, ensuring it could effectively detect activity between Kali and Metasploitable2.
![VirtualBox internal network settings for IDS Machine]("\Network-config\IDS-Network-Config-1.png") 
### 5. Custom Rule Creation and Logging Validation
- **Challenge:** After enabling promiscuous mode, the fast.log was still empty. Further investigation revealed that the default Suricata rules might not log simple nmap scans as alerts, and my initial attempts at custom rules resulted in "duplicate signature" and "error parsing signature" errors due to incorrect syntax and conflicting Signature IDs (SIDs).
- **Soltuion:** This led me to research and develop a custom Suricata rule for my local.rules file. Through iterative testing and understanding proper rule syntax (eg., alert ip any any -> any any (msg:"IP Test Rule"; sid:123; rev:1;), specifying unique SIDs within the custom range), I successfully created a rule that generated verifiable entries in the fast.log. This demonstrates my ability to author and deploy custom detection logic.
![Suricata test.rules file]("\Network-config\IDS-Network-Config-2.png")
### 6. Time Synchronization for Accurate Logs
- **Challenge:** Observing log entries, I noticed that timestamps in fast.log were often incorrect or lagged behind actual events, making correlation with nmap scan times difficult. This indicated a VM clock drift issue.
- **Solution:** I diagnosed the issue by using the timedatectl command to deduce the problem; I noticed the ntp service was active, but the system clock was not synchronized. I corrected the time synchronization issue by configuring the Ubuntu-IDS VM with a second network adapter (NAT) to provide internet access. I then enabled and configured systemd-timesyncd to use reliable NTP servers (time.cloudflare.com, time.google.com), ensuring the System clock synchronized: yes status. This resulted in accurate timestamps, vital for forensic analysis and event correlation.
![VirtualBox NAT network settings for IDS Machine]("\Network-config\IDS-Network-Config-2.png") 
## Conclusion 
This project successfully established a functional and monitored virtual homelab environment. I gained invaluable hands-on experience in:
- Virtual machine networking and configuration.
- Linux system administration (Netplan, systemd, permissions).
- Deploying, configuring, and troubleshooting a critical open-source IDS (Suricata).
- Writing and validating custom IDS rules.
- Methodical problem-solving and debugging in a complex technical environment.
This foundational lab now serves as a platform for further exploration into advanced threat detection, log analysis, and vulnerability management.
## Demonstration
Here's a quick demonstration of the IDS in action, showing an nmap scan from Kali Linux being detected and logged by Suricata on the Ubuntu-IDS VM.
![Kali Machine executing an nmap scan to Metasploitable]("\proof-of-completion\Kali-nmap.png")
![IDS Machine showing sudo tail /var/log/suricata/fast.log with the generated alerts from the nmap scan]("\proof-of-completion\IDS-nmap-detection.png")

