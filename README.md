<p align="center">
<img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo"/>
</p>

<h1>On-premises Active Directory Deployed in the Cloud (Azure)</h1>
This tutorial outlines the implementation of on-premises Active Directory within Azure Virtual Machines.<br />



<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Computers)
- Remote Desktop
- Active Directory Domain Services
- PowerShell

<h2>Operating Systems Used </h2>

- Windows Server 2022 Datacenter: Azure Edition- x64 Gen2
- Windows 10 (22H2)


<h2>Deployment and Configuration Steps</h2>

<p>
<img width="862" height="790" alt="1" src="https://github.com/user-attachments/assets/275587ef-d899-4a38-bbe5-394658f9b1f8" />

</p>
<p>
In this procedure, we will be covering how to configure Active Directory on Azure between two virtual machines. One being the Domain Controller (VM named DC-1) and another being the client (named client-1). Active Directory, hosted on a domain controller virtual machine, enables an organization to centrally manage, authenticate, and authorize users and computers, while controlling access to resources based on assigned roles and responsibilities. To get started, we will first create a virtual machine running Windows Server 2022, this will serve as our Domain Controller. Next we will create the client's virtual machine using Windows 10. Before creating the virtual machines, however, we will need to create a virtual network for both machines to communicate with eachother effectively. In Azure, search "Virtual Networks" -> Create -> We will name this virtual network as "Active-Directory-Vnet" (Regions throughout this procedure will stay the same).
</p>
<br />

<p>
<img width="886" height="1029" alt="2" src="https://github.com/user-attachments/assets/dd8b0d2a-449c-4ae2-9a8c-7cd48a99c19a" />

</p>
<p>
We will be creating both virtual machines with the machine images (operating systems) mentioned in the previous step (above example shows VM settings for DC-1), at least 2vcpus for the size, and having them on the same virtual network we had just created (Active-Directory-Vnet).
</p>
<br />

<p>

<img width="1302" height="585" alt="3" src="https://github.com/user-attachments/assets/892073a7-467f-48d2-912f-08cee6c9d06a" />

<img width="2543" height="738" alt="4" src="https://github.com/user-attachments/assets/91fda5ca-5198-4abc-aedb-a23d1feaac7c" />

</p>
<p>
Next we will set the IP address of the domain controller to static, as a dynamic IP will not be convenient for the client's virtual machines DNS server, as we would want the client's VM to look at the domain controller as the DNS server itself. To do this, click on the DC-1 Virtual machine (Search Virtual Machines in the Azure search box then click on DC-1) -> Click on Network Settings -> Click on DC-1's NIC (Network Interface Card) -> IP Configuration -> change the IP allocation from "dynamic" to "static".
</p>
<br />

<p>
<img width="1092" height="777" alt="5" src="https://github.com/user-attachments/assets/5c95ddcb-ba7f-43ec-a80b-026447ac7225" />

</p>
<p>
Now we will log onto Dc-1 (domain controller) on our Remote Desktop and configure the firewall settings to enable connectivity between the two machines. In the Dc-1 virtual machine, type "run" in the windows search box -> type "wf.msc" -> click "Windows Defender Firewall Properties" -> select "Off" by Firewall state, under Domain Profile, Private Profile, and Public Profile -> Click Apply.
</p>
<br />
<p>
<img width="969" height="589" alt="6" src="https://github.com/user-attachments/assets/3184e718-ebc9-494a-b5da-9a984dd150fb" />

</p>
<p>
Next we will configure Client-1's DNS network settings to be set to DC-1's static IP address. To do this, copy the private IP address of DC-1 -> click on Client-1's VM in Azure -> Network Settings -> Client-1's NIC -> DNS Servers -> select custom under "DNS Servers" -> paste DC-1's private IP address and save.
</p>
<br />
<p>
  <img width="604" height="321" alt="7" src="https://github.com/user-attachments/assets/3328963d-5560-42d0-b40e-469cc5635a4d" />

