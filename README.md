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

<b>We've successfully created two VMs (Virtual Machines), one running Windows Server, to act as a Domain Controller. The other VM as a client, running Windows 10. Next I will deploy AD, run a script that will create users in the domain, which I can log into from the client VM, then manage the accounts and update the group policies, all to simulate a real life environment!  </b>
<br />

To start, in the DC (Domain Controller), bring up the Server Manager dashboard, if it's not up already: <br/>
<img src="https://i.imgur.com/Hu1SI4S.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Here, click on "Add roles and features" > Next > Next > For server selection there should only be one to select: <br/>
<img src="https://i.imgur.com/2aEzeqm.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, check "Active Directory Domain Services" > Click "Add Features":  <br/>
<img src="https://i.imgur.com/pTu3pNB.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Select "Active Directory Domain Services" > Next > Next > Next > Now select "Restart the destination server automatically if required" and click "Install": <br/>
<img src="https://i.imgur.com/7QIfWrO.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Next, I have to promote this machine as a DC by configuring it as a new forest as "mydomain.com" (this can be anything, mydomain.com is just easy to remember) So, to do this, I will click on the flag icon with the caution symbol, near the top-right of the Server Manager Dashboard and select "Promote this server to a domain controller":  <br/>
<img src="https://i.imgur.com/tWUuLF7.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Then select "Add new forest" and in "Root domain name" I'll type "mydomain.com". Then "Next":  <br/>
<img src="https://i.imgur.com/DIPlTd0.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Type in a password of your choosing on the next screen like so then hit "Next":  <br/>
<img src="https://i.imgur.com/0mbmFsN.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Leave "Create DNS delegation" unchecked and click "Next". Hit "Next" until you reach this screen. On this screen, click "Install". The machine will restart after the installation:  <br/>
<img src="https://i.imgur.com/WcmGn1w.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
It may take some time to restart, but once it does, sign back in using "mydomain.com\labuser" (if you're using the same names as me) as the username. Use the password you set for "labuser". Here we have to sign in as "mydomain.com\labuser" instead of just "labuser", because now we've promoted this machine to a DC and it needs to know the context of who we're siging in as (someone in the domain, or a local user). I want to sign in as a user on the domain, so I specify that with the leading "mydomain.com\" part at sign in:  <br/>
<img src="https://i.imgur.com/R5UwW3a.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, we'll create an admin user. To do this, in the start search bar, search for "Active Directory Users and Computers":  <br/>
<img src="https://i.imgur.com/ArS11GE.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Right click on "mydomain.com" and select "New" > "Organizational Unit" and name it EXACTLY "_EMPLOYEES" (In a future project we will run a script that uses this name to work). Do the same steps here to create an OU (Organizational Unit) named "_ADMINS":  <br/>
<img src="https://i.imgur.com/LKndRfZ.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Next, create a new user in "_ADMINS" by right-clicking on "_ADMINS" > New > User and fill it out like so:  <br/>
<img src="https://i.imgur.com/otH0WZ9.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Then, create a password and uncheck "User must change password at next logon" and check "Password never expires" (You probably wouldn't do this in real life, but we'll do it for this lab where nothing's really at stake):  <br/>
<img src="https://i.imgur.com/Bl1TDNe.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Jane Doe is now apart of the "_ADMINS" OU, but isn't actually an admin. To make her an admin right-click on her name > Properties > Member Of > Add... > Enter "domain admins" > Check Names > OK > Apply > OK:  <br/>
<img src="https://i.imgur.com/JNzV00w.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, we can log out of DC and log back in using Jane's credentials:  <br/>
<img src="https://i.imgur.com/hTRt4PJ.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Once logged in, log into client-1 VM, if not already. Here I'll join this client to the domain by right-clicking the start menu > Systems > Rename this PC (advanced) > Change > Select "Domain" and enter "mydomain.com" and hit "OK":  <br/>
<img src="https://i.imgur.com/Pj8kswj.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
It'll ask for an account with permission to join the domain. We can use our admin's credentials for Jane here. Then a pop up saying welcome to the domain will appear and the machine will try and restart:  <br/>
<img src="https://i.imgur.com/vm5DhNE.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
After the restart, client-1 is now a member of the domain. To check, in the DC machine start seach bar, search for "Active Directory Users and Computers" > mydomain.com > Computers > client-1 should be listed:  <br/>
<img src="https://i.imgur.com/jvU4PGD.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now we can create another OU as we did before and name it "_CLIENTS". Then drag and drop client-1 from "computers" to "_CLIENTS":  <br/>
<img src="https://i.imgur.com/wHqvwVL.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />

<h2>Active Directory is Deployed and Ready for Use! </h2>

<b>We've successfully installed AD on the domain controller, created a domain admin, and joined the client VM to the domain. Next, we'll create users in Powershell by running a script, and then manage the accounts and group policy!  </b>

To start, log into client-1 as the domain admin (jane_admin), using Remote Desktop Connection. I want to allow Remote Desktop for non-administrative users. So, right-click on the start button > System > on the right-side click "Remote Desktop" > "Select users that can remotely access this PC": <br/>
<img src="https://i.imgur.com/XLBZ0K7.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Here, enter "Domain Users" > Check Names > OK > OK. Now all domain users can access this client via Remote Desktop:  <br/>
<img src="https://i.imgur.com/UYtCJAV.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Next, we'll create a bunch of users in Powershell. To start, log into the DC (Domain Controller) as our admin (jane_admin)
Once logged in, open Powershell ISE as an admin and start a new script:  <br/>
<img src="https://i.imgur.com/yDauCac.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Use Ctrl + S and save this like so:  <br/>
<img src="https://i.imgur.com/VgY2B14.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, we can copy and paste the user creation script in the top section and click the "Run" button which has a green play symbol located in the top bar:  <br/>
<img src="https://i.imgur.com/2XRBB2C.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Powershell will begin to run the script and create random users and place them in the OU (Organizational Unit), we created in the previous section of this project, named "_EMPLOYEES". We can check by searching "Active Directory Users and Computers" > mydomain.com > _EMPLOYEES. Here we will pick a user and log into client-1 using this username and the password the script sets for all of these users which is "Password1":  <br/>
<img src="https://i.imgur.com/X9fbOZb.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, log out of Jane's account on client-1:  <br/>
<img src="https://i.imgur.com/TU8czsa.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
We can log back into client-1 using our newly created user, making sure to specify the context of which we want to log in by adding "mydomain.com\" before the username:  <br/>

![Screenshot 2024-10-17 163822](https://github.com/user-attachments/assets/23fd7602-f727-4877-9083-c324eee07664)

<br />
<br />
When we're logged in we can see a user folder for our user if we click on file explorer > C: > Users. We can also see the folder for the other users that have signed into this client. However, we cannot access them, because we don't have those permissions as a user:  <br/>
<img src="https://i.imgur.com/Z6N0iQB.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, let's log out of this user, because we are going to set an account lockout group policy and attempt to lock this user out:  <br/>
<img src="https://i.imgur.com/EFSeORv.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Go back to our DC and right click the start button > Run > type "gpmc.msc" to bring up the Group Policy Managment Console:  <br/>
<img src="https://i.imgur.com/5B60ncN.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Expand Domains > mydomain.com > select Default Domain Policy:  <br/>
<img src="https://i.imgur.com/sMRUMIa.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Under Computer Configuration expand Policies > Window Settings > Security Settings > select Account Lockout Policy:  <br/>
<img src="https://i.imgur.com/7vkAH1G.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Here we'll select "Account lockout threshhold" and choose to make the account lockout after 5 invalid attempts. Make sure to hit Apply:  <br/>
<img src="https://i.imgur.com/7xP0ADr.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
You can double-check this by going to the group policy settings and scrolling down to view the lockout policy:  <br/>
<img src="https://i.imgur.com/ZaI76lr.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
The policy will update automatically, but it will take some time. We can force the update on client-1 by signing into it as our admin and running "gpupdate /force" in the command line:  <br/>

![Screenshot 2024-10-17 164029](https://github.com/user-attachments/assets/759049c6-3b65-4448-9336-8eb12f3f1970)

<br />
<br />
<img src="https://i.imgur.com/UeDCEv8.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, we can sign out of client-1 and attempt to sign back into it with our user with an invalid password 5 times. A lockout message will appear:  <br/>

![Screenshot 2024-10-17 163214](https://github.com/user-attachments/assets/452e8fc7-3085-47f7-b44b-3b88818158e3)

<br />
<br />

![Screenshot 2024-10-17 163016](https://github.com/user-attachments/assets/6e9a4ce7-9996-4a59-a40e-960aca1b33d9)

<br />
<br />

To unlock the users account, head back to the DC > Active Directory Users and Computers > mydomain.com > _EMPLOYEES > double-click on the locked out user > Account tab > check the "Unlock account" box:  <br/>
<img src="https://i.imgur.com/UUT5jw2.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
(Alternatively you could right-click on the user > Reset Password and there will be a "Unlock users account" option. That way you can change the password at the same time you unlock it):  <br/>
<img src="https://i.imgur.com/4UOG3gY.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now our user can sign into clinet-1 as normal:  <br/>

![Screenshot 2024-10-17 164322](https://github.com/user-attachments/assets/d19891da-57d5-42ef-b7fc-d228740e1154)

<br />
<br />
To disable users, go to the DC and in Active Directory Users and Computers > mydomain.com > _EMPLOYEES > right-click on user you want to disable > Disable:  <br/>
<img src="https://i.imgur.com/HqbkjOY.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
If we sign out of client-1 now, and try to sign back in using the disabled account, this message will appear:  <br/>

![Screenshot 2024-10-17 164515](https://github.com/user-attachments/assets/8349cb85-a767-4151-89f4-638392424ec2)

<br />
<br />
To enable the user again go back to the DC and right-click again on the user and select Enable:  <br/>
<img src="https://i.imgur.com/P4aL7iw.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
The user can now log into client-1, so we'll do that and once we are logged in we'll view some logs through the event viewer. To do so, search for "Event Viewer" in the start search bar > Windows Logs > Security. You'll notice we cannot view these:  <br/>
<img src="https://i.imgur.com/oQveEHJ.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
This is because we are not an admin. Not to worry, we don't need to sign out of this whole client and sign in as Jane (our admin). We already know her credentials. We just need to close out of this window and open it again, but this time, as an administrator. It will have a pop-up asking for admin credentials:  <br/>
<img src="https://i.imgur.com/u96rZar.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Now, if we navigate back to the security log page, we can view them:  <br/>
<img src="https://i.imgur.com/25pXQAI.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
Here we can see a bunch of different information like successful and failed log on attempts, who attempted them and from what IP address, along with the date and time they happened:  <br/>
<img src="https://i.imgur.com/weTncbY.png" height="80%" width="80%" alt="Setting Up in Azure"/>
<br />
<br />
<h2>Active Directory: Creating Users, Group Policy, and Managing Accounts in Azure is Now Complete </h2>

<b>We've successfully configured Remote Desktop for non-administrative users, automated user creation with PowerShell, and managed group policies. Additionally, we covered account lockouts and log monitoring to simulate a real-life IT environment!  </b>
<br />
<br />
</p>



<br />
</p>
