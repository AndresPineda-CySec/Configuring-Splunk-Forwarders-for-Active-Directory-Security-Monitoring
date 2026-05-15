# Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring
<h2>Description</h2>
In this project, I configured Splunk Universal Forwarders on a Windows host and an Active Directory server to centralize Security Event logs for monitoring and analysis. I then simulated a brute-force attack against the host machine and used Splunk to identify failed authentication attempts.
<br />


<h2>Utilities Used</h2>

- <b>VirtualBox</b> 
- <b>Hydra</b>
- <b>Splunk</b>
- <b>CMD</b>
- <b>Active Directory</b>

<h2>Environments Used</h2>

- <b>Windows 10</b>
- <b>Windows Server</b>
- <b>Ubuntu</b>
- <b>Kali</b>

<h2>Skills Demonstrated</h2>

- <b>Threat Detection and Analysis</b>
- <b>Log Collection and forwarding</b>
- <b>Security Information and Event Management</b>
- <b>Active Directory Administration</b>
- <b>Threat Intelligence</b>
- <b>Windows Event Log Analysis</b>
- <b>Splunk Querying and Search Processing Language (SPL)</b>
- <b>Brute Force Attack Detection</b>
- <b>Windows Server Administration</b>
- <b>SIEM Configuration</b>


<h2>Project Walk-Through:</h2>


