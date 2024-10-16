# SOC-Automation-Lab

## Objective

The Detection Lab project aimed to establish a controlled environment for simulating and detecting cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system, generating test telemetry to mimic real-world attack scenarios. This hands-on experience was designed to deepen understanding of network security, attack patterns, and defensive strategies. I would like to especially thank the MyDFIR YouTube channel (Insert Hyperlink) for providing a tutorial on setting up and configuring this project. 

### Skills Learned

- Advanced understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting network logs.
- Ability to generate and recognize attack signatures and patterns.
- Enhanced knowledge of network protocols and security vulnerabilities.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- Security Information and Event Management (SIEM) system for log ingestion and analysis.
- Network analysis tools (such as Wireshark) for capturing and examining network traffic.
- Telemetry generation tools to create realistic network traffic and attack scenarios.

## Steps

As breifly explained above, the overall goal of this project is to set-up a basic SOC environment using Wazuh which will be designed to monitor a single Windows host using an appropriate agent. Once this point has been reach, the project will be expanded using TheHive and Shuffle in order to implement a Security Orchestration, Automation, and Response (SOAR) environment. We want this environment to successfully detect a credential dump on the Windows machine using Mimikatz and then alert our simulated SOC analyst via email. Accordingly, the lab will consist of a SOC analyst machine (my laptop), a Windows 10 host, two Ubuntu 22.04 LTS servers, one of which will host TheHive, the other Wazuh. The Windows machine and both Ubuntu machines will be hosted in the cloud through Vultr. A network diagram illustrating this setup is provided immediately below.

<div>
  <img src="./SOC%20Automation%20Lab%20Network%20XML.drawio.png" alt="SOC Automation Network Diagram" width="800" height="800">
</div>

*Ref 1: Network Diagram*

### Deploying the Windows 10 Client and Installing Sysmon
The first step in the lab will be to deploy the Windows 10 client which will act as the victim in our project scenario. To do this, we navigate to our Vultr panel and deploy a new instance, specifying Windows 10 as the operating system and allocating an appropriate amount of RAM, storage, and CPU cores. We also create a Firewall policy that blocks all traffic from all IP addresses except for the SOC analyst laptop, which is given TCP access on all ports, and then add the new Windows host to this policy. This is a critical step, as otherwise our host will be exposed to the entire internet, and will be the victim of a shockingly large number of automated brute force attacks (see the 30 Day Challenge project for an account of my first encounter with this phenomenon). 

Once we have the host up and running, we will use the Windows Remote Desktop Protocol (RDP) to connect to it via our SOC Analyst laptop in order to install Sysmon. Sysmon is a Windows service used to collect detailed information on process creation, network connections, and even file modification. 

<div>
  <img src="lab_images/Firewall Group.png" alt="SOC Automation Network Diagram" width="1100" height="150">
  
  *Ref. 2: A screenshot of the single rule added to the Auto-SOC-Default Firewall Group, the IP address of the SOC Analyst laptop has been redacted for the privacy of the author*
</div>

We will be using Sysmon to detect the actions of a simulated hack further on in the lab, and it will accordingly be Sysmon data that will be ingested by our SIEM.

To install the applicaiton, we navigate to the official download page on Microsoft's website:

    https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

and follow the usual procedure for downloading something from the internet. Once the download is finished, a folder called Sysmon.zip will be present in the downloads folder. This folder must be extracted, it doesn't really matter where, but in this case we will just use the Downloads folder. Next, we will need to grab the configuration file we will be using from the Github page at which it is hosted:

    https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml

saving the file in the extracted Sysmon folder we just created. 

<div>
  <img src="lab_images/Sysmonconfig.png" alt="SOC Automation Network Diagram" width="1100" height="200">
  
  *Ref. 3: The beginning of the Sysmon configuration file utilized in this project*
</div>

Once this is accomplished, we need to open a PowerShell session as an administrator and run the installation executable. To do so, we navigate to the appropriate directory,

    cd C:\Users\Administrator\Downloads\Sysmon

and then run the executable, using the -i option to specify the configuration file:

    .\Sysmon64 -i sysmonconfig.xml

Now, we check the Windows services list to ensure that Sysmon has indeed been installed and is present on the machine. To do this, one navigates to the Services app (which can be found by simply searching for 'services' in the Windows start menu), then scrolling to the S section of the window (the services are ordered alphabetically) and searching for Sysmon64 on the list. In this case, we are successful and will move onto the next step, creating the Wazuh server.

