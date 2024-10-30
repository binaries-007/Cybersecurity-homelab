

## Introduction

Learning and implementing cybersecurity concepts can be challenging without access to practical and secure infrastructure. These challenges are further complicated by budget constraints that limit the acquisition of necessary hardware resources.

To overcome this, this home lab guide provides instructions for provisioning, configuring, optimizing, and securing IT infrastructure using a combination of local virtual machines (VMs) and cloud resources for practical use cases. This approach enables deploying less resource-intensive tools on local VMs while leveraging the cloud for more demanding applications. It simulates both on-premises and cloud environments. The knowledge gained here can aid in production and large-scale, enterprise-level infrastructures despite your smaller scale.

## What is a Home Lab?

A home lab is a personal setup within your home designed for hands-on practice and skill development in specific fields such as IT or cybersecurity. It mimics larger-scale infrastructures using similar components and tools, providing a safe and controlled environment to experiment, learn, and refine your skills.

## Lab Contents

<ul>
<li style="list-style='none';"><a href="#1" >Lab Design and Topology</a></li>
<li><a href="#2">Building/Choosing a Host PC</a>
<ul>
<li><a href="#2.1">Additional Suggestions for Local Setup Without Cloud Tunneling<a/></li>
</ul></li>

<li><a href="#3">Downloading, Installing, and Setting Up VMware Fusion for Mac  (VMware Workstation Pro for Windows)</a>
<ul>
<li><a href="#3.1">Setting Up Virtual Machine Networks (VMNets) on VMware</a></li>
</ul>
</li>
<li><a href="#4">Installing pfSense for Network Segmentation and Security</a></li>

<li><a href="#5">Installing Kali Linux</a></li>
<li><a href="#6">Configuring PfSense Interfaces and Dynamic DNS</a>
<ul><li><a href="#6.1">Configuring Dynamic DNS (DDNS)</a></li></ul></li>
<li><a href="#7">Creating and Setting Up a Microsoft Azure Account</a></li>
<li><a href="#8">Creating a Virtual Network And Setting Up a VPN Connection on Azure</a></li>
<li><a href="#9">Configuring VPN connection on pfSense</a></li>

<li><a href="#10">Installing and Configuring Security Onion</a></li>
<li><a href="#11">Configuring Packet Forwarding from pfSense to Security Onion using Netflow protocol</a></li>

<li><a href="#12"> Configuring a Windows Server as a Domain Controller</a><ul><li><a href="#12.1">Configuring Active Directory Certificate Services on our Domain Controller</a></li>
<li><a href="#12.2">Configuring DHCP Server on our Domain Controller</a></li></ul></li>

<li><a href="#13">Configuring Windows Desktops & Onboarding Users Accounts to the AD Domain</a></li>

<li><a href="#14">Installing and Configuring Splunk</a></li>

<li><a href="#15">Installing Splunk Universal Forwarder on Windows Server</a></li>

<li><a href="#16">Ubuntu/CentOS/Metasploitable/DVWA/Vulnhub Machines: Optional machines for exploitation, detection, and monitoring purposes</a></li>
 </ul>



<h2 id="1">Lab Design and Topology</h2>
![Lab Design and Topology](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sclc6n78n1ccxa0kapmk.png)

<br/>

<h2 id="2">Building/Choosing a Host PC</h2>

