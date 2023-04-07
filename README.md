# Azure
<h1> SIEM - Azure Sentinel Setup Tutorial</h1>
<p align="center">
<img src="https://i.imgur.com/NRfvbrz.png" alt="Azure Sentinel Logo"/>
</p>

<h1>How to Setup a Basic Home Lab and use Azure Sentinel (SIEM), connect it to a live virtual machine acting as a honey pot. We will observe live attacks from around the world. We will also use a custom PowerShell script to look up the attackers geolocation information and plot it on the Sentinel Map.</h1>

<h2>Environments and Technologies Used</h2>

- Oracle VirtualBox (Virtual Machines/Compute)
- Remote Desktop
- Microsoft Azure
- Azure Sentinel (SIEM)
- Log Analytics Workspace
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2019
- Windows 10 (21H2)

<h2>Deployment and Configuration Steps</h2>

<img src="https://i.imgur.com/wk2mBsL.jpg" alt="Azure Setup"/>
<p>
In this lab we will create a virtual machine in Azure, and create a vulnerable environment by turning the external and Windows firewall off to make it enticing to attackers. Next we are going to create a Log Repository in Azure called a Log Analytics Workspace, which will be used to ingest our logs from the virtual machine. And then we're going to setup Azure Sentinel (SIEM) to create a map that maps all the different attacks from all over the world. To help us locate locate each attack by country and coordniates, we will use a custom PowerShell script to extract the IP Address from the Windows Log and send it to a third party API. And the API will derive the information and send it back to our virtual machine, which will then be used to create a custom log with the attackers location data.
</p>
<br />
  Our first step is creating our Azure account and setting up our virtual machine for our vulnerable environment. We will name our resource group "Honeypot" and virtual machine "Honeypot-vm" respectfully.
  <img src="https://i.imgur.com/7Dtl9jQ.png" alt="Lab Plan"/>
  
  Next we will create our Network Security Group, name it "Honeypot-vm-nsg" and set custom settings to allow our computer to be very discoverable on the internet. Now we can create our virtual machine, this process will take a few minutes to deploy. Next we can create our Log Analytics Workspace, which will be keeping logs of each login attempt for our honeypot.
  
  <img src="https://i.imgur.com/s3cOZM0.png" alt="Network Security Group"/>
  
  Search "log analytics workspace" in the Azure search bar, select our resource group and name it "law-honeypot1," Review and Create. Next we're going to go to the security center to enable the ability to gather logs from the virtual machine to the log analytics workspace. From Security Center select "settings," turn Azure Defender "on" select Data Collection and toggle "all events". From the log analytcs workspace we will connect it to our virtual machine.
  Next we can setup and add Microsoft Sentinel to our new workspace, and this is the SIEM we will be using to visually track and map our attacks by country, longitude, latitude, date & time, and number of attacks. To do so search "Sentinel" and select our "law-honeypot1" resource group that we just created.
  
 <img src="https://i.imgur.com/azR6cxG.png" alt="Adding Sentinel"/>
 
 With Sentinel added to our workspace, we can now open our virtual machine environment. From Azure we can find the public IP address for our VM, from here we will copy it and open Oracle VirtualBox. With our Windows 10 VM open search "Remote Desktop Connection" to create access our Azure VM using the IP address provided.
 
 <img src="https://i.imgur.com/4qCnbuf.png" alt="Remote Desktop Connection"/>
 
 Once we are logged in to our Azure VM, the first thing we want to do is turn off the firewalls. Search "wf.msc" in the search bar and turn off firewalls for both our "domain profile" and "public profile."

<img src="https://i.imgur.com/10msmyU.png" alt="Firewall settings"/>

Next we're going to implement our custom PowerShell script which will be logging each attack on our VM, using the following link:
<a href="https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1">PowerShell Script</a>
Copy the script and and open PowerShell ISE from the VM. Click "New" > paste > save "Log_Exporter" to our desktop. Before running change the API key so that we're able to receive the geodata on our VM, a new API key can be obtained here: <a href="https://ipgeolocation.io">API Key</a>. Sign up, create an account and receive your API key which we will input into line 2 of our "Log_exporter" script.

The script essentially looks through the computers' Security Event Log for every failed login attempt, and creates a new log file containing geodata. Output logs can be found at "C:\ProgramData\".

Once you've entered your API key you are ready to run your script. Any failed logins will be present below in the purple output containing geo data.

<img src="https://i.imgur.com/6LXu1Qs.png" alt="PowerShell ISE"/>

Failed login data can be found at C:\ProgramData\failed_rdp including sample data to later train our analytics workspace what to sort data for our Sentinel Map.

<img src="https://i.imgur.com/IYSTSLC.png" alt="Failed RDP Logs"/>

The next thing we're going to do is create a custom log inside of our Log Analytics Workspace that will allow us to bring the custom log with out geo data to our LAW. From law-honeypot select custom logs > add custom log > add log file. From here we have to bring over our custom log from our virtual machine with the login data "C:\ProgamData\failed_rdp". Open "notepad" paste the data and save as a ".log". Now we can upload this to our log files. This is going to be used to train log analytics what to look for in the log files. Name our custom log name "FAILED_RDP_WITH_GEO" > Review and create custom log.

<br />
Now we can run a query on our workspace. Select "Logs" on the first script line type "FAILED_RDP_WITH_GEO" and select "Run" and all of our failed login entries will print below.

<br />
<br />
We're going to create different sorting fields for our Raw Data, which contains our geo data.
Expand one of the logs > right click > "Extract fields..."

<br />
<br />
Highlight latitude, longitude, destination host, username, sourcehost, state, country, label, and timestamp inputs and label them accordingly. This will sort each entry into it's own column in our results field.

<img src="https://i.imgur.com/du1XPbP.png" alt="Custom Fields"/>

Run "Query" again and you will see log data is now parsed into individual columns. 
<br />
Next we can set up our map in Sentinel. Go to our Azure Portal, type Sentinel > Overview. You will see a default dashboard, we want to edit this to set up our geo map. Click on "Workbook" > Add Workbook > Edit (remove the two default widgets) > Add query > paste (FAILED_RDP_WITH_GEO_CL | summarize event_count=count() by sourcehost_CF, latitude_CF, longitude_CF, country_CF, label_CF, destinationhost_CF | where destinationhost_CF != "samplehost" | where sourcehost_CF != "" )
<br />
Under Visualization > select "Map" and this will bring up "map settings"
<p>
<img src="https://i.imgur.com/FQAOcI6.png?2" alt="Map Settings"/>
  </p>