<div>
  <img src="lab_images/Sysmon install success.png" alt="SOC Automation Network Diagram" width="1100" height="600">
  
  *Ref. 3: A snapshot demonstrating the location of the Sysmon64 service in the Windows services list*
</div>

### Deploying the Wazuh Server
Now, we move onto the Wazuh server. Q? What is Wazuh?
In this case, our Wazuh server will be located in the cloud, hosted by Vultr, instead of a VM on our local machine. So, to get started we first login to our Vultr account through their web portal. We then deploy a new server, named Auto-SOC-Wazuh, using Ubuntu 22.04 LTS for the operating system. In terms of system resources, this system will be somewhat beefy, although it is by no means a monster, weighing in at 8GB of RAM and ***GB of storage. Once the server is ready to go, we'll create a new Firewall Group on Vultr, allowing all TCP traffic but only for my local IP. Thus, we will be saved from constant attempts to break into the server by bots. We will add not only the Wazuh server to this group, but TheHive host as well.

<div>
  <img src="lab_images/Firewall Group.png" alt="SOC Automation Network Diagram" width="1100" height="150">
  
  *Ref. 4: A screenshot of the single rule added to the Auto-SOC-Default Firewall Group*
</div>

As soon as the serve finishes booting up, we ssh into its root account via a PowerShell session on the SOC Analyst Laptop, inputting credentials as necessary. In this case, we will install Wazuh using an automated script, which can be found in the Wazuh documentation and may be executed with the following:
  
    curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && bash wazuh-install.sh -a

It would no doubt be beneficial to practice installing the application by hand. This however, must await a future date when the lab will be redone using only manual installation techniques. as the script ran, it was provided a message that ports 1514, 1515, and 443 must all be open. Accordingly, we allowed these ports through the firewall on our machine via ufw:

    ufw allow 1515, 1515, 443
    
after the script was finished runnig. The script was able to finish running successfully, and we are presented with a message displaying the default credentials to be used to login into the web console:

<div>
  <img src="lab_images/Wazuh Finished + Password.png" alt="SOC Automation Network Diagram" width="900" height="75">

  *Ref. 5: A screenshot of the Wazuh credentials and installation completion message. The server has been dismantled so these credentials no longer correspond to a live instance*
</div>

### Spinning Up and Deploying TheHive
This process will be pretty similar to getting the Wazuh server going. However, TheHive is a very resource intensive application, requiring 16GB of RAM and 4 CPU cores to run at a bare minimum. The cloud instance chosen with thus have far stronger specifications than that used for Wazuh, though the process to create it is almost identical. TheHive server will also be added to the Auto-SOC-Default group to prevent constant brute forcing. 

As with Wazuh, an automated script will be used for install. The install process for TheHive is a little more involved than that of Wazuh, and thus a manual install would be even better practice. This will definitely have to be revisited in the future. Regardless, the script may be found in TheHive documentation, and is executed like so:

    wget -q -O /tmp/install.sh https://archives.strangebee.com/scripts/install.sh ; sudo -v ; bash /tmp/install.sh

after first establishing an SSH connection with the new cloud server via PowerShell. This installation will likely take a few minutes, but once the process is complete, TheHive will have been successfully installed.

<div>
  <img src="lab_images/TheHive Install Script.png" alt="SOC Automation Network Diagram" width="400" height="300">

  *Ref. 6: A screenshot of TheHive installation script*
</div>

### Configuring Wazuh, TheHive, and Cassandra
Now that we have successfully spun up the appropriate servers and downloaded the corresponding applications, we need to configure these applications for our purposes. We will begin with Cassandra
#### Cassandra Configuration
The file of interest is located at:

    /etc/cassandra/cassandra.yaml

We'll first change the cluster name to Auto-SOC to reflect the purpose of the server. We also need to change the listen address and the rpc address fields to the public address of the TheHive server. Finally, we need to change the address of the seed provider, also to the public TheHive server. 

<div>
  <img src="lab_images/Cassandra Config Snapshot.png" alt="SOC Automation Network Diagram" width="500" height="150">

  *Ref. 7: A screenshot of the Cassandra configuration file, and one of the changes made therein*