Due to budget constraints, I will use a MacBook Pro 2018 with 16GB of RAM, a 4GB dedicated graphics card, and a 512GB SSD for this lab. The recommended requirements for this lab are the same, a machine with at least 512GB of storage, an Intel Core i5 CPU (or its AMD equivalent), and 16GB of RAM. This lab is inspired by an article from [Cyberwox's blog](https://cyberwoxacademy.com/building-a-cybersecurity-homelab-for-detection-monitoring/).

**Note:** While macOS is used as the host platform for this lab, the hypervisor (virtualization software) is available on most platforms, including Windows, though configuration steps may vary slightly. 

<h3 id="2.1"> Additional Suggestions for Local Setup Without Cloud Tunneling</h3>

- **Build a Customized PC:** To run all virtual machines and instances locally, consider building a customized PC that meets the lab's requirements. You can follow this [article](https://cyberwoxacademy.com/building-a-cybersecurity-homelab-for-detection-monitoring/) for guidance.

<br/>

- **Leverage Existing Hardware:** If you have additional machines available, set up a cluster using Proxmox, provisioning networks, and VM instances as needed. Numerous tutorials are available on YouTube to help you through this process. I plan to explore this lab setup in the future.

<h2 id="3"> Downloading, Installing, and Setting Up VMware Fusion for Mac (VMware Workstation Pro for Windows) </h2>

VMware, (now owned by Broadcom) has made VMware Fusion and VMware Workstation Pro free for personal use. You can download and install these products by following their official blog [here](https://blogs.vmware.com/workstation/2024/05/vmware-workstation-pro-now-available-free-for-personal-use.html). If you encounter difficulties during the download or installation process, refer to these YouTube tutorials [here](https://www.youtube.com/watch?v=uMWDJwjlLNY) or [here](https://youtu.be/gp5eXjWZUBk).

Alternatively, VirtualBox is another virtualization option and can be downloaded [here](https://www.virtualbox.org/wiki/Downloads).

<h3 id="3.1">Setting Up Virtual Machine Networks (VMNets) on VMware </h3>

After installing VMware, launch the software and navigate to the menu bar at the top left corner.
![Menu Bar](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n42j5y39ifjzwp2499fm.png)

Click on **VMware Fusion** and select **Settings**.
![Menu Bar](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8qihwmnybjckjs5d2wlg.png)

Then, click on the **Network** tab in the settings window.
![Network Settings Window](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qvcg7e39ix7jt9uy15ka.png)

Here, you will see the predefined network configurations that define different network settings

- **"Share with my Mac":** Allows any VM using this setting to communicate externally (with the internet and other physical machines on the host’s network) using the host machine's IP address. All communications appear to originate from the host machine.
- **"Bridged Networking":** Enables the VM to act like a physical computer connected to the physical network.
- **"Private to my Mac":** Creates an isolated network where VMs can communicate only among themselves and are isolated from devices on the physical network.

Next, we will create four custom VMNets (vmnet2 to vmnet5) to assign machines to. Think of VMNets as networks on a router, where multiple machines can be connected. Each of these VMNets will be isolated, but they will be able to communicate with each other through a router. In this lab, pfSense will act as our router. Here is the configuration for VMNET2 to VMNET5:
![Network Config Window](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c7cp1ts9ulnjl8z6hs1s.png)


- Click the **+** icon to add new VMNets (vmnet2 to vmnet5).
- Leave the settings unchanged; there is no need to connect through the internet directly. Instead, we will use our virtual router device (pfSense) for greater flexibility.
- Ensure the host machine is not connected to any of these networks.
- Untick **provide addresses on this network via DHCP**

<h2 id="4"> Installing pfSense for Network Segmentation and Security</h2>

pfSense provides routing, firewall, and VPN functionality. In this lab, we will use pfSense as a firewall to segment our networks and set up a VPN tunnel from our networks to our AWS VPC.

1. Download the pfSense [ISO file](https://shop.netgate.com/products/netgate-installer?_gl=1*sxkmuw*_gcl_au*MTcwODkxNzI5MC4xNzI1MTMzMTQz*_ga*OTE0NDQ0OTc2LjE3MjUxMzMxNDM.*_ga_TM99KBGXCB*MTcyNTE3Nzg2Mi4zLjEuMTcyNTE3ODEwNi42MC4wLjA.), selecting "ISO IPMI/Virtual Machines."
2. Once downloaded, open your terminal, navigate to the download directory, and type `gunzip -d <file_name>` to uncompress the file.
3. Open VMware and create a new Virtual Machine.
4. Choose "Install from disc or image" and click **Continue**.
   ![VMware Setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g75t8i316w5m7uq6oplr.png)

5. Select the uncompressed ISO file and click **Continue**.
6. Choose "Legacy BIOS" if prompted.
7. Click **Customize Settings** and name the VM "pfSense" or a suitable name.
8. In the configuration window, click **Network Adapter** under **Removable Devices**. Add four (4) virtual network adapters and assign each adapter to the VMNets created earlier. This VM will act as a router and firewall, so it should be connected to all 4 custom networks.
9. Click **Add Device** to add additional network adapters until all VMNets are assigned.
   ![VMware Setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f0jraj21vemc9exlgtu3.png)

10. Go back to the main configuration window and click on **Hard Disk (SCSI)**. Set the disk size to 20GB and ensure "Split into multiple files" is selected.
    ![VMware Setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3zf33plh160yidns1jy3.png)

11. Click on **Processors and Memory** to allocate resources. Assign one processor and 2GB (2048 MB) of RAM.
12. Close the window and click the icon to start the VM.
    ![pfSense Boot Menu](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3damr0b8m1gddp2nangq.png)

13. Proceed with the installation by accepting all defaults, and pfSense will configure itself and reboot. If you encounter any issues, restart the VM.
    ![pfSense Setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mrbcmrg7bs1g61p77c0n.png)
 ![pfSense Setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z5fl8j7c32ftpfqdtafe.png)

14. After a successful reboot, select Option 1.
15. When prompted "Should VLANs be set now?", enter **n**.
16. Assign **em0**, **em1**, **em2**, **em3**, and **em4** to each respective question.
17. Confirm by entering **y**.
    ![pfSense Setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wy0mdgdv8liitjkznadz.png)

18. Now, configure the network interfaces:

- **LAN Interface (em1):** Use IP 192.168.1.1 to access the pfSense WebGUI via a Kali machine.

  ![LAN Interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dfe67se4hbly5zosckwf.png)

- **OPT1 and OPT2 Interfaces:** Configure as required.

  ![OPT1 Interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xvoayoka0y81skz7tq0k.png)

  ![OPT2 Interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3e2r4v2hnjlilfpkiovg.png)

- **OPT3 Interface:** Leave without an IP for span port traffic monitoring with Security Onion.

Further configuration will be done through the pfSense WebConfigurator via the Kali machine.



<h2 id="5"> Installing Kali Linux </h2>

Kali Linux is amongst the few Linux distros that come with a set of tools that can be used for offensive security. An alternative to Kali Linux is ParrotOS. Kali Linux can be used to perform attacks on the domain controller and other vulnerable machines in the lab. To begin, you can download the Kali Linux ISO image from [here](https://www.kali.org/get-kali/#kali-virtual-machines).

Download the image according to the VM platform you are using, for this lab, which will be VMWare. After downloading, extract the archive into an appropriate folder, and open the **.vmwarevm** file. You should see a window that starts the VM, kindly shut it down so we can configure its resources. Click the settings icon
![VMWare Settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g99e2hhnpctwps9060ct.png)

You should get the window below, 
![VMware settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cufoiuc206ozmxx2j1nb.png)

Proceed to **Processors & Memory** to provision the right resources, I will be using 2 cores, and 2GB (2048 MB) RAM for this VM. Also, go to **Network Adapter** to assign the default network adapter to **vmnet2**.

You can start the VM when done. The default user and password is **kali**. You can change the password by launching the terminal and using the `**passwd**` command.
![Change password](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2yh662cu5fc2mptrt1l2.png)



<h2 id="6"> Configuring PfSense Interfaces and Dynamic DNS </h2>

Now that the Kali machine is set up, navigate to the top left corner of the desktop window to open the Firefox browser, and enter **https://192.168.1.1**, this is the URL pfSense Web Configurator.
![Firefox](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/d4gu5ppzgn56vry2m3do.png)


![pfSense Web Configurator](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8hlzuojo5ugwodbm6czd.png)

Click **Advanced**, then, **Accept the Risk and Continue**. You should see the login page for the pfSense web Configurator. Login to pfSense using the default credentials, **admin** and **pfsense**.
![Pfsense](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kangs6qwv4qpl46sr22g.png)

Though this is a home lab, it is recommended to always change the default password of machines and software/platforms when provisioned.

Proceed with the wizard by clicking **Next** till you get to **Step 2 of 9**

Add **8.8.8.8** as the Primary DNS Server, and Add **1.1.1.1** as the Secondary DNS Server, these are Google's and Cloudflare's public DNS Servers respectively.

Proceed by clicking **Next**, at **Step 3 of 9**, Select your timezone.

Click **Next**, 
Untick the last two options at **Step 4 of 9**
![PfSense](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/73mwo1cjx5arbfafwkth.png)

At **Step 5 of 9**, Click **Next**
At **Step 6 of 9**, Set a new Admin Password, then Click **Next**
At **Step 7 of 9**, Click **Reload**, Click **Finish**.

At this point, the pfSense Wizard is complete and further configurations can be made. This is a home lab but, I recommend that you develop habits of creating a least-privileged user whenever you are using a root credential, as this will prevent account take-overs in real systems and ends up locking you out or wreaking havoc. You will find tons of tutorials about this on YouTube.

Now, let us proceed with configuring our interfaces.

Click on **Interfaces**

![Interfaces](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w7o4c19uv33og1ha2o9s.png)

Select **LAN**
![Lan](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/op0xaj39q5400mp2p3yr.png)

For **Description**, Change **LAN** to **SecAssessmentNetwork** as this is the network interface where Kali and Analyst machines will belong.
 
![SecAssessment Network](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nyfjkxqix66zu6djvcjv.png)

Scroll to the bottom of the page, Click **Save** and **Apply Changes**

If you encounter an error, you should check out this [article](https://blog.matrixpost.net/pfsense-2-5-0-bug-renaming-of-lan-interface-runs-into-an-error-regarding-router-advertisements-server-is-active/) to fix it.

Repeat the above steps until you have the named interfaces below

![Interfaces](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u4hrp3nl3zte5pga7hl4.png)

For **OPT3**, ensure you enable the interface as shown below

![OPT3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mcx1jm1uwgzbumtud083.png)

Next, Navigate to **Interfaces** _>>_ **Assignments** 

Select **Bridges**, click **Add**

**Member Interfaces**, Select **VICTIMNETWORK**

![Bridge](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4dkk5bxa8gbwc20rmbmi.png)

Click **Display Advanced**, under **Advanced Configuration** in the **Span Port** field, select **SPANPORT**

![SpanPort](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5d5lgvjwwt3p33vm2rlu.png)

Scroll down to the bottom of the page and click **Save** 


Next, we need to configure firewall rules,

On pfSense, navigate to **Firewall** _>>_ **Rules**

Under the **SECASSESSMENTNETWORK** tab, click **Add** to create a new firewall rule

Under **Edit Firewall Rule**, in the **Protocol** field, select **Any**, scroll to the page's bottom, click **Save**

**Note:** There's a predefined rule, the **Anti-Lockout Rule** created by pfSense to allow incoming connections to ports 80 and 443 which are the ports to its Web Configurator.

We added a rule to allow all connections to/from the SecAssessmentNetwork. We should avoid this as much as possible, this is only done for the convenience of the lab, and it is recommended for tweaking after the lab.



<h3 id="6.1"> Configuring Dynamic DNS (DDNS) </h3>

When we configure the Azure side of our VPN Tunnel, it is important that our VPN gateway can communicate with our on-prem gateway/router, which in the case of our home lab is our CPE (Customer Premises Equipment). Most SOHO (Small Office / Home Office) routers do not come bundled with a static or leased public IP address.

If we use the current public IP assigned, there is a high chance that Azure will lose communication with our gateway after some time (this is due to dynamic IP leasing by our ISP). To solve this, we can either lease some IPs from our ISP or use Dynamic DNS which enables Azure to track our public IP as it changes. Most SOHO routers have DDNS functionality, but in this lab, we will use pfSense.

There are so many DDNS providers, some domain name providers offer its functionality, while some providers offer it exclusively. Namecheap, Cloudflare, DynDNS, and NoIP, are some of these providers. For this lab, we will make use of DuckDNS (this is for practical purposes only, I recommend using Azure DNS, Cloudflare or Namecheap for reliable connections)

Proceed to [DuckDNS.org](https://www.duckdns.org/) to get started. Next, create an account and sign in. Enter the subdomain name of your choice and check if it is available. Once you have a domain name, note it and the generated token as it is needed in the following sections.

Next, navigate to the **install** section at the page's top nav bar.
![nav abr](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0j1gyeuuhf6925pi4htj.png)

Next, go back to our pfSense Web Configurator and log in, navigate to the **services** section and Click **Dynamic DNS**
![pfsense](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3slskw5wcnigptqazvpt.png)

Click **Add**
Select **Service Type** and Choose **Custom**
Navigate below to the **Update URL** section, and paste this
`https://www.duckdns.org/update?domains=<domain name given to you>&token=&<generated token>ip=%IP%` e.g, `https://www.duckdns.org/update?domains=example&token=f43562542412345676ip=%IP%`

In **Result Match** type **OK**. Enter **DUCKDNS** in the  **Description** field. Click **Save & Force Update** to finish setting it up. You should end up with something like this
![PfSense](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/80pkt5pfggp5dau8uofg.png)

We have successfully configured our DDNS.



<h2 id="7"> Creating and Setting Up a Microsoft Azure Account</h2>

In this lab, we decided to choose Azure as our Cloud Service Provider (CSP). To get started with this section you can log in with your existing credentials or open an account if you do not have one already, proceed to the [azure portal](https://signup.azure.com/signup).

Once created, new accounts are given $200 worth of credits to try out their services, while some services are always free, some have quotas, which is useful for this lab. One of the security best practices I have over time gotten accustomed to is avoiding using a super-user or root account for my regular tasks. This is useful as whenever your standard/privileged account gets compromised, you can quickly use the root account to withdraw its access compared to when a root account is ATOed. So let us create a different account for our daily use.

Proceed to the top-left corner of the page's nav bar, Click the hamburger menu
![Azure home](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dun333bvvvo8w5wuavcp.png)

Click on **Microsoft Entra ID**

Click on **Users**, and you should see a page with a list of users. When you open an Azure account, by default a new user is created for you, and it is assigned a **Global Administrator** role, this is the same thing as a root user.

Click on **New user**
![New user](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j468xtljp4uhnpglb3l0.png)

Click **Create new user**, fill in the fields

Proceed by clicking **Next: Properties**, Fill the necessary fields

Continue by clicking **Next: Assignments**

Click on **Add role** and Add the following roles
- **Network Administrator**

Click **Review + create ** to finish the user creation process.

We are done with our user creation, but we need to assign access to our created user on the subscription level. Click the hamburger menu and proceed to **Home**.

Click the **Subscriptions** from the **Azure services** section

![azure services](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gisfuwd6kryxr0hhb9bs.png)

Alternatively, you can search **subscription** on the search menu also
![search menu](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/eh6odwrarg4ds2heeiw6.png)

Select the subscription name, for new users, this will be **Azure subscription 1**.

Click on **Access control (IAM)**
![iam](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5mxxuwmj129avyaacw9m.png)

Click **Add**, then **Add role assignment**
![role assignment](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/juc6o9lghvcpoza07bcp.png)

Select **Privileged administrator roles**
![pam](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/261g9dctavzkervg4a0h.png)

Select **Owner**, Click **Next**

Click **Select members** and add our newly created user

Add a description (optional)
![member add](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ql71w6e4d96fmfaybgau.png)

Proceed by clicking **Next**

Select **Allow user to assign all roles except privileged administrator roles Owner, UAA, RBAC (Recommended) **

Click **Next**, then **Review + assign**

Sign out and re-login with the new user credentials, make sure you follow the prompts to enable MFA on the newly provisioned user.


<h2 id="8"> Creating a Virtual Network And Setting Up a VPN Connection on Azure </h2>

For the following steps, ensure you are logged in as our newly created and less privileged user. 

Click on **Resource groups**, this will help us create a container where we can create resources for our home labs and also assign a created user as the owner.
![Resource group](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qlm1vap8ze8n7s6epg4h.png)

Click **Create**

Assign a subscription (The free trial subscription is assigned to new accounts by default)

Enter **homelab-rg** in the **Resource group** field

Select an appropriate region closer to you and Click **Next: Tags**

Assign tag name **environment** and value **homelab**, this helps us to quickly filter our resources in the future.

Proceed by clicking **Next: Review + create**, Click **Create**

Click the refresh icon to see the newly created resource group. Proceed by clicking the resource name.


Now, let us create our virtual network 

Proceed by clicking the hamburger menu, then click on **Virtual Networks**

Click **Create**

Leave the defaults, and Enter a **Virtual network name** and **Region**
![virtual network config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9xcrg8f7g1kwwoi4ehts.png)

Click **Next** until you get to the **IP addresses** section

Enter **172.16.0.0/16** as the address space

![Vnet](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hbbtx9l13vq18vsw3d2l.png)

Click **Add a subnet**

Choose **Virtual Network Gateway** as the subnet purpose, and fill in the necessary fields using the below sample
![sample](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mjo8oozc2v175zdgotru.png)

Proceed with the remaining defaults by clicking **Add**

Click **Add a subnet** again. This time, we are creating a subnet with outbound internet access but restrictive inbound internet access. We need a subnet to place a NAT gateway, as our Security Onion instance will need to communicate with the Internet during installation. 

A NAT (Network Address Translation) gateway allows our resources to reach the Internet but prevents the Internet from reaching them. Although we can use the default subnet created automatically for us, I decided to have the default subnet be a more restrictive private subnet that does not have access to the Internet, so I will not be assigning a NAT gateway to it.

Now let us create our NAT-enabled private subnet

Proceed by following the inputs in the sample
![NAT-subnet](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kv7swpwo7tyw0qnixcix.png)

Ensure you selected **Enable private subnet (no default outbound access)** as we want to explicitly grant outbound access.

Next, in the **NAT gateway** section, click **Create new**

Enter a name for the NAT gateway

Create a Public IP address for the NAT gateway also
![natgw](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6cgnfr51lczrojnzfb14.png)

You should end up with the below setup
![subnets](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gbsfjjqjw3361t6z3jqi.png)

Proceed by clicking **Next**

Add a tag name **environment** and a tag value **homelab**
![tag](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kr05necafj4xbbg2s2b8.png)

Click **Review + create**, then **Create**

![summary](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ejad0hv0vigsz3g885kp.png)

Next, You can go to the home screen.

Now that we have successfully created our Virtual Network, Let us create our VPN gateway and set up our site-to-site VPN connection.

On the home screen, proceed by clicking **Create a resource**
![create a resource](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/suhjy52s5fwtw7nscx87.png)

Under the **Categories** section, click **Networking**
![categories](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/uo72v0q24i8p95btznbz.png) 

Click on **create** under the **Virtual network gateway** section
![vnet-gw](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oac6w08zcfs0md8i5hba.png)

Enter a name, **homelab-vnetgw**

Proceed to the **Virtual network** field and select the virtual network we created
![vnet](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u92okoxescbw9v7ee9cs.png)

In the **Public IP address**, select **Create new**

Give the Public IP address resource a name

Fill in the remaining fields by using the sample below
![vnetgw](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8x4ng7rfbgkjdbwgh1jf.png)


Click **Review + create**, then **Create**
![vnetgw](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nyxrfcoh0sttx3cbnfdu.png)

It takes roughly 20 minutes for our Virtual Network Gateway to be fully deployed.

Next, we need to create a local network gateway to enable us to create a connection to our on-prem machine.

To begin, proceed by Clicking **Create a resource** on the home screen and selecting **Networking** on the **Categories section**
![Local netgw](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ylvnxteosepd0oh3qedr.png)

Click on **Create** in the **Local network gateway** section.

Select the appropriate resource group

On the **Instance details** section, enter an instance name of your choice

Select **FQDN** on our **Endpoint** field

Enter our DDNS FQDN (Fully Qualified Domain Name) we created from duckDNS.org e.g. example.duckdns.org

Next, add the address spaces to our on-prem machine. You should end up with something similar to the one below 
![lngw](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t0714u5oyxz9de1bva1w.png)

Proceed by clicking **Review + create**, then **Create**

Finally, to finalize the Azure end of the VPN connection, we need to create a connection instance. 

Let's proceed by creating another network resource, this time a connection.
![connection](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9si4tlljmcrg0ozglkta.png)

Proceed by clicking **Create**

Select the appropriate resource group

In the **Connection type** field, select **Site-to-site (IPsec)**

Give it an appropriate name
![conn](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ojpnhoy1c8ghngmnb06u.png)

Proceed by clicking **Next: Settings**

In the **Virtual network gateway** field, select the virtual network gateway we created earlier

In the **Local network gateway** field, select the local network gateway we had earlier created 

In the **Authentication Method** field, leave it as **Shared Key(PSK)**

Enter a Pre-Shared Key (PSK) of your choice, this is more like a password, but I recommend that it should be complex and hard to brute force, though in a production environment, you will use a Public Key Infrastructure. I will be generating a key [here](https://randomkeygen.com/). 

Do save your PSK as it is needed on the pfSense side.
![conn](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j553me9fyz3q7ki4f2bd.png)

Next, Choose **Custom** in the **IPsec / IKE policy** field, and use the below entries
![IPsec config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/um1ld0qepsjwmmo0o106.png)

Proceed by clicking **Review + create**, then **Create**.

Once the connection is created, click **Go to resource** and **Download configuration**
![config download](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hvoka2r5l7y3uieee6lt.png)

Fill in the fields with the values in the sample below
![config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wrxa9epto1g7hnle7nle.png)

Save the configuration file as it will be needed in setting up the pfSense end of the VPN connection.

We are done setting up the VPN connection at the Azure end.

<h2 id="9"> Configuring VPN connection on pfSense</h2>

In this section, we are going to configure the pfSense part of the VPN connection. We proceed by navigating to our pfSense Web Configurator via our Kali Linux VM. Login to the pfSense Web Configurator portal.

Click **VPN**, then **IPsec**

Go to **Tunnels**
![tunnels](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/z3bzbz2uemrk7j58478f.png)

Click **Add P1**

In the **Description** field, enter any description of your choice, e.g. **Homelab to Azure Site-to-site tunnel**

In the **Remote Gateway field** of the **IKE Endpoint Configuration** section, enter the Public address found in the **Network parameters**  of the configuration file we downloaded.

The configuration entries we use in this section can be found in the **Network parameters** and **IPsec/IKE parameters** sections of the configuration file we downloaded.
![config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yhoqba52sxkyhv0owspp.png)

![general section](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i4100rw8abhhn7mps1rj.png)

Next, skip to the Pre-shared key field, and enter the PSK we used while setting up the Azure part of the connection, you can also find it in the  **IPsec/IKE parameters** section of the configuration file we downloaded.
![Ipsec config](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/slglqdbmwej1pd73nkfh.png)

Proceed with the defaults, and click **Save**, then **Apply Changes**

Next, click **Show Phase 2 entries**, click **Add P2**, Enter the next configurations using the sample below
![phase 2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y88aurczx1t0u4j7jd1q.png)
![phase 2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s0rgu4rqzosxyd2lgc3n.png)

Click **Save** and **Apply Changes**

Next, proceed by navigating to **Status**, Click **IPsec**

![status](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cf3ivbzgpliuhie3r418.png)
![status](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6vd7pltlmrksd8yl5ayv.png)

Click **Connect P1 and P2s**, You should get something similar to the below output
![output](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3s4qq16gp3hfv1wnvftk.png)

**Note**: If you encounter any issues with the above step, make sure you are not behind a firewall or be sure to allow IPsec traffic on your host machine or modem.

After a couple of minutes (which takes around 10 minutes), hit the refresh button on the connection instance, and you should see that you are connected 
![status](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qajpr7n8fkusemy28lzv.png)

And, we are done setting up our Site-to-site VPN tunnel


<h2 id="10"> Installing and Configuring Security Onion</h2>

In the previous section, we successfully configured our VPN tunnel. In this section, we will configure Security Onion as our IDS solution. Security Onion is a free and open platform that can be used by cybersecurity analysts and engineers. You can read more about Security Onion [here](https://docs.securityonion.net/en/2.4/introduction.html).

We will be using the **Eval** Node Type of the Security Onion Architecture which is used mainly for testing purposes, it enables us to sniff live network traffic. The evaluation mode simply allows us to test out Security Onion. 
![seconion](https://docs.securityonion.net/en/2.4/_images/network-horiz.png)
source https://docs.securityonion.net/en/2.4/_images/network-horiz.png

Security Onion has heavy resource requirements, for high-end labs, you may not worry about it, but this lab allows us to provision resources in the cloud when our lab cannot handle such resources.

Let us head to Azure to provision our security onion instance.

There are two ways we can provision our Security Onion instance, the first is using a production-ready image from the Azure marketplace, and the other is creating one from scratch, while I will show you how to accomplish the first, this lab will focus on the latter. 


### Option 1 - Creating a Security Onion distro via Azure Marketplace

Navigate to the portal's home screen, Click the search bar and search **security onion**, you should see something similar to the one below

![security onion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k7x1lx1m6pr2lvr0mndp.png)

Click **Security Onion** in the **Marketplace** search results

**Note**, You get a first month free using this image from the marketplace, then starting at **0.028/hr** plus Azure Infrastructure costs. Alternatively, you can rent a VM and upload the security onion image which is free (though you need to still take into account Azure infra costs).

![sec onion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qc755xno33cve23bt6dd.png)

![sec onion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a1rbp8e49dolcqnxpaax.png)

Click **Create**
![sec onion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mzhgs1naqjjj3rh84u9v.png)

Next, you will be taken to **Create a virtual machine window**

For new customers with trial subscriptions, most of the costs incurred in creating and using this machine will be deducted from the trial credits of $200. Let us proceed

Select the appropriate resource group

Enter a **Virtual machine name**

For **Availability options**, select **No infrastructure redundancy required**

**Security Type**, enter **Standard**
![vm](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r8j6dpfpxd8dt9aah8e2.png)

**Size**, It is recommended to use 4vcpus and a minimum of 12GiB memory to run an Evaluation instance, please be cost-conscious.

**Authentication type**, Select **SSH public key**, this is needed as we will configure the instance via our home-lab Kali Linux.

Generate a new key pair
![seconion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dfdphshw7rrercxz8287.png)

Click **Next: Disks** for disk set-up

At the **OS disk type**, change to **Standard SSD**

Untick **Delete with VM**, This enables us to tear down the instance without losing the saved data (we avoid paying for the VM instance, but pay little for the storage), **you will have to delete it separately if not needed**
![seconion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/armcq9k6klmvqnwrxdel.png)

Proceed by clicking **Next: Networking** to set up our network configuration

We want to select our home lab virtual network

**Subnet** should be the default or any private subnet within the vnet.

**Public IP**, select **None**
![Subnet](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fwnx34o0gn59utdh783n.png)

Proceed with the remaining defaults by Clicking **Review + create**
![sec onion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1eeiw0eojil516kcpt9r.png)



### Option 2 - Creating a Security Onion distro from scratch

Using the previous step is great for production use cases especially when we have extra bucks to spare. But in this lab setup, we will build our instance by creating our Security Onion distro. Let's get started by going to our Azure portal.

Proceed by creating a new virtual machine, 

**Resource group**, select the **homelab-rg** resource group we created

**Virtual machine name**, give it an appropriate name

**Image**, click **See all images**

Search for **Rocky Linux**

Choose **Rocky Linux for x86_64 (AMD64) - Official**

Select **Rocky Linux 9 - x64 Gen 2**

![VM creation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9gr0yuihq3xgzd8l0uj4.png)

![rocky linux 9](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4dyslk72ytijlavej572.png)



**Size**, select **B4ms** (this has 4 vCPUs and 16 GiB RAM which is the recommended requirements to run an EVAL version of Security Onion) 

**Note** Make sure you deallocate all VM instances when not in use as they can accumulate costs.


**Authentication type**, select **Password**


**Username**, enter **securityonion**

**Password**, Enter a password

**Public inbound ports**, select **None**

![VM creation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sci274a4n0wi4gvsjotl.png)

Proceed with the remaining defaults, 

Click **Next: Disks**

**OS disk size**, Select at least 200GB as it is the least recommended in the Security Onion docs.

**OS disk type**, Select **Standard SSD** (gives us reduced cost)

![VM creation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jm08fdceekhg0nr63pto.png)


Click **Next: Networking**

**Subnet**, Select a subnet with a NAT gateway attached (as it is needed to connect to the internet)

**Public IP**, select **None** (we only want to connect via our VPN and not via the public internet)

![VM creation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fwq1w1wp3c8n0xi19dbh.png)


Click **Review + create**, click **Create**


Once created, Click **Go to resource***

![Go to resource](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iugnktbj8rq06qmqd60s.png)

Click on  **Stop** to stop the running VM

![stop VM](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/szjkdih23s70e5k7nuf8.png)

Next, we need to create 2 network adapters to be attached to our instance
 
select **Network settings** under the **Networking** section at the left sidebar of the page.

![side bar](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hl34y6yfkkk9zyqjm0mg.png)

Next, click on **Attach network interface**

![Network interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ke2jfoncpbkbxinhrs73.png)

Click **Create and attach network interface**

![create and attach network interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o7v9vbfcaflty2rpf2xi.png)

**Resource group**, choose the **homelab-rg** resource group

**Subnet**, choose the default subnet (172.16.0.0)

**Private IP address assignment**, choose **Static** and provide an IP address

![vtap](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/45yfrktfoutkphfml4ff.png)

Proceed by clicking **Create**

Once created, we need to create another one,

![vtap2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kk5ffovrz9zb9fntfp3s.png)


Next, go back to **Overview** and Start the machine (ensure the 2 NICs are attached before starting).


Now, let us connect to our instance via our local Kali VM. Proceed by launching the terminal on Kali, and make sure that you are connected to our Azure VPC via VPN.

First, we try to ping our security onion instance

![Ping](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kav3linj349772vkc03n.png)

If you are having any trouble, 
- Make sure your VPN is working
- Your remote instance is up and running
- Your DDNS settings are correct and your current IP has been updated

Next, we SSH into our remote VM

![ssh](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6pw6it1ixrrlbgk1jpoi.png)

Before we install Security Onion, we must configure the newly attached Network Interface Card (NIC). 

First, we need to know the network adapters we will be configuring. We can know this by listing the interfaces on our instance. Type:

`sudo ip addr show`

![ip addr show](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u8wy7ya72h2qwme0wjh1.png)

In the screenshot above, we have the following NICs

| Interface | IP Address  | MAC Address       |
| --------- |------------ | ----------------- |
| lo        | 127.0.0.1   | 00:00:00:00:00:00 |
| eth0      | 172.16.2.4  | 7c:1e:52:5f:57:44 |
| eth1      | 172.16.0.10 | 60:45:bd:97:80:08 |
| eth2      | 172.16.0.11 | 60:45:bd:97:84:0a |

While the IPs and interface cards might be similar in your set-up, the MACs will be different, and it is okay.

eth0 interface will be used as the management interface, take note of its MAC address

eth1 will be used as a monitor interface

eth2 will be used to receive NetFlow traffic from our pfSense machine.

Let us configure the newly attached NICs.

```
sudo dnf update -y

sudo dnf install NetworkManager-dispatcher-routing-rules -y

sudo systemctl enable NetworkManager-dispatcher.service

sudo systemctl start NetworkManager-dispatcher.service

echo "201 eth1-rt" | sudo tee -a /etc/iproute2/rt_tables
echo "202 eth2-rt" | sudo tee -a /etc/iproute2/rt_tables

sudo tee -a /etc/sysconfig/network-scripts/rule-eth1 <<EOF
from 172.16.0.10/32 table eth1-rt
to 172.16.0.10/32 table eth1-rt
EOF

sudo tee -a /etc/sysconfig/network-scripts/rule-eth2 <<EOF
from 172.16.0.11/32 table eth2-rt
to 172.16.0.11/32 table eth2-rt
EOF

sudo tee -a /etc/sysconfig/network-scripts/route-eth1 <<EOF
172.16.0.0/24 dev eth1 table eth1-rt
default via 172.16.0.1 dev eth1 table eth1-rt
EOF

sudo tee -a /etc/sysconfig/network-scripts/route-eth2 <<EOF
172.16.0.0/24 dev eth2 table eth2-rt
default via 172.16.0.1 dev eth2 table eth2-rt
EOF

sudo systemctl restart NetworkManager

```


We can now proceed to install some packages (creating our Security Onion distro)

```
SEC_ONION_REPO="https://github.com/Security-Onion-Solutions"

sudo dnf update -y 

sudo dnf install git -y

git clone ${SEC_ONION_REPO}/securityonion.git

sudo chown $USER:$USER securityonion

sudo mv securityonion /opt/

sudo /opt/securityonion/so-setup-network

```

![Package installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w5chpkgyhsf0w92ikbr5.png)
 
Next, you should see an interface like this

![Sec Onion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jyequj2e4123l5quicua.png)

**Would you like to continue the install?**, select **Yes**

![Sec onion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6htfliq20xs42rsiypd4.png)

**Would you like to continue?**, select **Yes**


![EVAL](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/grw0k44emzypgt97cjlc.png)

**What kind of installation would you like to do?**, select **EVAL**


![agree](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2viayffx82mjlpsrjd9h.png)

Type **AGREE**, and select **Ok**

**Enter the hostname (not FQDN) you would like to set:**

Enter a hostname of your choice

![hostname](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hp2qc15lcgj1jv7m3l8w.png)

Select **Ok** or press **Enter key** to proceed

**Since this is a network install we assume the management interface, DNS, Hostname, etc are already set up. Select Yes if you've already configured these settings. Otherwise, select No to quit.**, Select **Yes**

![network install](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f3di2afgc7ei4he3a5l9.png)


Next, select **Ok**

![Warning](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f3ty9bdk31x1nxgtacn2.png)

**Please select the NIC you would like to use for management.**, Select the first item, **eth0** and proceed by pressing **Enter key** or **Ok**

![NIC](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yqxk7eshgpgfgxksz7s5.png)

**How would you like to connect to the Internet?**, Select **Direct**

![Direct](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4u7ww5nq3hxji0elxy86.png)

**Do you want to keep the default Docker IP range?**, select **Yes**
![Docker IP Range](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ijq8fykj43neerx71ihe.png)

**Please add NICs to the Monitor Interface:**, Using the Spacebar Select **eth1**  as the monitor interface.

![Add NICs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zb9rdk7hq84pepnrpio0.png)

Next, Enter an email and password which will be used to create the admin account.
![Email](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/57wqpqn0ygp3lt4acarn.png)

![Password](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rhv5mv7np47pr2pg0au4.png)

**How would you like to access the web interface?**, Select **IP**

![IP](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/paok06bdtftl2vba93qa.png)

**Do you want to allow access to this Security Onion installation via the web interface?**, Select **Yes**

![Web Interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ji2ykm9ml52wumw93tt9.png)

**Enter a single IP address or an IP range, in CIDR notation, to allow:**, Enter **192.168.1.0/24**

![192.168](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/anwlku17orgf2qq0wvqv.png)

Next, your choice

![Telementry](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e5zr5x530d4vybt01s7t.png)


Proceed by selecting **Yes**

![Selecting yes](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xpkr5xkw0gnudct3agor.png)

Next, Security Onion will start installing some necessary packages and proceed to configuring them. Go have a coffee, as this may take a while.

![Package install](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8s6ule0ouptr02kdrqdg.png)

After a while, you should get an interface similar to the one below indicating a successful installation.

![Successful install](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bablas3fcc56e19mimzu.png)

Next, open our browser on the Kali VM, navigate to https://172.16.2.4, Click **Advanced**, then click **Accept the Risk and Continue**

![firefox](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/374dhrtaoqosoavztx37.png)

We should have something similar to the one below

![Seconion login](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zmofxcbk0zeyqg6e7p0r.png)

This is the Security Onion Console's login page, enter the email and password you used when setting up Security Onion. You should see something similar to the one below on a successful login

![soc dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5c08vjcvvui1nb10m6ce.png)

We now have a Security Onion instance we can practice with.


<h2 id="11"> Configuring Packet Forwarding from pfSense to Security Onion using Netflow protocol </h2>

In this lab session, we need to forward packets captured by the SpanPort interface to our security onion instance on Azure.

Firstly, we need to add the Elastic integration for NetFlow Records on our Security Onion instance, we can do this by logging into our Security Onion console page via our Analyst Workstation (Kali VM).

Next, click on **Elastic Fleet**, enter the credentials you used when creating the Security Onion instance

![Elastic Fleet](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0kbh8fvl6c0x7r7ir1d6.png)

![Elastic Fleet login](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ld7wxc00dzgtylaxt2xp.png)

On Elastic dashboard, click **Agent policies** tab, click **so-grid-nodes-general**

![Agent policies](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2cr612okbndtz6i970op.png) 


Click **Add integration**

![Add integration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e1r987g8xvts8tilevjm.png)

Search for **netflow** and then click on **NetFlow Records**

![NetFlow Records](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7dgb1rinsjrefkrwsyrf.png)

The Elastic Integration page will show an overview of the NetFlow Integration. Review all information on the page and then click the Add **NetFlow Records** button.

In the **Add NetFlow Records integration** page, enter the following values for the fields:

**integration name**: **netflow**

**UDP host to listen on**: **0.0.0.0**

**UDP port to listen on**: **2055**

![Add NetFlow Records integration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ns1i4wlf1w4t69w1y1xj.png)

Click the **Save and continue** button and then click **Save and deploy changes**

Next, we need to allow netflow traffic through the firewall on our Security Onion instance. Let us do this going back through our Console dashboard

Navigate to **Administration** _>>_ **Configuration**

![Administration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yf97hbq5m0a6ndsx1xe9.png)

At the top of the page, click the **Options** menu and then enable the **Show advanced settings** option

![Show advanced settings](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/44m1ime15x5d25zyapsq.png)

On the left side, go to **firewall**, select **hostgroups**, and click the **customhostgroup0** group. On the right side, enter the IP address/CIDR block of the NetFlow exporter (192.168.0.0/16) and click the checkmark to save.

![firewall group0](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x55by339sko9p61c5h9k.png)

![right side](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ow8z92zhm1430cqqqawz.png)

On the left side, go to **firewall**, select **portgroups**, select the **customportgroup0** group, and then click **udp**. On the right side, enter the NetFlow listener port (2055) and click the checkmark to save.

On the left side, go to **firewall**, select **role**, and then select the node type that will receive the NetFlow records (eval). Then drill into **chain** _>>_ **INPUT** _>>_ **hostgroups** _>>_ **customhostgroup0** _>>_ **portgroups**. On the right side, enter **customportgroup0** and click the checkmark to save.

![role setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/com4u9xnsxbp620vhwy3.png)

Under the Options menu at the top of the page, click the **SYNCHRONIZE GRID** button to immediately apply the rules

Next, let us proceed by logging into our pfSense dashboard, navigate to **System** _>>_ **Package Manager** _>>_ **Available Packages**

![Package Manager](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6e59i72wk1ktzhdrobeh.png)

![Available Packages](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/r2qecr9rhj89ongzqlx4.png)

In the **Search term** field, search for **softflowd** and install it

![Softflowd](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kk4c8ejcptem0c2bxe4k.png)

Next, navigate to **Services** _>>_ **softflowd**

![softflowd](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/28ctlf7gzlvw3p489ya4.png)

Under the **General Settings**, in the **Interface** selection box, select **SPANPORT** 

**Host**, enter **172.16.0.11**
**Port**, enter **2055**

Scroll to the bottom of the page and click **Save**

Once all configuration is complete, you should be able to go to the Security Onion Console and under **Dashboards**, select the NetFlow dashboard to see your NetFlow records.

![dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ay5ez0ihctfl7emkm3zk.png)

![dashboard](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/89zgdoysgas81lt6by51.png)

You can also collect firewall logs  from pfSense by following the steps [here](https://docs.securityonion.net/en/2.4/pfsense.html)

That is all for Security Onion

<h2 id="12"> Configuring a Windows Server as a Domain Controller </h2>

In this section of the lab, we will set up an Active Directory (AD) Domain using a Windows 2019 Server as the Domain Controller and also proceed to add 2 Windows machines to the Domain Controller.

Proceed by downloading the [Windows 2019 Server Eval Copy](https://go.microsoft.com/fwlink/p/?linkid=2195685&clcid=0x409&culture=en-us&country=us) and [Windows 11](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise)

**Note:** Before proceeding with the Windows Server installation, do not start the machine, until: 
 
* Ensure you install on VMWare with the defaults
* Ignore the Product key and simply skip it
* By default, a network adapter is attached when creating the VM, ensure you change the assigned network to **VMnet3**

![Windows Server](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vs24avuoxbwx7rrobtc4.png)

Let us proceed by powering up the VM.

Click **Next**

![Windows](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m8myiwd7azanir5ckv4j.png)


Click **Install now**

![install now](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/c0i0e1non0wi228clen2.png)

Select **Windows Server 2019 Standard Evaluation (Desktop Experience) **
![windows](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xg761s1zem0awk7ie06j.png)

Accept the licence terms and Click **Next**

![Accept License](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vjo7k84enomc5zjhhksu.png)


Select **Custom: Install Windows only (advanced)**

![windows](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5h9i2w5i37qw6sjag4tq.png)

Click **Next**

![windows](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zltk5ii57dcirbvhh96h.png)

When the installation completes, create a password, and sign in

![Customization](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a24prnx50rwl6djv87rf.png)

Upon a successful installation, you should end up with the screen below

![Upon installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i7xkmlzqrgl7lm5wxgw6.png)



### Rename the Domain Controller

1. **Open System Properties**:
  - Press `Win + R` to open the Run dialog.
  - Type `sysdm.cpl` and press `Enter`.

2. **Change the Computer Name**:
   - In the **System Properties** window, go to the **Computer Name** tab.
   - Click on **Change** to rename the domain controller.

3. **Enter the New Name**:
   - Under **Computer Name**, type the new name for your domain controller.
   - Click **OK** and follow any prompts.

4. **Restart the Domain Controller**:
   - A restart is required to apply the name change.
   - The domain controller will restart and reflect the new name upon completion.


![Update Domain Controller Name](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tjoijiyzkd6kastp78tt.png)


After the reboot, On the **Server Manager Dashboard**, Click **Manage** _>>_ **Add Roles and Features**

![Server Manager](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/aayxg7vc5fku2m7pey6r.png)

Click **Next** until you get to **Server Roles** Menu, Select **Active Directory Domain Services**

Click **Add Features**

![Features](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t2g8aivfy3nl8pojsggw.png)

Proceed by Click **Next** until you get to **Confirmation** Menu, then click **Install**

![Confirmation Menu Screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k51yz2mxy0ltmn0g72mh.png)

After the installation, Click **Close**

![Installation progress screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/udgipub6w5dxpup4iujn.png)

Next, on the top-right corner of the dashboard, click on the flag with a yellow caution icon. Then click **Promote this server to the domain controller**

![Flag with caution](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v7ggf9s6625j0uto1ta8.png)


- Select **Add a new forest**
- Specify a domain name
- Click **Next**

![Create a Forest](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m7i5cr426ojckh7fdk6e.png)

Set a password

Click **Next** until you get to the **Prerequisites Check** Menu

Click **Install** and wait for reboot.

<h3 id="12.1"> Configuring Active Directory Certificate Services on our Domain Controller </h3>

In this sub-section, We aim to install and configure AD Certificate Services

Once the system reboots, Log back in

Select **Manage** _>>_ **Add Roles and Features** 

Click **Next** until you get to **Server Roles**

Select **Active Directory Certificate Services**

Click **Add Features**

![Active Directory Certificate Services screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j6wmg1xmsy5allr56wf0.png)

Click **Next** until you get to **Confirmation** Menu

Check **Restart the destination server automatically if required**

Click **Yes** in the pop-up dialog box

Click **Install**

![Confirmation screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/us2l7o1rzs72y342as6w.png)

After the installation, click **Close**

Next, Click on the flag with the yellow caution icon located at the top-right corner of the page

Click **Configure Active Directory Certificate Services on the destination server**

![Active Directory Certificate Services](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/60d1owsyucqc8etdc1x4.png)

On the wizard screen, click **Next**

![Wizard screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fit6fgx6bi22ps76llae.png)

On the **Role Services** Menu, check **Certification Authority**

Click **Next** until you get to **Validity Period** Sub-Menu under **Private Key**

Change to **15** Years, Click **Next** until you get to **Confirmation** menu

![PKI setup](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f5w0uyu3o0decr7r3m5j.png)

Click **Configure**, then click **Close**

![Configure](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xmrh3lvqz118sa8ap55i.png)

You should manually restart the server for changes to take effect.


<h3 id="12.2"> Configuring DHCP Server on our Domain Controller </h3>

In this sub-section, our aim is to set up Dynamic Host Configuration Protocol (DHCP) Service so our domain controller can issue IPs on its network. 

**Note:** We could have enabled DHCP for the network on the pfSense side, I chose this approach instead.


Select **Manage** _>>_ **Add Roles and Features** 

Click **Next** until you get to **Server Roles**

Select **DHCP Server**

Click **Add Features**

![DHCP Server screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/49yqmm41h89uqo26zbu9.png)

Click **Next** until you get to **Confirmation** Menu

Check **Restart the destination server automatically if required**

Click **Yes** in the pop-up dialog box

Click **Install**

After the installation, click **Close**

Next, Click on the flag with the yellow caution icon located at the top-right corner of the page

Click **Complete DHCP configuration**

![Complete DHCP configuration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v87e8njl1kp1nf8kqyry.png)

On the wizard screen, click **Next**

On the **Authorization** Menu, click **Commit**

Click **Close**

Next, let's configure DHCP Scopes

On the top-right corner, click **tools** _>>_ **DHCP**

In the DHCP management console, click on our domain name **(cybercrex.internal)

Right-click on **IPv4** (or **IPv6** if applicable) and select **New Scope**.

![New scope](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ov040bhbdjw6ngm6miss.png)

Follow the New Scope Wizard to configure a range of IP addresses, subnet mask, and other options to be distributed to clients.

**Name**, enter **Desktop clients**

Click **Next**

**Start IP address**, enter **192.168.2.1**

**End IP address**, enter **192.168.2.254**

Click **Next**

Exclude the following ranges,

**192.168.2.1** to **192.168.2.10**

**192.168.2.201** to **192.168.2.254**

![IP exclusion](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/feba6aa00hqesr7398xh.png)

Click **Next** until you get to **Router (Default Gateway)

Add **192.168.2.1**
![router gateway](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o5jue8zznq5popsaelm6.png)

Click **Next** 

**Server name**, enter your domain controller name <domain_name>-dc (e.g.cybercrex-dc) and click resolve

Next, click **Add**

![DNS](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0eyjoclxudlg8ko8g7l1.png)

Click **Next** until **Finish**


Next, let us add some users

On the **Server Manager** dashboard, Navigate to the top-right corner of the screen, click **Tools** _>>_ **Active Directory Users and Computers**

![Active Directory Users and Computers](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mj052hy88vfrztyevron.png)

Select your domain name **(cybercrex.internal)** _>_ **Users**

Right-Click on **Users** _>_ **New** _>_ **User**

![Add User](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m3z93uw1g3t3gezxse3l.png)

Fill in the User details, **First Name**, **Last Name**, and **User logon name**

![User Add window](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wpzbmz8v7qr903jxoipw.png)

Enter a password (in an organization, this can be a deterministically created password which is then required from the user to change at the next logon)

Check **User must change password at next logon**

![Password](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/84lvgik9ndnn65l4um4w.png)

Click **Next**, _>>_ **Finish**

Next, create another user with  different details

![Another user](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/1isomtlvrz2x6ajc8dot.png)

Next, we must configure our AD's default gateway to pfSense.

To open Network Connections settings, you can follow these steps:

1. Press `Win + R` to open the Run dialog.
2. Type **`ncpa.cpl`** and press **Enter**.

This will open the **Network Connections** window, where we can view and manage your network adapters and settings.

Right-click on the adapter **Ethernet 0**, click **Properties**

![adapter properties](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iygl4y0ja4djcmydtp27.png)

Double-click on **Internet Protocol Version 4 (TCP/IPv4)**

![Internet Protocol Version 4 ](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/iwqfk3afjyytomlj0b87.png)

Enter the following configuration, and click **Ok**

![Internet Protocol Version 4 Properties](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/n6bfr7gbu5r26neatcq7.png)

This is the end of the Domain Controller's configuration. You can check [The Cyber Mentor's video](https://www.youtube.com/watch?v=xftEuVQ7kY0) and follow it by this lab.


<h2 id="13"> Configuring Windows Desktops & Onboarding Users Accounts to the AD Domain </h2>

In this lab section, we aim to add 2 Windows desktops to the Domain and complete the AD lab. This portion of the lab is easy to set up, and it will be on [The Cyber Mentor's YouTube guide](https://www.youtube.com/watch?v=xftEuVQ7kY0), which is referenced on the original [Cyberwox's lab](https://cyberwoxacademy.com/building-a-cybersecurity-homelab-for-detection-monitoring/).

It is not a must to add 2 Desktops in this lab, successfully adding one is sufficient.

Ensure you have the [Windows 11 evaluation copy](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise) downloaded.

**Note:** Before proceeding with the Windows Desktop installation, do not start the machine, until: 
 
* Ensure you install on VMWare with the defaults
* Ignore the Product key and simply skip it
* By default, a network adapter is attached when creating the VM, ensure you change the assigned network to **VMnet3**

![Network](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4onfqw9my00c7e6ct0vt.png)

Next, power on the VM to begin the installation

Click **Next**

![windows 11 installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ul8dlojlumq8a75i6g2m.png)

Click **Next**

Make sure **Install Windows 11** is selected

Check **I agree everything will be deleted including files, apps, and settings**

Click **Next**

![installation](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dwrfh4e8unhwbpyd6l62.png)


Click **Accept**

![Accept](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dbkw3zrs7r9ckse0znoh.png)

Click **Next**

![Disk selection](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q9mvwtfyiwqtzkftbah3.png)

Click **Install**

After installation, the VM will restart

Select your preferred language and keyboard

Select **I don't have internet**

Enter the name of the first user we created on our AD (John Doe)

Create a password and follow the wizard through

Once installation is finished, we proceed to  join this PC to our Domain

To join our domain, follow these steps:

1. Press `Win + R` to open the Run dialog.
2. Type **`sysdm.cpl`** and press **Enter**. This opens the **System Properties** window.
3. In the **System Properties** window, make sure you are at the **Computer Name** tab.
4. Click on **Change...** next to "To rename this computer or change its domain..."
5. In the next window, select **Domain** under "Member of," and enter the name of the domain you want to join e.g. **(cybercrex.internal)**.
6. Click **OK** and provide domain credentials when prompted.
7. Restart the computer to complete the process.

![credentials](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/t0cgi8ald7l5y9o5dzn2.png)

![Success](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/au4pibw4vlgkm70u64wo.png)

After the restart, Click **Other User** then sign in with any of the user's credentials we created on our AD

![Login](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ab05hgt7pefxq4j4waj4.png)

Login and complete the onboarding process

Repeat the steps with the other machine, you can try it using Windows 10 too. Download [Windows 10 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise)

<h2 id="14"> Installing and Configuring Splunk </h2>

In this section, we are going to install and configure Splunk. 

In the cybersecurity industry, Splunk is a leading platform for collecting, monitoring, and analyzing security data in real time, enabling rapid threat detection, incident response, and compliance through powerful data insights and automation.

You can learn more about Splunk [here](https://www.splunk.com/en_us/training/course-catalog.html?sort=Newest&filters=filterGroup1FreeCourses)

We will be creating our Splunk instance on a Ubuntu Server VM, so let us download the Ubuntu server image [here](https://ubuntu.com/download/server)

After downloading the image, create a new VM using the Ubuntu Server image. The VM should have the following setup:

- **RAM**: 4GB (4096 MB)
- **Processors**: 2 
- **Hard Disk**: 100GB

You can start the VM to begin the installation

Proceed by accepting the defaults

Use the following settings for **Guided Storage configuration**

![Storage configuration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ezksi6b2hnuiqvjt7ws4.png)


Next set up a profile 

![profile configuration](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j1jhqjfe8bnr8dh0nerq.png)

**Upgrade to Ubuntu Pro**, select **Skip for now**

![Upgrade to ubuntu pro](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9vscmcwe3srjwmuwsrx7.png)


Next, **SSH configuration**, depending on your preference, you can install **OpenSSH server**

Next, proceed with the defaults and reboot when installation is complete.

During reboot, you will asked to unmount the image, simply press the **Enter** key to proceed

After a successful reboot, you should be shown a similar interface as below, simply enter the credentials you used during the Ubuntu installation

![login interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zvpzbktuy37h476390jm.png)

![logged-in interface](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gkrkx3ttl0tlbanpnw1u.png)

For the Splunk server installation, there are two options:

1. Accessing it via an Analyst workstation/VM using SSH
2. Installing a GUI (Ubuntu Desktop) on the Ubuntu Server 

In this lab, I'll be installing a GUI on the Ubuntu Server for this lab using the following steps:

```

# Install tasksel

sudo apt update
sudo apt install tasksel

# Install the Ubuntu desktop GUI but note that there are a variety of desktop flavors to choose from

sudo tasksel install desktop

# Reboot the server
reboot

```

![Ubuntu desktop](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/pvqrmqsbe1ivajtrp6ir.png)

After rebooting, you should have your GUI

![Ubuntu GUI](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w6xdtx388hedx8ucy73a.png)


### Installing Splunk

On the Ubuntu server, open your browser and navigate to [https://splunk.com](splunk.com)

Click on **Free Splunk**

Create an account or log in

![Splunk](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j3yy6szem80aqod42rqm.png)

Under **Products** _>>_ **Free Trials & Downloads** _>>_ **Splunk Enterprise**

Click **Get My Free Trial**

![Get My Free Trial](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qstum5vgbld17r555oqn.png)

Select the **linux** package and download the **.tgz** package

![.tgz package](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ab0n7elu7lwhhmg8la9h.png)

Next, open the terminal and navigate to the **Downloads** directory

![Launch the terminal](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zedsvackj60pb01iyg4f.png)

![Terminal](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wonlq361an9sm0hy9vpa.png)

Next, untar and install Splunk

```
# Untar the download

tar -zxf splunk-*

./splunk/bin/splunk start

```

Enter an administrator username and a password

![admin name](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/u2l8gegmtx0wj1odv0p2.png)

Next, open your browser and navigate to HTTP://splunk:8000

Login with the credentials you created

![Splunk Login](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/srd1ccjdso698hhjigwp.png)


![Splunk Page](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mdfw5b1puvkzi8dxc1c5.png)


 <h2 id="15"> Installing Splunk Universal Forwarder on Windows Server </h3>

One of the processes to accomplish Endpoint Detection and Response (EDR) is to log the activities of our endpoint. To log the activities on our endpoint, Splunk uses a method or agent called the **Universal Forwarder**. The Universal Forwarder can be installed on Linux/Unix, Windows and Mac systems to forward logs to our Splunk instance.

Before proceeding to our Windows Server, Add a new network adapter to the Splunk instance, and ensure you assign the adapter to the **vmnet4** network.

![VMNet](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ymnisvwttdy00w3g4797.png)

After adding the network adapter, open the Splunk dashboard, navigate to **Settings** _>>_ **Forwarding and receiving** _>>_ **Add new** receiving port 

![Settings drop-down](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cwajpkt17vei77gi5jic.png)

![Receiving port](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/89meqtpxpf2xmq0v3171.png)

Enter **9997** and Click **Save**

![Save port](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/v1a2we7kybowzvlv77gs.png)

Navigate to **Settings** _>>_ **Indexes** _>>_ **New Index**

![Settings drop-down](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gl09mk6k86pd4bb75o9l.png)

![Add Index](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/w2ykahi8y41r2a2sfv5y.png)

**Index Name**, enter **wineventlog** and Click **Save**

Next, open your terminal and type

`sudo ip link show`, what we are looking for is the name of our newly attached interface which is currently down. In my own case below, it is **ens7**

![terminal](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x4nuurpo3yn4hhux623g.png)

Next type **sudo ip link set dev <interface> up** e.g. `sudo ip link set dev ens37 up`

Next run this script, replace **ens37** with the appropriate interface

```
sudo tee -a /etc/netplan/01-netcfg.yaml <<EOF

network:
  version: 2
  ethernets:
    ens37:  # Replace with your network interface name
      dhcp4: false
      addresses:
        - 192.168.3.10/24  
      routes:
        - to: default
          via: 192.168.3.1  
      nameservers:
        addresses:
          - 192.168.3.1
          - 8.8.4.4

EOF

sudo netplan apply
```

We can now proceed to the Windows Server, open the browser and download the [Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html)

You may encounter an issue when using Internet Explorer, follow these steps to fix it:

1. Open **Internet Options** by clicking on the gear icon or from the Control Panel.
2. Go to the **Security** tab, select **Internet** zone, then click **Custom level...**.
3. Scroll to **Downloads**, locate **File download**, and select **Enable**.
4. Click **OK** to save the settings, then **Apply** and **OK** to close.

Restart Internet Explorer and try the download again.

I recommend you download a different browser


![Universal Forwarder](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gz25phlhikw9kf2moa4l.png)


After downloading, install it

Accept the License Agreement

![Splunk Universal Forwarder](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7xeyn6ubcs2rkazftbxz.png)

Click **Next**

Create a username and password and Click **Next**

Under the **Deployment Server**,

![Deployment Server](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4xcqxh49yp0nroa3zjwe.png)

In the **Hostname** field, enter **192.168.3.10**, and enter **8089** in the **Port** field

Under the **Receiving Indexer**

![Receiving Indexer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qoufe8rnd47bnja4smwl.png)

In the **Hostname** field, enter **192.168.3.10**, and enter **9997** in the **Port** field


Next, let's proceed to our Splunk Instance's dashboard

Navigate to **Settings** _>>_ **Add Data** _>>_ **Forward**

![Add Data](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hgpnp63abg602zhlk312.png)

![Splunk platform selection](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ix9v5k5wuiilmg9s5x2x.png)

Select a **Server Class** under the **Available hosts(s)** menu, select our Windows Domain Controller, in the **New Server Class Name** field,enter **Domain Controller**

At the top-right corner, Click **Next**

Select **Local Event Logs**, choose your desired event logs

![Desired event logs](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6ngl7sv9cfuh15nx76nq.png)

Click **Next**

Select **wineventlog** (the receiver index we created) as the index

![input settings screen](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/g0rfqqesa594cyg69g6p.png)

Click **Next** and click **Submit**



<h2 id="16"> Ubuntu/CentOS/Metasploitable/DVWA/Vulnhub Machines: Optional machines for exploitation, detection, and monitoring purposes </h2>

We have concluded the lab, note that we can further advance the lab by adding different types of machines for practice.

You have garnered the knowledge and tools you need to do a lot of labs, research, and anything you want to do. Work on detection rules, SIEM content, rule tuning, and attack scenarios to build skills from various angles.

### Important Notes

1. To avoid outrageous costs in the cloud, ensure all VMs are shut down via the Azure console, APIs or CLI when not in use.

2. After shutting down VMs, you will be charged for storage, and public IP resources that are not deprovisioned.

3. Also, you will be charged for the VPN appliance until you deprovision it.

4. You are also charged for egress traffic to the internet and cross-regional communication.
























