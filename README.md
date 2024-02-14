<h1> Security Operation Center(SOC) Automamtion</h1>
<h2>Description</h2>
  In this home lab, I am going to build and configure a Wazuh security information and event management(SIEM) and a security orchestration, automation, and response(SOAR). I will set up a Wazuh server and integrate a SOAR to help automate tasks within the environment to make identification, containment, and eradication quicker. I will create a rule to identify the use of Mimikatz, a prominent credential dumper, within the SOC environment.
<br />
<h2>Utilities Used</h2>

- <b>Wazuh-SIEM and extended detection and response(XDR)</b>
- <b>TheHive- Case Management System</b>
- <b>Shuffle-SOAR</b>
- <b>VirusTotal</b>
- <b>Powershell</b>
- <b>Windows 10 Virtual Machine using Sysmon to generate logs</b>
- <b>Ubuntu Server- to host Wazuh and TheHive


<h2>Program walk-through:</h2>


Step 1: Draw a logical diagram to depict the lab environment <br/>
<p align="center">
<img src="https://i.imgur.com/8frDNQ2.png" height="80%" width="80%" alt="Diagram"/>
</p><br />
<align="left">I used Draw.io to create this logical depiction of the lab environment I am going to create.
<br />
<br />
<h3>Part 2: Install Applications and Virtual Machines</h3>
Step 1: Create a virtual machine and enable Sysmon logging 
<br/>
<p align="center">
<img src="https://i.imgur.com/T4teFLr.png" height="80%" width="80%" alt="Sysmon logging"/>
</p><br />
I used VirtualBox to create a VM running Windows 10. After creating the VM I download Sysmon from GitHub. I used Sysmon-modular to configure my Sysmon logs. I then used PowerShell to install Sysmon.
<br />
<br />
Step 2: Create a VM in Azure to host my Wazuh SIEM.
<br /> 
<p align="center">
<img src="https://i.imgur.com/IPf1t81.png" height="80%" width="80%" alt="Wazuh Server"/>
</p>
<br />
Step 3: Install Wazuh <br/>
<p align="center">
<img src="https://i.imgur.com/NCrq1Ns.png" height="80%" width="80%" alt="Install Server"/>
</p>
<br />
Step 4: Create a VM in Azure to host TheHive<br/>
<p align="center">
<img src="https://i.imgur.com/PBpxsVa.png" height="80%" width="80%" alt="TheHive Server"/>
</p><br />
<br /> 
Step 5: Install TheHive and dependencies <br/>
<p align="center">
<img src="https://i.imgur.com/1BG71U6.png" height="80%" width="80%" alt="Dependencies"/>
</p>
<br />
<align="left">TheHive requires Java, Elasticsearch, and Cassandra to run.
</p>
<br />
Step 6: Add an agent to Wazuh<br />
<p align="center">
<img src="https://i.imgur.com/Z7s3173.png" height="80%" width="80%" alt="Agent add"/>
</p>
<br /><br />
Step 7: Configure TheHive<br />
<p align="center">
<img src="https://i.imgur.com/DT1NbRi.png" height="80%" width="80%" alt="Cassandra.yaml"/>
</p>
<br />
<align="left">I started by using nano to configure Cassandra, by editing the file located at /etc/cassandra/cassandra.yaml. I set the listen address, the rpc address, and the seed address to reflect the public IP of TheHive. Then restart the service after the configuration.
<br />
<br /> 
<p align="center">
<img src="https://i.imgur.com/gKyFbAF.png" height="80%" width="80%" alt="Elasticsearch"/>
</p>
<br />
<align="left">I started by using nano to configure Elasticsearch, by editing the file located at /etc/elasticsearch/elasticsearch.yml. I set the cluster name, removed the comment to enable 'node.name', set the network host address to the public IP address of TheHive, and enabled 'cluster.initial_master_node'. Then restarted elasticsearch
<br />
<br />
<p align="center">
<img src="https://i.imgur.com/G7i7W5b.png" height="80%" width="80%" alt="Elasticsearch"/>
</p>
<br />
<align="left">I used nano to configure TheHive, by editing the file located at /etc/thehive/application.conf. I set the hostnames' IP address, the cluster-name, and  the application.baseUrl. Then restarted TheHive
<br />
<br />
Step 8: Configure Wazuh
<br/>
<p align="center">
<img src="https://i.imgur.com/Zt5xbSE.png" height="80%" width="80%" alt="Config Wazuh"/>
</p><br />
I edited the ossec.conf file to ingest Sysmon logs. I then ran Mimkatz on the VM to generate security event logs
<br/>
<br />
<p align="center">
<img src="https://i.imgur.com/1LyLUHR.png" height="80%" width="80%" alt="ossec file"/>
</p><br />
I edited the /var/ossec/etc/ossec.conf file on the server to archive and display .json logs. Then restarted Wazuh.
<br/>
<br />
<p align="center">
<img src="https://i.imgur.com/vTUd5D7.png" height="80%" width="80%" alt="filebeat"/>
</p><br />
I edited the /etc/filebeat/filebeat.yml file on the server to archive logs. Then restarted the filebeats service.
<br/>
<br />
<p align="center">
<img src="https://i.imgur.com/fMDuLPK.png" height="80%" width="80%" alt="archiving"/>
</p><br />
I added an index in Wazuh to display all archived logs even if an alert was not generated.
</p>
<br />
<br />
Step 9: Create a custom rule in Wazuh to generate an alert on Mimikatz usage<br/>
<p align="center">
<img src="https://i.imgur.com/kb5Wy58.png" height="80%" width="80%" alt="Rule"/>
</p><br /> I set the alert to generate when the program's original filename was run, so even if an attacker renamed Mimikatz an alert will still be generated.
<br />
<br />
<h3>Part 3:Connect Shuffle(our SOAR)</h3>
Step 1: Connect Wazuh Alerts
<br /> 
<p align="center">
<img src="https://i.imgur.com/1LyLUHR.png" height="80%" width="80%" alt="Webhook"/>
</p>
<br /> First I connect a webhook to my Wazuh named Wazuh-alerts by editing my Wazuh-manager's ossec file.
<br /> 
<br />
Step 2: Extract File Hash from Alert 
<p align="center">
<img src="https://i.imgur.com/m7yA65E.png" height="80%" width="80%" alt="Regex"/>
</p>
<br /> I connected the Webhook to a Shuffle tool to parse the Wazuh alert for the SHA256 hash of the alert.
<br /> 
<br />
Step 3: Check Reputation Score w/ VirusTotal 
<p align="center">
<img src="https://i.imgur.com/0FLswQz.png" height="80%" width="80%" alt="VT App"/>
</p>
<br /> I connected the Suffle tool to a VirusTotal app to check the reputation score of the parsed SHA256 from the alert
<br /> 
<br />
Step 4: Send Details to TheHive to Create an Alert
<p align="center">
<img src="https://i.imgur.com/CMlfKXN.png" height="80%" width="80%" alt="TheHive App"/>
</p>
<br /> I then connected TheHive to my VirusTotal app inside of Shuffle so I could have cases created inside of TheHive once Wazuh generates an alert.
<br /> 
<br />
Step 5: Send an E-mail to the SOC Analyst to Begin the Investigation 
<p align="center">
<img src="https://i.imgur.com/So95PGA.png" height="80%" width="80%" alt="e-mail app"/>
</p>
<br /> I then connected an E-mail app so Shuffle will email me/ the analyst every time an alert is generated to take action.
<br /> 
<br />
<p align="center">
<img src="https://i.imgur.com/vtKH3Kl.png" height="80%" width="80%" alt="Done"/>
</p>
<br /> The last thing I do is connect the Wazuh manager to workflow to have Wazuh actively respond and block the source IP when a Mimikatz alert is generated. Now we have fully functioning SIEM and SOAR!
