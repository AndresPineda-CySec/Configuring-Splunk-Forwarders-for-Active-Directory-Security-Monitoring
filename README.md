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
These are the VMs I will be using for this lab. The Splunk machine is running on Ubuntu, and the machine named ADDC01 will be the Active Directory server running on Windows Server 2022.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20155242.png?raw=true" height="40%" width="40%"/> <br />
<br />
<br />
I first need to configure my virtual machines to align with the logical network topology settings. <br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Topology.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
I will begin by creating the network environment for this project using a NAT Network, which allows all of the virtual machines to communicate with each other while still providing internet access through the host machine.<br />
For the network address, I selected 192.168.10.0/24. The /24 subnet mask corresponds to 255.255.255.0, which provides up to 254 usable host addresses within the network. This subnet size is commonly used in small to medium-sized environments because it is simple to manage while still providing more than enough IP addresses for this lab setup.<br />
The usable IP address range for this network will be 192.168.10.1 – 192.168.10.254 and the name of the network will be “Active Directory Lab."<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-08%20160358.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
<br />
<br />
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
