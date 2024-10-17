# Active-Directory-Deployment-in-the-Cloud

<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines. A software built by microsoft that essentially lets you manage thousands of user accounts in a single place. (Accounts, Passwords, Permissions) Provides a way to contol access to resources on a large scale. (Ex. A big workplace that has a bunch of employees & departments working in 1 building)  <br />


<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022
- Windows 10 (21H2)

<h2>High-Level Deployment and Configuration Steps</h2>

- Create Resources in Azure. Setup Domain Controller & Client 1 Virtual Machine 
- Attach them to the same Virtual Network & Set Client 1 DNS settings to the DC Private IP address. Then Ping DC-1 from Clientt 1 to ensure connection.
- From DC 1 Install Active Directory Domain Services 
- Create a Domain Admin user & 2 Organizational Units called Employees & Admins from Active Directory Users & Computers(ADUC)
- Join Client 1 to your domain
- Setup Remote Desktop for non-administrative Users & create a bunch of additional users using Power Shell
- Group Policies & Managing accounts
- Observing Logs

<h2>Deployment and Configuration Steps</h2>

Navigate to Microsoft Azure and create a resource group: <br/>
![Screenshot 2024-10-17 154759](https://github.com/user-attachments/assets/12005479-0f21-49c1-9c9d-f07d93681b22)
<br />
<br />
Next, create a virtual network like so: <br/>
![Screenshot 2024-10-17 154857](https://github.com/user-attachments/assets/5dd5a791-5ee4-4fff-904c-0fbfdda8f51a)
<br />
<br />
Once my resource group and network is created, I'll create and set up the virtual machine that will act as our Domain Controller. For the image, make sure you use Windows Server:  <br/>
![Screenshot 2024-10-17 155008](https://github.com/user-attachments/assets/6e12b8cf-3fc4-43be-b99c-f09db88f0b2e)
<br />
<br />
![Screenshot 2024-10-17 155124](https://github.com/user-attachments/assets/724af854-8546-42f1-9d40-9e17e23a3206)
<br />
<br />
In the Networking tab of this VM, I'll make sure it will create itself on the virtual network I just created. I'll leave all other settings default and create this VM: <br/>
![Screenshot 2024-10-17 155208](https://github.com/user-attachments/assets/2d0c5232-6992-471a-8ba6-011819ea7757)
<br />
<br />
Now, I'll create another VM that will serve as the client. The image for this machine should be Windows 10, NOT Windows Server like I did for the previous machine:  <br/>
![Screenshot 2024-10-17 155400](https://github.com/user-attachments/assets/4fc136a6-a2b9-4134-8ae0-b6837e9c8703)
<br />
<br />

<br />
<br />
In the Networking tab of this VM, I'll make sure it will create itself on the same virtual network of the previous machine created. I'll leave all other settings default and create this VM:  <br/>

![Screenshot 2024-10-17 155626](https://github.com/user-attachments/assets/2fe98df4-4b39-4228-b317-0ce77a4ba539)

<br />
<br />

I now need to set our DC (Domain Controller) private IP address to "static" as by default it is set to "dynamic". I want this to be static, because this DC will double as a DNS (Domain Name System) server, which I will tell our client to use as a DNS server later. If the IP allocation setting were set to dynamic, the IP address could change leaving the DNS configuration of our client invalid. So, I'll go to the network settings of the DC and switch the IP allocation to static:  <br/>
![Screenshot 2024-10-17 155711](https://github.com/user-attachments/assets/fbd99ac9-cfbb-4281-89d9-00d1afe4efa2)
<br />
<br />
![Screenshot 2024-10-17 155747](https://github.com/user-attachments/assets/820ee6f9-ce03-47d9-b1a9-0d8fd9855c02)
<br />
<br />
Next, I'll use Remote Desktop Connection to connect to the DC using its public IP and the log in credentials I created when setting up this machine  <br/>
Once I'm logged in, the following screen will appear with the Server Manager open. (If this isn't what you're seeing and instead it a regular windows desktop, you may have connected to the client VM instead or chose the wrong image when creating the DC):  
<br/>

![Screenshot 2024-10-17 155849](https://github.com/user-attachments/assets/8232b470-aeb3-47cb-ba27-27dc4ac57ef2)

<br />
<br />
Next, I'm going to disable the firewall (you probably wouldn't do this in real life, but for the sake of this lab where nothing is at stake, I'll go ahead and do it). So, to disable the firewall I'll right click on the "Start" button and select "Run". Then type "wf.msc":
Click on "Windows Defender Firewall Properties" then on the, "Domain Profile", "Private Profile, and the "Public Profile" tabs, turn the firewall state off:  <br/>
<br/>

<img src="https://i.imgur.com/0ztwOZI.png" height="80%" width="80%" alt="Setting Up in Azure"/>

<img src="https://i.imgur.com/5luuIqk.png" height="80%" width="80%" alt="Setting Up in Azure"/>

<br />
<br />
<img src="https://i.imgur.com/uZJpkGP.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
<img src="https://i.imgur.com/3wiNDiy.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
You should see that all the firewall settings are disable:  <br/>
<img src="https://i.imgur.com/wnHbVDn.png" height="80%" width="80%" alt="Setting Up in Azure"/>

<br />
<br />
Next, I need configure our clients DNS settings to the DC. To start, back in Azure, I'll grab the DCs private IP address:  <br/>

![Screenshot 2024-10-17 160813](https://github.com/user-attachments/assets/744f5205-7b73-4852-be10-5b94f249d37a)

<br />
<br />
Then, I'll go to the network setting of the client machine. click on the NIC (Network Interface Card), go to settings, then DNS servers and switch from "Inherit from virtual network" to "Custom". Input the DCs private IP here and save:  <br/>

![Screenshot 2024-10-17 160905](https://github.com/user-attachments/assets/a8bdd7d6-9fc9-4aa2-8ec4-9875b11dca73)

<br />
<br />
After that's saved, I'll restart the client machine:  <br/>

![Screenshot 2024-10-17 160936](https://github.com/user-attachments/assets/f145a43f-1a8b-436e-ab04-ec5427d59831)

<br />
<br />
Once the machine as restarted, I'll use Remote Desktop connection to connect to the client machine using its public IP and the log in credentials I created while setting up this machine.
Once logged in, I will open Powershell and attempt to ping the DC using the ping command and its private IP address. In my case it'll look like this. (If there is an error and the connection timed out, double check in Azure to make sure both of the machines are on the same virtual network. If they aren't this is likely causing the error and you'll need to set up the machine again on the same network):  <br/>

<img src="https://i.imgur.com/5bEWedc.png" height="80%" width="80%" alt="Setting Up in Azure"/>

<br />
<br />

While I'm here I can double check that the DNS server settings are pointing to the DC. I'll run "ipconfig /all" and look for the "DNS Servers" and it should point to our DC if everything is set up properly:  <br/>
<img src="https://i.imgur.com/dgShrBB.png" height="80%" width="80%" alt="Setting Up in Azure"/>

<br />
<br />

<h2>Active Directory Infrastructure is Now Prepared! </h2>

<b>We've successfully created two VMs (Virtual Machines), one running Windows Server, to act as a Domain Controller. The other VM as a client, running Windows 10. Don't forget: In later projects I will deploy AD, run a script that will create users in the domain, which I can log into from the client VM, then manage the accounts and update the group policies, all to simulate a real life environment!  </b>
<br />
<br />
</p>
