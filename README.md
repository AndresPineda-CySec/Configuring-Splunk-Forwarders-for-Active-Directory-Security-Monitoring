# Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring
<h2>Description</h2>
In this project, I configured Splunk Universal Forwarders on a Windows host and an Active Directory server to centralize Security Event logs for monitoring and analysis. I then simulated a brute-force attack against the host machine and used Splunk to identify failed authentication attempts.
<br />


<h2>Utilities Used</h2>

- <b>VirtualBox</b> 
- <b>Hydra</b>
- <b>Splunk</b>
- <b>CMD</b>
- <b>Terminal</b>
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
Now that I have updated the inputs.conf file, I went to services to restart the forwarder. There, I adjusted the “Log On As” setting to run as a local service account and restarted the Splunk Universal Forwarder in Services.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-10%20223902.png?raw=true" height="80%" width="80%"/> <br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-10%20223951.png?raw=true" height="50%" width="50%"/> <br />
<br />
<br />
I repeat all Windows 10 steps on my Windows Server VM with the only difference being the IP address: 192.168.10.7.<br />
<br />
<br />
The last machine to configure is my Kali VM. I updated it to use a static IP address, and according to my network topology, the Kali machine is set to 192.168.10.250. To change this, I right-clicked the network icon in the top-right corner of the screen and selected “Edit Connections.” From there, I opened “Wired Connection 1” and went to the IPv4 tab. I then updated the address, netmask, and gateway settings. I also set the DNS server to 8.8.8.8.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20151630.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
<br />
<br />  
<h3 align="center">Configuring Splunk:</h3>
<p align="center">
<br />
<br />
In the Inputs.conf file, I configured the Splunk Universal Forwarder to send specific information to the SIEM. The line "Index = endpoint" means that the specified source above will send the requested information to the index named endpoint. 
<br />
<br />
Now I have to create the index in Splunk. I log in to Splunk and navigate to "settings," then "data," and finally "indexes." Once selected, I choose to create a new index named "endpoints" and set it to listen on port 9997.
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-10%20232449.png?raw=true" height="80%" width="80%"/> <br />
<h3 align="center">Results:</h3>
<br />
<br />
Now, when I travel to "Indexes," I can search for "endpoints," and once selected, I see my virtual machines there under "values."
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20112107.png?raw=true" height="80%" width="80%"/> <br />
