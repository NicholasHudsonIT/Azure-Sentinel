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
- LLog Analytics Workspace
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2019
- Windows 10 (21H2)

<h2>Deployment and Configuration Steps</h2>

<p>
</p>
<p>
In this lab we will create two Virtual Machines. One will be a Domain Controller using Windows Server 2019, and the other will be the Client machine using Windows 10. We will use two NICs on our Domain Controller, one will connected to our personal home network and we will use the internal network to establish a static IP, then download Active Directory Domain Services on our DC to connect to the client machine. Client machine will then be joined to the domain. We will establish a connection through the DNS settings on the client machine, the client machine will use the DC as its DNS server. We will also be adding 1,000 active users using a Powershell script that are able to login using our Client machine.
</p>
<br />
