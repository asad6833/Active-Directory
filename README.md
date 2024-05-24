# Building an Active Directory Environment
 ![260781883-9a11fa39-b1ad-432e-a010-465602dce3bf](https://github.com/Lachiecodes/Active-Directory/assets/138475757/aad72280-e6e2-40a1-a7f3-6d3ce19dac06)

## Introduction
In today's digital landscape, establishing a robust network infrastructure is essential for organizations to ensure seamless operations and effective resource management. One critical component of such infrastructure is an Active Directory (AD) environment, which provides centralized authentication, authorization, and network management capabilities. 

For this project, VMware was utilized to create an integrated environment that consisted of a Windows Server 2019 Virtual Machine (VM) serving as the Domain Controller (DC) with Active Directory (AD) service. A custom PowerShell script was then executed to populate AD with approximately 1000 fictional users. I then created two VMs to act as clients on the AD server, one with Windows 10 Pro and the other with Linux Ubuntu 22.04.4 LTS, which where integrated into the AD internal network as CLIENT1 and CLIENT2 respectively.

## Technologies and Components Utilized:
- Active Directory
- AD Domain Service
- NAT
- DNS
- Networking
- PowerShell
- VMware
- Windows Server 2019
- Windows 10 Pro
- Linux Ubuntu 22.04.4 LTS

## Windows Server 2019 Setup
First, I downloaded VMware and an ISO for both Windows Server 2019 and Windows 10. The first virtual machine I created was with Windows Server 2019, which will be my Active Directory Domain Controllers. The DC houses 2 network adaptors, the first is to connect to the outside internet, and the second is to connect to the VirtualBox private network. You must make sure to assign 2 adaptors to the DC virtual machine in settings, the first using NAT, and the second dedicated to the internal network.

- Two network adapters were used to separate traffic between external and internal.
  
  ![68747470733a2f2f692e696d6775722e636f6d2f5751796478314d6c2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/1ba37026-0246-4941-85e2-3ec9954c411f)

Next, I set up IP Addressing for the internal network. After that, I set up Active Directory Domain Services and assign the Server 2019 machine as the DC using the domain: mydomain.com. Once the domain is created, I create an Organizational Unit named _ADMINS to add myself as a domain administrator.

The third step is to install RAS/NAT to allow the Windows 10 client (second VM) to be on the private virtual network, but still be able to access the internet via the DC. To do this, go to add roles & features > remote access > routing > install. Once completed, go to routing and remote access from server manager to install NAT (this allows internal clients to access internet using one public IP address).

Once RAS/NAT is set up, we will set up a DHCP server on the DC with the scope information pictured at the top. To do this, we go to add roles & features > DHCP Server > install. We can now go to tools > DHCP in order to set up the scope to give IP addresses in the range with the correct subnet mask. In DHCP settings we will use the DC IP address as the default gateway.

- The following roles and services were configured.
  ![68747470733a2f2f692e696d6775722e636f6d2f68444a347679666c2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/73d212ab-940d-48b2-a1c0-f0b0001e559e)

Next, we will use a PowerShell script to add 1000 users to Active Directory. To make it easier, we used a random name generator script to add the 1000 names to a .txt file. This way we can easily call on that text file in the PowerShell script.

- PowerShell script used to generate approximately 1000 fictional users.
  ![68747470733a2f2f692e696d6775722e636f6d2f64684e495263716c2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/4d6ed4db-6f2a-4372-8b55-24778f9e0d73)
  ![68747470733a2f2f692e696d6775722e636f6d2f434730507538766c2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/eb298c63-b9fe-455b-966f-c876d0a39358)

- After the configurations were completed, I then took time to explore AD by performing tasks such as: creating groups, organizational units, modifying user permissions, disabling users, deleting users, and so on.
![68747470733a2f2f692e696d6775722e636f6d2f3246564c6a356d6c2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/4d06030c-f4d3-47ea-8b1f-ed177f845519)

## Connecting Windows 10 Machine to AD
The final step is to create the second virtual machine running Windows 10. During the installation I placed the VM on the internal network and created a local account. I named it CLIENT1 for simplicity. I then updated the system's name and joined it to my domain. Once this is complete, go to the command prompt and type: ipconfig, then: ping www.google.com. This is to make sure the DNS server is working and the DC is properly NATing and forwarding traffic to the internet, and then returning the ping back to the client machine.<br>

- Configuring system name and connecting it to our Active Directory domain: mydomain.com<br>
  ![68747470733a2f2f692e696d6775722e636f6d2f3936427a416c696d2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/22ce57a1-ac13-4477-a610-ffcf895b7bc5)

- From there I verified the VM was connected to AD by logging into several of the random users I had created.
  ![68747470733a2f2f692e696d6775722e636f6d2f546f45363050676d2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/c082ad84-7199-4d2c-88c9-970f27caf648)

## Connecting Linux Ubuntu 22.04.4 LTS Machine to AD
I wanted to expand the lab to include a second client machine other than Windows, so I decided to connect Linux Ubuntu to Active Directory. This was a little bit more tricky to configure, as straight out of the box, Ubuntu LTS version doesn't automatically connect to Windows AD. During installation, you are presented with the option to enter your domain details, but during the installation this error message appears:<br>
![68747470733a2f2f692e696d6775722e636f6d2f736473453670706c2e706e67](https://github.com/Lachiecodes/Active-Directory/assets/138475757/69a15972-7efa-4a31-b2ed-3f143742ba4e)

This is a common issue with Ubuntu LTS version, and requires manual configuration of the integration into Active Directory. The steps are as follows:

Computer Name: CLIENT2<br>
Domain Name: mydomain.com<br>
Host Name: CLIENT2.mydomain.com<br>
Administrator Account: user<br>

1. Create your Ubuntu VM, update everything, and take a snapshot so you can easily go back if something goes wrong. Network settings will be the same as your Windows 10 Pro VM.
2. Open up the command line interface.
3. Verify you can ping the DC and that the DC can ping back.
4. Set the host name for the machine: `sudo hostnamectl set-hostname LINUX.mydomain.com`
5. Verify the host name: `hostnamectl`
6. Install the following: `sudo apt install sssd-ad sssd-tools realmd adcli`
7. Discover the DC: `sudo realm -v discover mydomain.com`
8. Install the kerberos default config file: `sudo apt-get install -y krb5.conf`
9. Edit the krb5.conf file. (Capilization matters, verify default_realm is your DC and add rdns = false): `sudo nano /etc/krb5.conf`
![Screenshot 2024-03-28 181543](https://github.com/Lachiecodes/Active-Directory/assets/138475757/669afca3-8548-4189-844e-6be507a005e8)
10. Install this package: `sudo apt install krb5-user`
11. Obtain Kerberous ticket with an account that has admin priviledges: `kinit username`
12. Connect the system to the DC: `realm join -v -U username mydomain.com`
13. Verify you have can see random users in your AD: `id user@mydomain.com`
![Screenshot 2024-03-28 181910](https://github.com/Lachiecodes/Active-Directory/assets/138475757/3c7c6f47-db25-41fd-865b-162b26c7c5a3)
14. Now log off your primary account and pick a random user in AD: user@mydomain.com
![Screenshot 2024-03-28 182106](https://github.com/Lachiecodes/Active-Directory/assets/138475757/9741e416-68b7-40f2-9352-0075d555b90f)
15. Finally, once Ubuntu does the initial setup, open the command line interface and verify: `who`
![Screenshot 2024-03-28 182201](https://github.com/Lachiecodes/Active-Directory/assets/138475757/e78bae65-6e93-4640-8a4a-b3de16aaa525)


## Conclusion
In this project, VMware was used to create an integrated AD environment. The first VM was a Windows Server 2019 that served as the DC, DHCP, and Active Directory populated with approximately 1000 users. Once the networking configurations were complete for the server, I then created two more VMs and connected them to an internal network using Active Directory.<br>

![Screenshot 2024-03-28 185527](https://github.com/Lachiecodes/Active-Directory/assets/138475757/9a1945d9-dac0-4c05-a879-448e53a70261)


Setting up a Windows Active Directory environment is a process that requires careful planning and execution. By leveraging virtualization technology and following best practices in configuration and policy settings, organizations can establish a robust foundation for their IT infrastructure.

It's important to note that specific configurations may vary based on software versions, network requirements, and security policies. Always refer to official documentation and consult with IT professionals when implementing complex network architectures. With proper implementation, an efficient and secure AD environment can be tailored to meet the unique needs of any organization.
