<h1>Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring<h1>

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
Now I have to create the index in Splunk. I log in to Splunk and navigate to "settings," then "data," and finally "indexes." Once selected, I choose to create a new index named "endpoints" and set it to listen on port 9997.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-10%20232449.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
Now, when I travel to "Indexes," I can search for "endpoints," and once selected, I see my virtual machines there under "values."<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20112107.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
<br />
<br />  
<h3 align="center">Setting Up Active Directory:</h3>
<p align="center">
<br />
<br />
The next step is to set up Active Directory on my Windows Server VM. After opening the VM, I go to “Manage” and then “Add Roles and Features” in Server Manager. This opens the “Add Roles and Features Wizard,” where I select “Role-based or feature-based installation.”<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20122548.png?raw=true" height="70%" width="70%"/> <br />
<br />
<br />
Next, I select my Windows Server machine as the destination server for the Active Directory installation.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20122618.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
Next, I leave “File and Storage Services” selected and choose “Active Directory Domain Services” in the “Select Server Roles” section. I do this because Active Directory Domain Services enables the server to function as a domain controller, which manages resources within the domain.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20122714.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
I select “Next” for the remaining options and complete the installation. After the installation finishes, I close the wizard and return to Server Manager, where a notification alert appears indicating that additional configuration is required for the newly installed Active Directory Domain Services role. To continue the setup process, I select “Promote this server to a domain controller.”<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20122949.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
Now the Active Directory Domain Services Configuration Wizard opens. In the Deployment Configuration section, I select “Add a new forest,” which indicates that this will be the first domain in a new Active Directory environment rather than joining an existing one. I then set the Root Domain Name to apcysec.local.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20123046.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
I select “Next” for the remaining configuration options and complete the wizard. With Active Directory now set up, I move on to creating test users. Back in Server Manager, I open the Tools menu in the top-right corner and select Active Directory Users and Computers. This launches the management console, where I can manage domains, users, and groups.
In the left pane, I right-click and choose Create New Group. For this lab, I create two groups, IT and HR, and add one user to each.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20124617.png?raw=true," height="80%" width="80%"/> <br />
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20124637.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
<br />
<br />
<h3 align="center">Joining Client Machines to Active Directory:</h3>
<p align="center">
<br />
<br />
After creating users on my Windows Server machine, I go to my Windows 10 VM to join the computer to the Active Directory Domain. I go to my system settings and select "Advanced System Settings." There, I select the "Change name" tab and click "Change" to change the domain name. I type apcysec.local as the domain name, but I get an error. This is an easy fix...<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20125105.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
…I go to Advanced Network Connections, open Ethernet properties, and double-click IPv4. Once there, I FINALLY replace my temporary DNS server (8.8.8.8) with my domain controller’s IP: 192.168.10.7.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20125250.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
Now, when I retry the previous step, I can sign in to my AD domain.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20125422.png?raw=true" height="60%" width="60%"/> <br />
<br />
<br />
After logging in, I receive a message confirming that I have successfully joined the domain. I restart the computer and log back in as both users I made earlier.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20125511.png?raw=true" height="50%" width="50%"/> <br />
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20125738.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
<br />
<br />
<h3 align="center">Setting Up the Attack:</h3>
<p align="center">
<br />
<br />
I'm going to use Hydra to brute-force my way into my new accounts via RDP. After logging back into the Elizabeth Go account, I'll navigate to Advanced Settings, which now requires admin credentials. Once permissions have been granted, I add the account to the RDP group so the attack can function properly. I'll repeat the same process for the Andy Pineda account.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20174730.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
The next step takes place on my Kali machine, so now I have to switch over. First, I create a directory on my desktop called ad-lab. After that, I use the ls command inside the wordlists directory to view its contents. Once I find the file I want to use for my brute-force attack, I copy it into the new directory I created on my desktop. I’m copying it because I plan on making a few modifications to the list before using it.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-11%20155257.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
Considering the wordlist file I’m using, rockyou.txt, contains over 14 million passwords, I’m going to create a smaller file called passwords using only the last 20 entries from the original list. I’m doing this to simplify the lab and drastically reduce the attack's runtime. While in the terminal, I change directory into my ad-lab folder using "cd ~/Desktop/ad-lab." Once I’m inside the folder, I extract the last 20 lines of rockyou.txt and redirect them into a new file called passwords using the command "tail -n 20 rockyou.txt > passwords." After that, I open the file in Nano to manually add the correct password for one of my test accounts. Inside Nano, I add the correct password at the bottom of the file, increasing the total count from 20 to 21 entries. After saving and exiting Nano, the updated password file is ready for the next step.
<br />
<br />
Next, I simulated a brute-force attack against the target machine using Hydra. The command I used was "hydra -l aPineda -P passwords.txt rdp://192.168.10.100 -v," which targeted the RDP service running on 192.168.10.100. The -l flag specified the username, -P pointed to a password list, and -v displayed every login attempt in real time. Once Hydra found the correct password, it reported the valid credentials and stopped the attack.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-12%20152326.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
<br />
<br />
<h3 align="center">Results:</h3>
<p align="center">
<br />
<br />
Now I log back into Splunk and open Search & Reporting. From there, I search for "index=endpoint" to filter for data generated by the index I made earlier, and I narrow the time range to the last 15 minutes to focus on the most recent activity. On the left-hand side, I select EventCode, which shows Windows Security Event IDs from the last 15 minutes. Two codes stand out the most: 4624 and 4625. Event Code 4624 represents successful logins, and while there are a relatively high number of them, the more concerning indicator is Event Code 4625, which represents failed login attempts.<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-12%20152601.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
I double-click 4625, which reveals the code’s meaning along with the host name and IP address that tried to log in. When I go through the entries, you can literally see each attempt happening seconds apart, which is a classic telltale sign of a brute-force attack. Then I double-click 4624 and look for the IP that made all those failed login attempts. Eventually, I learned that the attacker finally got in. Good thing this is just a lab :) <br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-12%20152712.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
<img src="https://github.com/AndresPineda-CySec/Configuring-Splunk-Forwarders-for-Active-Directory-Security-Monitoring/blob/main/images/Screenshot%202026-05-12%20152809.png?raw=true" height="80%" width="80%"/> <br />
<br />
<br />
That concludes the Project!