</p>
<p>
Now we will log into Client-1's VM on Remote Desktop and will attempt to ping DC-1's private IP address from the clients virtual machine. The "ping" command tests network connectivity between devices using their private IP address on the same network. First, search "Powershell" in the windows search bar -> ping the Private IP address. If Packets sent and received are 4 for each, and there is 0% packet loss, then that means the domain controller is online and is connected to the client's machine.
</p>
<br />
<p>
<img width="677" height="546" alt="8" src="https://github.com/user-attachments/assets/0140aea3-1bac-420d-9d32-9c7d0418a427" />

</p>
<p>
Now, to verify we are connected to the domain controller's DNS server, we will enter "ipconfig /all" to display it's current network settings. Under DNS Server, we will see DC-1's private IP as the DNS server.
</p>
<br />
<p>
<img width="1917" height="762" alt="9" src="https://github.com/user-attachments/assets/2b0eff95-cf36-480c-9d85-562ead9f0dfc" />

</p>
<p>
To setup Active Directory within the domain controller, we will first enable Active Directory Domain Services. To do this, we will click start -> server manager -> Add roles and features -> hit next up until you reach "Server Roles" and check the box next to "Active Directory Domain Services" and add the features -> Once at the Confirmation page, check "Restart the destination server" prompt -> Install.
</p>
<br />
<p>
<img width="2314" height="741" alt="10" src="https://github.com/user-attachments/assets/ba45952c-b7b3-4968-a312-30f6aa7083fd" />

</p>
<p>
Next, we will setup DC-1 as a domain controller by linking the server to a domain, in this example we will use "mydomain.com". For this to happen we will click on the flag on the top right of the Server Manager board -> click "Promote this server to a domain controller -? click "Add a new forest" -> enter the root domain "mydomain.com". Fill in the password in the next screen and leave "Create DNS Delegation" unchecked -> Install.
</p>
<br />
<p>
<img width="646" height="672" alt="11" src="https://github.com/user-attachments/assets/28453d47-9636-4496-95a4-c3d92e06ab40" />

</p>
<p>
Log back into dc-1 with the following format for the User: (mydomain.com\username). Now we will proceed with creating a Domain Admin within the domain controller. First, we will hit start in DC-1 -> click the drop down under "Windows Administrative Tools" -> click on "Active Directory Users and Computers"
</p>
<br />
<p>
<img width="668" height="618" alt="12" src="https://github.com/user-attachments/assets/e2807eb2-77d6-464c-82ab-771294f18025" />

</p>
<p>
Now we will create an Organizational Unit for our domain by right-clicking on "mydomain.com" -> click New -> Click Organizational Unit -> Name the Organizational Units to your preference, in this example we created "_EMPLOYEES" and "_ADMINS".
</p>
<br />
</p>
<br />
<p>
<img width="750" height="522" alt="13" src="https://github.com/user-attachments/assets/c0deb8cf-6463-47ac-98e7-c38ba60440ac" />
<img width="1125" height="686" alt="14" src="https://github.com/user-attachments/assets/a11b3446-2ed3-44fa-9edd-3cd379b57f97" />

</p>
<p>
Now we will create a user, in this case named "Jane Doe", she will be listed as an Admin in our domain controller. To do this, we will go to Admin folder -> Right click and select "New" -> User -> Enter in credentials. To make her an Admin, we will have to add her as an admin in the "Domain Admins Security Group", so we will right click her name in the Admins folder -> Properties -> Member Of -> Click "Add..." -> Enter "Domain Admins" in the text box and then hit ok and then apply.
</p>
<br />

</p>
<br />
<p>
<img width="537" height="702" alt="15" src="https://github.com/user-attachments/assets/b08812f6-c204-40b8-9af1-c2aa7db068fb" />

</p>
<p>
Now we will log out of DC-1 and log back in as Jane with her newly created username and password.
</p>
<br />

</p>
<br />
<p>
<img width="1195" height="651" alt="16" src="https://github.com/user-attachments/assets/3aae7976-4001-4235-8468-fb71ca4e015e" />

</p>
<p>
Switching over to Client-1's VM, we will now link client-1's machine to our domain controller. To do this, right click the start menu and click "System" -> Rename this PC (advanced) -> Click change under the Computer Name tab" -> enter domain name under "Member of" with the domain bubble checked. Once prompted for admin credentials, we will enter the credentials for an Admin created in our domain controller.
</p>
<br />

