# SOC-Automation-Lab

## Objective
[Brief Objective - Remove this afterwards]

The Detection Lab project aimed to establish a controlled environment for simulating and detecting cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system, generating test telemetry to mimic real-world attack scenarios. This hands-on experience was designed to deepen understanding of network security, attack patterns, and defensive strategies.

### Skills Learned
[Bullet Points - Remove this afterwards]

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used
[Bullet Points - Remove this afterwards]

- Security Information and Event Management (SIEM) system for log ingestion and analysis.
- Network analysis tools (such as Wireshark) for capturing and examining network traffic.
- Telemetry generation tools to create realistic network traffic and attack scenarios.

## Steps
drag & drop screenshots here or use imgur and reference them using imgsrc

Every screenshot should have some text explaining what the screenshot is about.

<div>
  <img src="https://github.com/beersb/SOC-Automation-Lab/blob/main/SOC%20Automation%20Network%20Diagram.drawio.png" alt="SOC Automation Network Diagram" width="800" height="800">
</div>

*Ref 1: Network Diagram*

### Deploying the Windows 10 Client
*Insert Description here*

### Deploying the Wazuh Server
<div>
Now, we move onto the Wazuh server. Q? What is Wazuh?
In this case, our Wazuh server will be located in the cloud, hosted by Vultr, instead of a VM on our local machine. So, to get started we first login to our Vultr account through their web portal. We then deploy a new server, named Auto-SOC-Wazuh, using Ubuntu 22.04 LTS for the operating system. In terms of system resources, this system will be somewhat beefy, although it is by no means a monster, weighing in at 8GB of RAM and ***GB of storage. Once the server is ready to go, we'll create a new Firewall Group on Vultr, allowing all TCP traffic but only for my local IP. Thus, we will be saved from constant attempts to break into the server by bots. 

As soon as the serve finishes booting up, we ssh into its root account via a PowerShell session on the SOC Analyst Laptop, inputting credentials as necessary. In this case, we will install Wazuh using an automated script, which can be found in the Wazuh documentation and may be executed with the following:
  
    curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && bash wazuh-install.sh -a

It would no doubt be beneficial to practice installing the application by hand. This however, must await a future date when the lab will be redone using only manual installation techniques. as the script ran, it was provided a message that ports 1514, 1515, and 443 must all be open. Accordingly, we allowed these ports through the firewall on our machine via ufw:

    ufw allow 1515, 1515, 443
    
after the script was finished runnig. The script was able to finish running successfully, and we are presented with a message displaying the default credentials to be used to login into the web console:

</dive>