</div>
The next step is to remove the old configuration files, which may be done via the command:

    rm -rf /var/lib/cassandra/*

Now that these old files are gone, we need to restart the Cassandra service, which may be done in the usual method via systemctl:

    systemctl start cassandra.service

We can also check the status of the service to make sure everything has worked properly:

    systemctl status cassandra.service

In our case, everything was successful, and we can move onto Elasticsearch.

<div>
  <img src="lab_images/Cassandra Further Config.png" alt="SOC Automation Network Diagram" width="700" height="150">

  *Ref. 8: A screenshot of the Cassandra status check, note the words enabled, Active, on lines blank, which indicate our success*
</div>

#### Elasticsearch Configuration
In this case, the config file is located at 

    /etc/elasticsearch/elasticsearch.yml
which we will open for editing with Vim. Once there, as in elasticsearch, we will need to change the IP address to which the service is going to bind to the public address of our server so it will be reachable over the network, and the cluster name. We thus locate the newtork host field and the cluster name fields, changing each to their appropriate value. We will also want to be sure to uncomment the line which contains the node.name field, which in our case is on line 23. Finally, we find the cluseter.initial_master field, and edit it so that the text contained within the brackets so that it only contains "node-1". This field is used to scale the service, but we will not be performing any scaling for this lab, so we only need node-1. 

As with Cassandra, we will now need to restart the service in order to use the changes we have made to the configuration file. 
<div>
  <img src="lab_images/Starting Elasticsearch.png" alt="SOC Automation Network Diagram" width="700" height="150">

  *Ref. 9: A screenshot of the Elasticsearch start and enablement sequence*
</div>

#### Configuring TheHive
This time around, the configuration file is located at 

    /etc/thehive/application.conf

after opening the file with Vim, we need to make similar changes as those applied to the pervious two files. First, we need to make sure the application binds to the server's public IP address. To accomplish this, we edit both the index.search field and the hostname field, found within the db.janusgraph section to the server's public address. We must also change the URI to the proper address, which is specified by the application.baseUri field. After making all of these changes, one may notice the following section: 

<div>
  <img src="lab_images/thehive Storage File Specification.png" alt="SOC Automation Network Diagram" width="700" height="150">

  *Ref. 10: A screenshot of the TheHive configuration file which gives instructions for changing the ownership of the /opt/thp directory*
</div>

Due to the color scheme I have configured on my PowerShell instance, the text is a little difficult to read. However, it indicates we need to change the ownership of a particular directory to a service account associated with TheHive application in order for TheHive to work properly. We do this with the following command:

    chown -R thehive:thehive /opt/thp
Once we have made these changes, we once again must restart and enable the service, which will be done with the following two commands. First,

    systemctl start thehive.service

then 

    systemctl enable thehive.service
Now, we will navigate to the web console and see if we made a mistake somewhere. 

#### Checking the Configuration
TheHive console is accessed through a web browser, so we navigate to the public address of the IP, specifying port 9000, using our browser of choice:

    http://155.138.165.187:9000

And... success! We are presented with the login screen. 

<div>
  <img src="lab_images/TheHive Console Success.png" alt="SOC Automation Network Diagram" width="700" height="300">

  *Ref. 11: A screenshot of the TheHive login page*
</div>

The default credentials are: 

    Username: admin@TheHive.local
    Password: secret

and after inputting these into their respective form fields, we are granted access to the basic TheHive panel. This is great news and means that our configuration attempts were most likely a completes success! The next step in the process is to configure Wazuh. 

### Configuring Wazuh
Configuring Wazuh will be a slightly different process than configuring TheHive. In this step, we will be installing an agent on our Windows machine. This agent will be responsible for ingesting the sysmon data generated on that machine, and sending it to the Wazuh server for processing. To begin, we need to access the Wazuh application, which, similarly to TheHive, is accessed via web browser, and so we navigate to the public IP address of the Wazuh server:

    https://66.42.82.228
    
(Wazuh defaults to port 443 so we don't need to specify the port in our address). We are thus presented with the Wazuh login screen, and after inputting the credentials outputted in the installation process, we gain access to the Wazuh console.  

<div>
  <img src="lab_images/Wazuh_Console_Success.png" alt="SOC Automation Network Diagram" width="700" height="300">

  *Ref. 12: A screenshot of the Wazuh console page upon initial login*
</div>

There will be a convenient link to add a new agent prominently displayed on the console, and we can simply click this link to begin the process of installing the Wazuh agent on our Windows machine. 

Once the link has been clicked, we must specify the operating system of the host on which the agent will be installed, the address the agent will use to communicate with the central Wazuh server (the public IP address of our Wazuh server in this case), as well as optional settings such as an agent name and a group. We will skip the group for now, but input the agent name as Auto-SOC-WIN. After filling in the necessary information, a PowerShell command will be provided that can be used to download the agent on our Windows machine. Thus, we RDP into the machine and run the command. 

<div>
  <img src="lab_images/Downloading_Wazuh_Agent.png" alt="SOC Automation Network Diagram" width="700" height="100">

  *Ref. 13: The PowerShell command which downloads the Wazuh agent onto the Windows host*
</div>

After the command finishes exuting, we start the Wazuh service on our Windows host with the command below:

    net start wazuhsvc

and then hope the agent is able to orchestrate a connection with the Wazuh server. We can verify the object of our hope by checking back into the Wazuh console to see if the agent has connected to Wazuh successfully, and in our case, the connection was indeed successful. 

<div>
  <img src="lab_images/Windows_Agent_Connected.png" alt="SOC Automation Network Diagram" width="650" height="250">

  *Ref. 14: The Wazuh console after our Windows agent has successfully connected*
</div>

Now that we have a working Wazuh agent, we create a Wazuh rule for detecting the malicious executable Mimikatz. 

### Downloading Mimikatz and Creating a Rule to Detect It
As alluded to in the conluding lines of the last session, it is now finally time to simulate some malicious activity on our Windows host and we will hopefully be able to detect this technological malfeasance on through our Wazuh SIEM. First, we'll load up Mimikatz on the Windows target, and once this is done, we'll proceed to create the rule to detect it. 

To download Mimikatz, we will first need to add an exclusion to our Windows Defender configuration. An exlusion instructs Defender to not scan a particular folder, file, file type, or process, and we will need to add such an exlusion to the folder into which we intend to download Mimikatz, otherwise Defender will remove it from our machine immediately upon install (this is usually a good thing, as Mimikatz is a very well known malicious file used to pillage the Windows LSASS system for user credentials!). To add the exclusion, one must again open the Start Menu, and search for Windows Security or just Security and select the Windows Security result. From here, we will select the 'Virus & threat protection' option, and then the 'Manage settings' link under 'Virus & threat protection settings'. Then, we scroll all the way to the bottom of the page and click 'Add or remove an exclusion' under Exlusions, then 'Yes' to allow Windows Security to make changes to the device. Finally, we select the 'Add an exclusion+' button, select Folder from the drop-down menu, and then input the file-path for Mimikatz. In my case, I used the Downloads folder, which would be highly unwise if this were a production environment. 

<div>
  <img src="lab_images/part_4/Excluding the Downloads Folder.png" alt="SOC Automation Network Diagram" width="750" height="350">

  *Ref. 17: An image showing how an exclusion is displayed in the exlusions menu after it has been successfully added*
</div>

Now that Windows Defender will not look askance at our file, we may download it.  <INSERT DOWNLOAD PROCESS FOR MIMIKATZ HERE>

Now, we move on to the Wazuh agent. First, we need to configure the Wazuh agent on our Windows host to ingest the logs generated by Sysmon, as this functionality is not present in the default configuration. The configuration file for the agent is located at the following path:

    C:\Program Files (x86)\ossec-agent\ossec.conf

It's best to copy this file as a backup just in case we make a mistake and our agent breaks. On Windows machines, this is especially straighforward, and can be accomplished by simply right-clicking on the file, selecting copy, and then pasting the copied file in the same location as the original. In our case, we renamed the file to reflect that it was a backup. Now, we open the file configuration file using Notepad. According to the Wazuh documentation (contained in the references section of this project, located below), we will need to add the following lines of code in order for our agent to being collecting logs from Sysmon:

<div>
  <img src="lab_images/part_4/Configure Wazuh Agent to get Sysmon.png" alt="SOC Automation Network Diagram" width="750" height="125">

  *Ref. 16: A snapshot of the lines added to the Wazuh agent configuration file to enable Sysmon ingestion*
</div>

Once we have added these lines to the file, we need to save the file and restart the Sysmon service so it can read in the changes we just made to its configuration. We can do this in a few different ways, the most straightforward of which is to simply execute the below command in a PowerShell session.

    Restart-Service -Name wazuh

If, however, GUIs are more your style, this can also be done via the Services applicaiton on the Windows machine. One simply needs to search for 'services' in the Start Menu, select the first option, and then find the Wazuh service within the services list and select it. There should then be a restart option to select in the righthand side of the window. 

<div>
  <img src="lab_images/part_4/Restarting Wazuh.png" alt="SOC Automation Network Diagram" width="750" height="250">

  *Ref. 17: An image portraying the Wazuh service from within the Windows Services application which portrays the Restart option*
</div>

At this stage in the process, we have a working Mimikatz executable on the Windows host, and our 