<h3 align="center">Setting up the environment:</h3>
<p align="center">
<br />
<br />
These are the VMs I will be using for this lab. The Splunk machine runs Ubuntu, and the machine named ADDC01 will be the Active Directory server running Windows Server 2022. Both Windows machines will be equipped with Sysmon and Splunk Universal Forwarders.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20155242.png?raw=true" height="40%" width="40%"/> <br />
<br />
<br />
I first need to configure my virtual machines to align with the logical network topology settings. <br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Topology.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
I will begin by creating the network environment for this project using a NAT Network, which allows all of the virtual machines to communicate with each other while still providing internet access through the host machine.<br />
For the network address, I selected 192.168.10.0/24. The /24 subnet mask corresponds to 255.255.255.0, which provides up to 254 usable host addresses within the network. This subnet size is commonly used in small to medium-sized environments because it is simple to manage while still providing more than enough IP addresses for this lab setup.<br />
The usable IP address range for this network will be 192.168.10.1 – 192.168.10.254, and the name of the network will be “Active Directory Lab."<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20160358.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
Next, I will add all virtual machines to my NAT Network. Right-click on a machine, select Settings, and then go to the Network tab. Here, I select "Enable Network Adapter," change the "Attached to" tab to NAT Network, and select the network I made in the previous step for the "Name" tab.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20160531.png?raw=true" height="100%" width="100%"/> <br />
<br />
<br />
The next step is to configure the machines.<br />
<br />
<br />
I will start by assigning my Ubuntu Splunk server its correct IP.<br/>
First, I list the Netplan configuration files to identify which specific Netplan file my version of Ubuntu is using. Once identified, I open the file using the Nano text editor to begin editing the network configuration.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20162003.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
I added the seven boxed lines below to the file. First, I turned off DHCP to assign the machine a static IP address of 192.168.10.10/24. The nameserver was set to 8.8.8.8 so the machine can use Google's public DNS server to resolve DNS queries. I also configured the default route to use 192.168.10.1 as the gateway. Everything outside of the box was left as is.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20162032.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
I save the file and close the editor. I then type the command "sudo netplan apply" to finalize the configuration changes to the 50-cloud-init.yaml file.<br />
<br />
<br />
Typing "ip a" into the console shows the newly configured IP address, confirming the changes were successfully applied.<br/>
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20162539.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
Moving onto my Windows 10 machine...
<br />
<br />
...I entered the network settings, right-clicked the Ethernet adapter, and selected Properties. From there, I double-clicked IPv4 and disabled DHCP. Once DHCP is disabled, I can now assign the machine a static IP address of 192.168.10.100 and a subnet mask of 255.255.255.0 (which corresponds to the /24 CIDR notation). I set the default gateway to 192.168.10.1 and configured the DNS server to use 8.8.8.8; this will only be temporary.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-10%20221322.png?raw=true" height="100%" width="100%"/> <br />
<br />
<br />
I then installed Splunk Universal Forwarder and Sysmon. To configure the forwarder to know what data to send to Splunk, I created an inputs.conf file and placed it in the local folder inside the Splunk Universal Forwarder directory.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-10%20223701.png?raw=true" height="100%" width="100%"/> <br />
<br />
<br />
I repeat the last three steps on my Windows Server VM with the only difference being the IP address: 192.168.10.7.<br />
<br />
<br />
The last machine to configure is my Kali VM. I updated it to use a static IP address, and according to my network topology, the Kali machine is set to 192.168.10.250. To change this, I right-clicked the network icon in the top-right corner of the screen and selected “Edit Connections.” From there, I opened “Wired Connection 1” and went to the IPv4 tab. I then updated the address, netmask, and gateway settings. I also set the DNS server to 8.8.8.8.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20151630.png?raw=true" height="100%" width="100%"/> <br />













  
<h3 align="center">Installing Cowrie:</h3>
<p align="center">
<br />
<br />
Before installing Cowrie, I have to make sure I install a few dependencies to ensure Cowrie can run efficiently.
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Cowrie-and-Splunk-Honeypot-Threat-Analysis/blob/main/Images/installDependency.png?raw=true" height="80%" width="80%"/> <br />
This command installs essential dependencies for setting up Cowrie. It installs "git" for version control, "python3-venv" for creating isolated Python environments, "libssl-dev" for cryptographic functions, "libffi-dev" for interfacing with C libraries, "build-essential" for compiling software, "libpython3-dev" for Python development headers, and "python3-minimal" for the minimal Python 3 installation required to run Python applications. These packages ensure that Cowrie runs securely and has all the necessary tools for building and interacting with Python code.<br />
<br />
<br />
<br />
<br />
<h3 align="center">Configuring Splunk:</h3>
<p align="center">
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Cowrie-and-Splunk-Honeypot-Threat-Analysis/blob/main/Images/splunkAddData.png?raw=true" height="100%" width="810%"/> <br />
To integrate Cowrie with Splunk, I must first create an HTTP Event Collector (HEC) in Splunk. I start by navigating to "Settings" and selecting "Add Data" to begin the setup process.<br />
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Cowrie-and-Splunk-Honeypot-Threat-Analysis/blob/main/Images/SplunkAddMonitor.png?raw=true" height="100%" width="100%"/> <br />
This takes me to the data input page, where I select the "Monitor" option to continue setting up the HTTP Event Collector.<br />
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Cowrie-and-Splunk-Honeypot-Threat-Analysis/blob/main/Images/SplunkHTTPEventCollector.png?raw=true" height="100%" width="100%"/> <br />
After selecting "Monitor," I arrive at the "Add Data" page. Here, I choose "HTTP Event Collector" from the left panel and set the new HEC name to "Cowrie." I keep the default settings for the remaining configurations and click "Next" to proceed through the setup steps.<br />
<br />
<br />
<br />
<br />
<h3 align="center">Exporting Cowrie Logs to Splunk:</h3>
<p align="center">
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Cowrie-and-Splunk-Honeypot-Threat-Analysis/blob/main/Images/Cowrie.cfg.png?raw=true" height="60%" width="60%"/> <br />
To link my honeypot to Splunk, I first access my Ubuntu VM and switch to the "cowrie" user through the terminal. From there, I navigate the directory to get to the Cowrie configuration file and use the command "nano cowrie.cfg" to access cowrie.cfg.<br /> 
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Cowrie-and-Splunk-Honeypot-Threat-Analysis/blob/main/Images/OriginalCowrieSplunkconfig.png?raw=true" height="100%" width="100%"/> <br />
Inside the config file, I located the Splunk output section, which was initially disabled and configured with the incorrect URL and token; both need to be updated for proper integration.<br />
<br />
<br />
<br />
<br />
<h3 align="center">Results:</h3>
<p align="center">
After Allowing Cowrie to run for two days straight, these are the results:
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Cowrie-and-Splunk-Honeypot-Threat-Analysis/blob/main/Images/CowrieCommandLog.png?raw=true" height="60%" width="60%"/> <br />
Any data sent to the honeypot is automatically logged. To view these logs, I run "cat cowrie.log" to display the default log file used by Cowrie.<br />
<br />
<br />