</p>
<br />
<p>
  <img width="749" height="341" alt="17" src="https://github.com/user-attachments/assets/947e7d09-dc2b-4c61-86fa-cf1684caa7a2" />

</p>
<p>
To verify if Client-1's computer has been connected to our domain controller, we will first open up "Active Directory Users and Computers" on our domain controller virtual machine. Then we will click the drop down of "mydomain.com" -> then click "Computers". Here we will see client-1 listed under computers connected to our domain controller. This serves as proof that our client machine is now linked to our domain controller's forest domain via Active Directory.
</p>
<br />

</p>
<br />
<p>
<img width="747" height="528" alt="18" src="https://github.com/user-attachments/assets/987886a7-92c9-404d-992e-e645f493363b" />
</p>
<p>
Now we will enable remote desktop access for users wanting to access our domain remotely. To do this, we will first create an orgazinational unit named "_CLIENTS". Follow the steps previously mentioned to create the organizational unity. Then we will move our "client-1" computer from the Computers folder to "_CLIENTS"
</p>
<br />

</p>
<br />
<p>
<img width="747" height="528" alt="18" src="https://github.com/user-attachments/assets/987886a7-92c9-404d-992e-e645f493363b" />

</p>
<p>
Next we will log out of client-1 and log back in as our admin account created in DC-1. Then right click the start button -> System -> click Remote Desktop -> click on "Select users that can remotely access this PC" under User Accounts -> click "Add..." -> From here we will type "Domain Users" or "Domain Admins". This will provide the security group selected, access to the client's computer remotely. We now allowed all Domain Users to access client-1 remotely.
</p>
<br />

</p>
<br />
<p>
<img width="828" height="679" alt="19" src="https://github.com/user-attachments/assets/f10ac3c0-ce4e-4afc-8d3c-4e66db19da32" />

</p>
<p>
Now we will test the above by creating multiple users and attempting to log into client-1's VM with their credentials. First we will jump back to DC-1 and open up "Powershell ISE" as an administrator (in Windows search box, type Powershell ISE -> right click and run as admin).
</p>
<br />
<p>
<img width="1127" height="917" alt="20" src="https://github.com/user-attachments/assets/5399e82e-f9f8-4ad3-ac81-ecd2054abbb0" />

</p>
<p>
Once open, click the top left blank page icon to create a new script. Hit Ctrl + S , to save the file onto the desktop, we will name this file "create-users".
</p>
<br />
<p>
<img width="828" height="679" alt="19" src="https://github.com/user-attachments/assets/f10ac3c0-ce4e-4afc-8d3c-4e66db19da32" />

</p>
<p>
Now we will paste the following <a href="https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1">script. </p> <p> This will allow us to generate multiple accounts in the organizational unit "_EMPLOYEES".</p>

<br />
<p>
<img width="828" height="679" alt="19" src="https://github.com/user-attachments/assets/f10ac3c0-ce4e-4afc-8d3c-4e66db19da32" />

</p>
<p>
To create the users in the "_EMPLOYEES" organizational unit, press run, or the play button located on the top of the Powershell window. To verify the employee accounts being created, go back to Active Directory Users and Computers and click on the "_EMPLOYEES" folder. You will see new user accounts being created here. 
</p>
<br />
<p>
<img width="598" height="377" alt="image" src="https://github.com/user-attachments/assets/66b2d51b-ba1b-4fe8-af21-3e673b68a0c3" />

</p>
<p>
Here, we will use "bif.cod" as a domain user and log in using this account name onto Client-1 to test account creation and connectivity to Active Directory.
</p>
<br />
<p>
<img width="451" height="549" alt="23" src="https://github.com/user-attachments/assets/d61d0ad4-ff37-439f-8955-036b55987044" />


</p>
<p>
To verify an account has been successfully enabled for remote log in to our client-1 VM, we will log in as "bif.cod"
</p>
<br />
<p>


<img width="1548" height="942" alt="24" src="https://github.com/user-attachments/assets/f31818f8-deb6-4989-8485-ebc888810465" />

</p>
<p>
We are now officially signed onto client-1's virtual machine as a user we had generated using the script. This user will only have priveledges listed under Domain User's security group in active directory. We have now concluded our procedure on active directory in Azure.
</p>
<br />

