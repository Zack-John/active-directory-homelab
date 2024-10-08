<h1 align="center">Active Directory Home Lab</h1>


<h4 align="center"><i>*Note: The scope of this lab has (greatly) expanded to incorporate lessons on networking fundementals, common troubleshooting steps, and more in addition to the original setup guide. As such, certain parts of the lab are still under construction. The core process of creating the lab is done, but many sections will be in a rough draft state and/or incomplete. </br> Thanks for your patience :)</i></h4>

<h2></h2>

The purpose of this lab is to simulate a (very) small-scale enterprise environment and provide a comprehensive, hands-on introduction to the fundementals of networking. There are many tutorials that walk you through the steps to set up a labs similar to this one, but few provide any pertinent information about the _how_ and _why_. I hope this lab will help newcomers understand and contextualize some of these concepts, providing a foundation for future learning. The instructions should work for Windows and MacOS. Finally, I'm assuming that you have very limited or no prior experience using these tools; however, some basic familiarity with using your operating system (installing applications, navigating the file system, etc) is expected.

The majority of time in this lab will be spent configuring a domain controller running Windows Server 2019. In addtion to handling Active Directory domain services, the domain controller will act as the internal network's default gateway and provide critical networking services like a DNS server, a DHCP server, and a remote access server.

In addition to the domain controller, we will also create a Windows 10 Pro client machine and connect it to the network. This machine will be used to emulate the end-user experience and verify that our Active Directory policies, accounts, and networking infrastructure are working as expected.


Upon completion of this lab, you will have:

- Created and configured two virtual machines from ISO images
- Installed drivers and software using a (virtual) installation media (guest additions)
- Assigned a static IP address, subnet mask, and DNS server address to an internal network adapter
- Installed and configured RAS and NAT on Server 2019
- Installed and configured DNS and DHCP on Server 2019
- Created an Active Directory domain, organizational unit (OU), and domain administrator account
- and more!


<h2>Tools and Technologies Covered</h2>

- Virtualization*
- DHCP
- APIPA
- IP Addresses + Subnet Masks
- DNS
- Active Directory Domain Services
- RAS + NAT


<h2>Prerequisites</h2>

- [VirtualBox + Extension Pack](https://www.virtualbox.org/wiki/Downloads)
- [Windows Server 2019 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10)


<h1>Part 1: Virtual Machine Setup</h1>

The first thing we'll need to do is create the virtual machine that will run our Server 2019 operating system.

Click the New button at the top of your VirtualBox window and provide the required information: give the new machine a descriptive name, choose where you'd like the machine to be installed, and show VirtualBox where to find your Server 2019 ISO. After that, check the Skip Unattended Installation box.

<i>Note: Unattended Installation is a feature that allows VirtualBox to automate some of the initial setup of a new operating system for you. While this feature is handy, and the tasks it automates are relatively minor, we are skipping it here because we want to learn to do these things manually.</i>

<p align="center"> <img src="https://i.imgur.com/269sDD0.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

On the next screens, specify the amount of hardware resources you'd like your virtual machine to have access to. Ideally you would be able to allocate at least 2GB (2048 MB) of RAM and two processors to the machine, but if you're on a relatively modest device or you're unsure about what hardware your host machine has, the default settings should suffice.

It is worth noting that the size of the virtual hard disk is the _upper limit_ of storage for the drive, and not how much space it will occupy on your storage device upon creation (unless you select 'Pre allocate Full Size', which you should definitely not do).

<p align="center"> <img src="https://i.imgur.com/KqpOHEZ.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/eDima1A.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

Verify your settings are correct, and then click Finish.

<p align="center"> <img src="https://i.imgur.com/oefdKJL.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

Voilà! You now have a virtual machine that is ready to install a guest OS. But before we do that, there are a few settings we'll need to change. Open the machine settings from the right-click menu:

<p align="center"> <img src="https://i.imgur.com/h8EgzH9.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

The first settings we'll change are technically optional, but are very convenient if we ever need them. Navigate to the Advanced tab of the general settings and change both Shared Clipboard and Drag'n'Drop to 'Bidirectional'. These features allow us to copy/paste things or drag them between our host machine and our guest machine.

<p align="center"> <img src="https://i.imgur.com/7450JvP.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

Next, click on the Network tab. Our domain controller will have two network interfaces: one that connects to the internet through our home network, and one that connects to our internal network (like what you would connect to if you were logging in to a machine at work). The adapter that will connect to the internet is already set up for us under Adapter 1, and will automatically be assigned an IP address from your router. Click on Adapter 2, check the Enable Network Adapter box, and use the 'Attached to' dropdown menu to select Internal Network.

<p align="center"> <img src="https://i.imgur.com/6cbK9KD.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

Click OK to save your changes. We're done with VirtualBox for the time being, and are ready to move inside the virtual machine!


<h1>Part 2: Domain Controller Configuration</h1>

<h2>Server 2019 Installation</h2>

Double-click the virtual machine to boot it. After a few moments, you'll be prompted to choose a few language and location settings.

<p align="center"> <img src="https://i.imgur.com/bwAUHyC.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Once that is done, click on the big Install Now button, and on the next screen <b>be sure to select the Standard Evaluation (Desktop Experience)</b> edition. This version will provide you with a traditional desktop GUI instead of just a command line interface.

<p align="center"> <img src="https://i.imgur.com/hidgoTI.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

On the next screen, select the Custom installation method (since we are installing the OS fresh rather than upgrading an existing OS). Select the virtual disk we created during the VirtualBox setup, and click through until the OS begins to install.

<p align="center"> <img src="https://i.imgur.com/0B3tWNR.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/WaBAjj1.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

The installation will likely take awhile, and the virtual machine will reboot itself once it is done.

<p align="center"> <img src="https://i.imgur.com/WI1AyLb.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Once the installation is done and the machine has rebooted, you'll be prompted to enter a password for the default administrator account.

Enter a password, click Finish, and the system will bring you to the Windows lock screen. Your OS is officially installed!

You may notice that the lock screen says to press Ctrl+Alt+Delete to unlock, but when you enter the command it opens your host machine's Ctrl+Alt+Delete menu instead. This is expected behavior.
You can input Ctrl+Alt+Delete in a VM by either pressing the "host key" (right Ctrl by default) and Delete together, or click the Input menu at the top of the window, hover over Keyboard, and click Insert Ctrl+Alt+Delete.

Using one of these methods, you should now be able to enter your administrator password and log into Windows. Once inside Windows, you should see a prompt appear on the right side of your window asking if you'd like to allow this machine to be discoverable on the network. Click Yes.

<p align="center"> <img src="https://i.imgur.com/heND3fY.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

The first thing I like to when setting up a new machine is to change its name to something more descriptive. This is valuable when you have a large number of devices on a network and don't want to remember a bunch of arbitrary, OS-generated names.

Right click on the Windows menu button (bottom left of your desktop) and click on System. Then click the grey button with the label "Rename this PC". Name it whatever you'd like. I went with "Domain-Controller". Click next and select Restart later.


<h2>Installing Guest Additions</h2>

The last bit of setup for our OS installation is to install the VirtualBox Guest Additions for Windows. Guest Additions are a set of drivers and programs that offer a host of quality of life features, such as improved performance and dynamic window resizing. This step isn't technically required for the lab to work, but it will make your life much easier.

Open the Devices menu at the top of your window and click on "Insert Guest Additions CD image...". This button essentially inserts a virtual CD-ROM into a virtual CD drive on our virtual machine (virtually!).

<p align="center"> <img src="https://i.imgur.com/qORYzQp.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Once the "CD" has been "inserted", navigate to My Computer and click on the CD drive to open its contents. You should see a list of files and applications. Click the application called "VBoxWindowsAdditions-<b>amd64</b>" to run the installer. Once the installer has run, reboot the machine.

<p align="center"> <img src="https://i.imgur.com/e9MjwU5.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

After rebooting, your machine should now have the additions installed. The easiest way to verify the install worked is simply resizing your window. The OS should automatically adjust the display resolution to fit the new size. If it doesn't, you can run the installer again and this time select "I will manually reboot later" at the last step, and then shut down and restart the virtual machine.

With the Guest Additions installed, our installation is set up and we are finally ready to start diving into our domain controller configuration! The first step is to set up our network interfaces on this machine.


<h2>Configuring Network Interfaces</h2>

Click on the network icon in your system tray (the little icon that looks like a monitor and an ethernet cable at the bottom-right of your screen), and then click on Unidentified network (it may also just say "Network"). You should see a menu with two ethernet adapters listed. We want to click on "Change adapter options" under Related settings. This will bring you to the Network Connections screen where you should see two different ethernet connections.

<p align="center"> <img src="https://i.imgur.com/jhnSuZ0.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Recall that we set up the virtual machine to have two different network interfaces: one that will connect to the internet through your home router, and one that will connect to the (very small) internal network we're building. The "internet" adapter will automatically be assigned an IP address by your router via [DHCP](https://www.youtube.com/watch?v=ldtUSSZJCGg), which allows the VM to act like any other device on your home network. The "internal" adapter will need to be configured manually.

First, we need to figure out which adapter is which. Double click on either of the adapters and click the "Details..." button on the status menu that appears.

<p align="center"> <img src="https://i.imgur.com/Xs9jEcN.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

This will open a detailed breakdown of that adapters network configuration. If the adapter has an IP address that starts with 169.254, then it is your internal adapter. Otherwise, the adapter will have a "normal" IP address (usually something like 10.0.2.15) and most of the fields (default gateway, DNS server, lease times, etc) will be filled out; this is your external adapter.

<p align="center">
  <img src="https://i.imgur.com/Ie0InIi.png" height="80%" width="80%" alt="Server 2019 Setup"/></br>
  <i>internal adapter - notice the 169.254.x.x IP address and empty data fields</i>
</p>

<p align="center">
  <img src="https://i.imgur.com/MvhUxEz.png" height="80%" width="80%" alt="Server 2019 Setup"/></br>
  <i>external adapter - notice the standard IP address and "Lease obtained" / "Lease expires" fields with information in them</i>
</p>



**__________________**

**Aside - APIPA Addressing**

Automatic Private IP Addressing (APIPA) is a feature of Windows that allows a device to assign itself an IP address (sometimes called a link-local address) if it does not receive one from a DHCP server (i.e., your router). APIPA always uses the IP address range of 169.254.0.1 - 169.254.255.254, so it is easy to tell at a glance if a machine failed to received an IP address from your DHCP server / router. With a link-local address, a device can still communicate on the local network it is connected to, but is unable to reach the internet.

<i>So why does our internal adapter have a link-local address but our external adapter have a DHCP address?</i>

Our external adapter can communicate with our router, so our router was able to provide it with an IP address just like any other "regular" device in your home. Our internal adapter is _only connected to our internal VirtualBox network_, which only has our single virtual machine on it at the moment. Since there is no DHCP server on our internal network, the adapter was unable to lease an IP address and therefore defaulted to assigning itself one via APIPA.

**__________________**


Once you've identified the internal and external adapters, I recommend changing the names (right-click --> rename) of the adapters to something like "internal" and "external" so you don't have to check the IP address each time you're changing adapter configurations.

Okay, lets assign a static IP address to our internal adapter so our clients know where to find this machine.

**__________________**

**Aside - IP Addresses, Subnet Masks, and DNS**

IP addressing is a deep topic with a lot of nuances, but the long and short of it is this: an IP address is a unique address assigned to a particular machine. Just like a street address on a house, an IP address tells other devices on a network where to find a particular machine. Networking devices use IP addresses to route data to and from that machine across the network.

A subnet mask essentially defines the range of addresses within a particular network that a machine can talk to. You will often see a subnet mask of 255.255.255.0, which means that a machine on this network can communicate with any other machine on the network that it shares the first three octets of its IP address with. For example, a machine with an IP address of 192.168.10.10 and a subnet mask of 255.255.255.0 can communicate with any other machine that has an IP address of format 192.168.10.x.

If this is all really confusing, don't worry! The specifics of IP addresses are not important for what we'll be doing in this lab, and you can accomplish a lot with only a basic understanding of networking and a few Google searches.

**__________________**


Right click on the internal adapter and select Properties. From the properties list, double-click on Internet Protocol Version 4 (TCP/IPv4).

<p align="center"> <img src="https://i.imgur.com/CUVioLA.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

On the IPv4 properties screen, click the "Use the following IP address" radio button and enter an IP address of <b>172.16.0.1</b> and a subnet mask of <b>255.255.255.0</b>.

<p align="center"> <img src="https://i.imgur.com/w9iHQQC.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

We can leave the default gateway field empty because this machine is going to be playing the role of default gateway itself. Simply put, a default gateway is the router on your network that will be connected to the internet. We are going to have our client machine(s) connect to our domain controller through the internal adapter we are currently configuring, and then our domain controller will route the traffic through to the external adapter that is connected to our home router. That way, the client machine(s) will have internet access without ever connecting to your home network. Pretty neat stuff!

As for the DNS servers, we are going to use the address <b>127.0.0.1</b>. This is a special IP address known as a loopback address, also known as the "localhost". This IP address is a way for a machine to essentially route network traffic to itself, and has the same behavior as entering this machines IP address (172.16.0.1) into this field.

Why would we use a loopback address for our DNS server you might ask? Active Directory Domain Services (ADDS) utilizes DNS to route traffic to and from a domain controller. So, when we install Active Directory in a few minutes, we can have this machine act as our local DNS server since it is already acting as our default gateway!

We will leave the alternate DNS server address empty for now. The alternative address acts as a fallback server in case the primary server is unable to resolve your DNS request for some reason. It is typically recommended to have an alternate server, but we'll leave it blank for now as a rudimentary way to test our domain controller's DNS functionality later; if it doesn't work with just the DC's address, but does work with an known-good address as an alternate, we can conclude that our DNS service is failing.

Double check all of your addresses are correct and click OK.

<p align="center"> <img src="https://i.imgur.com/GyUIx77.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

<h1>Part 3: Active Directory Domain Services (ADDS)</h1>

<h2>Installing ADDS</h2>

When you first booted Windows, it should have opened the Server Manager application. If you exited out of the app, launch it by clicking on the Windows menu button and locating it in the list of apps or searching for it by name.

On the Server Manager dashboard, find the button that says "Add roles and features" and click it.

<p align="center"> <img src="https://i.imgur.com/RPGkoNC.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

This will open the role installation wizard (which is basically just a nerdy name for a step-by-step guide). As we move through the steps of the installation wizard, you're going to see a _lot_ of different roles and features that can be installed. Feel free to poke around and read about any of the roles or features in these menus to get an idea of what they might be used for, but for now lets stick with the default selections unless otherwise mentioned.

Click through the next three pages, making sure the "Role-based or feature-based installation" button is checked on the second page, and that your server is highlighted on the third page.

<p align="center"> <img src="https://i.imgur.com/VUzWVvY.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/3kZqYZ2.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

You should now be on the list of roles that you can install on this server. Click the checkbox next to Active Directory Domain Services near the top of the list. This will open another window that lists the packages that will be installed.

<p align="center"> <img src="https://i.imgur.com/g2zZcDH.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Click the Add Features button to continue. You should be returned to the "Select server roles" screen with the box next to Active Directory Domain Services now checked.

<p align="center"> <img src="https://i.imgur.com/2onBlJm.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/qkpQ6sE.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Use the Next button to move through the next two screens ("Features" and "AD DS"). Make sure your confirmation page lists the same tools as the one pictured below and click the Install button.

<p align="center"> <img src="https://i.imgur.com/gfWLXzR.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Once the installation is done, close the installer if you haven't already.

<h2>Creating a Domain</h2>

After installing ADDS, you may have noticed a yellow "caution symbol" appear in the top right of the Server Manager window. This appeared because, although we have installed the ADDS software, we still need to configure it! The first step in setting up ADDS is to create a domain.

**What exactly _is_ a domain?**
**TODO**

Start by clicking on that flag icon with the caution symbol, then clicking "Promote this server to a domain controller".

In the deployment configuration menu, click on the radio button next to "Add a new forest". A "forest" is the name we use to describe the root-level (or top-level) structure of an organization's Active Directory domains. In other words, an organization can have multiple domains called "trees", and those trees together make up the "forest".

You can name your forest pretty much anything you want as long as it looks like a regular URL you'd enter in your web browser. I went with "zacksdomain.com". Click Next once you've entered a name.

<p align="center"> <img src="https://i.imgur.com/wRu2bi8.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Similar to when we were installing the ADDS role a moment ago, we are going to leave the rest of the settings in this wizard as the defaults. Read each screen to get an idea of what the wizard is doing here, but don't change anything and click Next through each screen until you are able to click Install. This installation will likely take a few minutes. The machine will restart itself once the install is complete.

<h2>Creating an Organizational Unit and Domain Admin Account(OU)</h2>

Once the machine has restarted, we have officially installed Active Directory and created our very own domain! You may notice when logging in this time that the account name now shows "YOURDOMAIN\administrator". You can still log in as normal, but our next steps will be to create an Organizational Unit within our domain, and our own personal administrator account within that OU.

From the desktop, open the start menu. You should see a new folder called "Windows Administrative Tools". Open that folder and click on "Active Directory Users and Computers".

<p align="center"> <img src="https://i.imgur.com/ZXLC5UP.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

You should see your domain in the left pane of this menu. Right click on the domain, hover over New, and select Organizational Unit from the submenu.

**What is an OU?**
**Coming soon!**

Name your OU "Administrators" and click OK. Your new OU will now appear under your domain on left side of the window.

Right click on the OU, go to New, and select User. Fill this form out with your information. Organizations will typically have a naming conventions for user accounts, and some way to differentiate admin accounts from standard accounts. User accounts are usually something along the lines of first initial and last name (ex: jsmith). Admin accounts will often have a prefix like "admin-" or "a-" in front of the user account (ex: a-jsmith). Once you've filled out this screen, click the Next button and enter a password for this account and uncheck the "User must change password and next logon" option. For the lab, I suggest also checking the "Password never expires" option for simplicity, but that is _not_ typical in an actual production environment. Once you've entered the password, click Next and then Finish. Your account will now appear inside the OU.

<p align="center"> <img src="https://i.imgur.com/rFwjUoP.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Now we need to actually promote the account to an admin account. Right click on the account and select Properties. In the properties menu, click the "Member Of" tab, and click "Add". A new window will pop up asking us to choose a group to add the account to. In the text box, enter "Domain Admins" and click the Check Names button to the right. If entered correctly, "Domain Admins" will be underlined. Click OK. The "Member Of" tab will now show the account as belonging to the domain admins group in addition to the domain users group. Click Apply and OK at the bottom of the Properties window.

<p align="center"> <img src="https://i.imgur.com/9EMOiSr.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Lets take that new admin account for a spin! Sign out of the generic admin account and on the login screen select "Other user". Log in with your new credentials.

<p align="center"> <img src="https://i.imgur.com/qKpHOUB.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Alright administrator, lets get adminstering!

<h2>Installing RAS / NAT</h2>

**WHAT IS NAT?**
**Coming soon!**

NAT "allows our internal clients to connect to the internet using one public IP address."

**WHAT IS RAS? WHATS IT FOR?**
**Coming soon!**

Currently, our domain controller can connect to the internet through our external network adapter. Eventually we will want our clients to have internet access from within our private network. To do that, we'll need our domain controller to route traffic from the external network to the internal network. That functionality is provided through NAT and a remote access service, which can all be installed with the "routing" role.

The installation process for the routing role is very similar to the process we used to install ADDS:

Open a Server Manager instance and click on "Add roles or features", make sure "Role-based or feature-based installation" is selected on the installation type screen, select the domain controller on the server selection screen (it will be the only one on the list), and click the check box next to "Remote Access" on the server roles screen, and leave everything default on the features screen.

<p align="center"> <img src="https://i.imgur.com/8MQj83q.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

After the features screen, you'll be on a page that explains what the remote access role provides a capable young administrator like yourself. Go ahead and read this page, but don't worry if it's not quite making sense -- it's a pretty jargin-heavy screen. Click Next once you're done, and click the box next to "Routing" on the role services screen. Click Add Features on the pop-up prompt. The wizard will automatically check "DirectAccess and VPN (RAS)" in addition to Routing.

<p align="center"> <img src="https://i.imgur.com/sdQBX69.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/hdm6eGd.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Click through the next few screens keeping default selected, and then click Install. Once the installation completes, I strongly recommend restarting the machine. The NAT installation we are about to do seems to have issues when I go straight from installation to setup.

<h2>Configuring RAS / NAT</h2>

Now that we have all of our routing features installed we need to configure them. Click on the Tools button in the top-right of the Server Manager window and select "Routing and Remote Access" from the dropdown menu.

<p align="center"> <img src="https://i.imgur.com/pNQSscL.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

In the new window that appears, you should see your domain controller in the left pane. Right click it and select "Configure and Enable Routing and Remote Access".

<p align="center"> <img src="https://i.imgur.com/Ni1feUB.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Click the radio button next to "Network address translation (NAT)" and click Next.

<p align="center"> <img src="https://i.imgur.com/M4pLmUJ.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

On the next screen, the button next to "Use this public interface to connnect to the Internet" should be selected automatically, and you should see a list of your network interfaces. If that is the case, select the external interface from the menu and click Next and then Finish. **If the top option is greyed out and you can only select the "Create a new demand-diale interface to the Internet, exit the wizard and restart the machine.** Once the machine restarts, you should be able to select the external interface from the list and continue as normal.

Your screen should look like this:

<p align="center"> <img src="https://i.imgur.com/8HYK7Tg.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Not this:

<p align="center"> <img src="https://i.imgur.com/fgQ5ThC.png" width="80%" alt="Server 2019 Setup"/> </p>

Once the wizard is done setting up NAT, our domain controller should have the ability to route our internet traffic for our clients! Our next step is to set up DHCP so that our clients can get an IP address and actually take advantage of these routing features.

<h2>DHCP Installation and Configuration</h2>

To set up DHCP, we will once again use the "Add roles and features" button on the Server Manager dashboard. This process is more-or-less the same as the first two roles we installed. Use the role-based installation type, select our domain controller, and select "DHCP" server from the server roles list. 

<p align="center"> <img src="https://i.imgur.com/CCqHDJz.png" width="80%" alt="Server 2019 Setup"/> </p>

Leave all of the default features selected, read the DHCP role explanation, and install the role.

Once that is done, click on the Server Manager's Tools menu in the top right and select "DHCP" to open the DHCP configuration panel.

<p align="center"> <img src="https://i.imgur.com/b28eB3M.png" width="80%" alt="Server 2019 Setup"/> </p>

As a reminder, DHCP automatically assigns machines on our network an IP address so that they can use the internet. DHCP servers assign addresses from a specified range called a "scope". Our scope for this lab will be the addresses from 172.16.0.100 through 172.16.0.200.

You should see your domain controller in the left column of the DHCP panel. Select it to reveal two subcategories: IPv4 and IPv6.

<p align="center"> <img src="https://i.imgur.com/jhPC0XG.png" width="80%" alt="Server 2019 Setup"/> </p>

We are only going to configure IPv4 addresses for now, but the process of configuring IPv6 is essentially the same. Click on IPv4 to target it, then right click on IPv4 and select "New Scope..." to open the wizard.

<p align="center"> <img src="https://i.imgur.com/5UvHNOG.png" width="80%" alt="Server 2019 Setup"/> </p>

You would typically assign a name to a scope that explains what the scope belongs to, whether it be the DHCP server that is providing the scope, which department will be assigned to said scope, the range of addresses itself, etc. For this lab, I'm just going to name it the range of addresses ("176.16.0.100-200").

Once you've named the scope, click Next and assign the start and end IP addresses. Increase the length of the mask to 24 and it should automatically change the mask to 255.255.255.0. Once that is done, click Next.

<p align="center"> <img src="https://i.imgur.com/H5Xj7Gf.png" width="80%" alt="Server 2019 Setup"/> </p>

The following screen is where you would apply any address exclusions and delays, which would be addresses within our scope that we _don't_ want to give out to client machines. Since we don't have any addresses we want to exclude, click Next to move on to the lease duration screen. The lease time describes how long a machine is allowed to have an IP address before it has to reach back out to the DHCP server and ask for a lease renewal. For a public network such as a coffee shop's wifi, where you will have many different devices coming and going for brief periods of time, you would typically apply a shorter lease duration. Lets leave the lease duration as the default (eight days) for now.

Once you've moved through those two screens, you will be asked if you want to configure the DHCP options for this scope. Make sure the "Yes" option is selected and click Next.

The first piece of information we need to provide is the IP address of our default gateway. In case you forgot, this machine's "internal" network interface _is_ our default gateway, meaning we need to enter that adapter's IP address (172.16.0.1). Type that address into the IP address text field and click the Add button. The address will appear in the list directly below the text field, meaning you can now select it as the gateway.

<p align="center"> <img src="https://i.imgur.com/6e4SOKb.png" width="80%" alt="Server 2019 Setup"/> </p>

The next screen is where we specify our DNS server, which is also this machine (recall: when we added ADDS to this machine, it also added DNS capabilities). It may automatically populate the IP address in the list below. If it doesn't, enter it and click Add like we did on the last page. Then select it and click Next.

<p align="center"> <img src="https://i.imgur.com/TAMMUx5.png" width="80%" alt="Server 2019 Setup"/> </p>

The next screen asking about WINS servers is not applicable to these machines, so you can leave the page black and click Next. Finally, select "Yes, I want to activate this scope now" and click Next, then click Finish.

You should now be able to see the scope we just made under the IPv4 section of the DHCP panel.

<p align="center"> <img src="https://i.imgur.com/lPMvhXa.png" width="80%" alt="Server 2019 Setup"/> </p>

Right click on the domain controller itself and select "Authorize", then right click it again and select "Refresh". You should now see the icons next to IPv4 and IPv6 turn green. This means our DHCP server is good to go!

<p align="center"> <img src="https://i.imgur.com/BQAsiFF.png" width="80%" alt="Server 2019 Setup"/> </p>

At this point, the domain controller should be completely configured to provide networking and Active Directory services to our clients. But before we spin up the client machine to test these features, we will need some user accounts to log in to.

<h2>Creating Active Directory User Accounts</h2>

The process for creating a new "regular" user is pretty much identical to the process for creating an admin account. Navigate to... **section currently being revised!**


<h1>Part 3: Client Machine Configuration</h1>

<h2>Creating the Client Virtual Machine</h2>

Just like when we created the virtual machine for the domain controller, we need to create another machine for our client computer. Go back to VirtualBox and click the New button. Give it a descriptive name (I went with "Client (Windows 10)" to keep naming consistent between my DC and this machine). Browse to the folder you want this machine to live in and navigate to your ISO image in the dropdown menus below. Finally, check the Skip Unattended Installation box.

<p align="center"> <img src="https://i.imgur.com/1KGjkOR.png" width="80%" alt="Server 2019 Setup"/> </p>

You will be running both the DC and the client machine at the same time, so if you are short on hardware resources, consider giving the client less memory and fewer processors than your DC. The size of your virtual hard disk shouldn't make much of a difference as long as you make sure not to select the "Pre-allocate Full Size" option. Verify your settings are correct on the last page, and click Finish.

<p align="center"> <img src="https://i.imgur.com/OOEgnW0.png" width="80%" alt="Server 2019 Setup"/> </p>

Once the machine is created, go into the machine settings, navigate to the Network tab, and change the "Attached to" option on Adapter 1 from NAT to Internal Network. Remember that we want this machine to be connected solely to the internal network, so make sure not to enable any other adapters in the Network menu. You can also activate the bidirectional copy and drag'n'drop features like we did on the DC while you're in the settings menu (General -> Advanced).

<p align="center"> <img src="https://i.imgur.com/OOEgnW0.png" width="80%" alt="Server 2019 Setup"/> </p>

Once that's done, go ahead and double click the machine to launch it.

You'll be greeted with the Windows setup screen; very similar to the one we used when we first set up the Server 2019 installation. Select your language settings and click Next, then Install now.

On the following screen you'll be asked to enter a Windows product key in order to activate your copy of Windows. We are just using the evaluation (aka free trial) version of Windows, so click "I don't have a product key" at the bottom of the screen.

<p align="center"> <img src="https://i.imgur.com/6iUlodH.png" width="80%" alt="Server 2019 Setup"/> </p>

A new menu will appear where you're able to choose which edition of Windows you want to install. **Make sure you select Windows 10 Pro!** Windows Home cannot join a domain, so this is a critical step.

<p align="center"> <img src="https://i.imgur.com/TmYMF2U.png" width="80%" alt="Server 2019 Setup"/> </p>

After that, accept the terms of service, select the Custom install option, verify the virtual disk you created during the VirtualBox setup is selected, and let Windows install itself.

<p align="center"> <img src="https://i.imgur.com/cs5YAxS.png" width="80%" alt="Server 2019 Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/IEY8bqG.png" width="80%" alt="Server 2019 Setup"/> </p>

Once Windows is installed, it will take you through the first-time setup process. Select your region and keyboard layout(s).

On the screen that prompts you to connect to a network, click on the option in the bottom left of the screen that says "I don't have internet". We don't want to connect Windows with a Microsoft account right now, so click "Continue with limited setup" on the following screen to continue with an offline Windows installation.

<p align="center"> <img src="https://i.imgur.com/6YuivNg.png" width="80%" alt="Server 2019 Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/MbkD0UO.png" width="80%" alt="Server 2019 Setup"/> </p>

When asked who is going to use this machine, just enter "User". Windows is creating a local (this machine only) account here, and we won't really be using this account. We are going to be logging into a domain account that we created earlier on the DC. Then you can either create a password for the local account or simply leave it blank and click next on the following screen.

<p align="center"> <img src="https://i.imgur.com/TuR6c2Y.png" width="80%" alt="Server 2019 Setup"/> </p>

The privacy settings don't necessarily matter on this machine, but I still uncheck all of the options like I would on any other machine. Click Accept once you're done.

<p align="center"> <img src="https://i.imgur.com/UzKUIJh.png" width="80%" alt="Server 2019 Setup"/> </p>

Finally, click "Not now" on the Cortana screen. After a few moments, your Windows 10 install setup is complete!

<p align="center"> <img src="https://i.imgur.com/cTtUPQV.png" width="80%" alt="Server 2019 Setup"/> </p>

If you get a prompt asking if you'd like to make this machine discoverable to your domain, click "Yes".

Your machine should get its IP address automatically from the DHCP server, and you should be able to browse the internet on your client machine. We have one last thing to do on our client machine before it can be considered complete: join the Active Directory domain.

When setting up a new client machine, you will also want to rename it to something more descriptive than the default, randomly generated name (just like we did for the domain controller). Luckily, Windows has a method to accomplish both of these goals at once!

Right click your start menu and select System. Instead of clicking the "Rename this PC" button like we did on the DC, scroll down until you see another option called "Rename this pc (advanced)".

<p align="center"> <img src="https://i.imgur.com/eDb47Oe.png" width="80%" alt="Server 2019 Setup"/> </p>

In the system properties window that appears, locate the text that says "To rename this computer or change its domain or workgroup, click Change" and do exactly that: click the "Change" button next to it.

Change the computer name to something like "Client". Then, locate the textbox that says "Domain" under the "Member of" heading below that. Click the radio button next to "Domain", enter your domain name, and click OK.

<p align="center"> <img src="https://i.imgur.com/1SZtM3r.png" width="80%" alt="Server 2019 Setup"/> </p>

You'll be prompted to enter a username and password. Use your domain administrator credentials we created on the domain controller. Once you've entered your credentials successfully, a confirmation message will appear on your screen.

<p align="center"> <img src="https://i.imgur.com/J5v2VhZ.png" width="80%" alt="Server 2019 Setup"/> </p>

According to the prompt, we need to restart before the changes will be completed. Click OK through all of the prompts and menus that are open and click "Restart Now" on the prompt when given the option.

<p align="center"> <img src="https://i.imgur.com/E5rPWqf.png" width="80%" alt="Server 2019 Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/9toaWzX.png" width="80%" alt="Server 2019 Setup"/> </p>

Once your machine has restarted, you will now be able to log into any of the domain accounts you created during the Active Directory setup. To do so, click on "Other user" on the login screen:

<p align="center"> <img src="https://i.imgur.com/db8Cjlq.png" width="80%" alt="Server 2019 Setup"/> </p>

If you go back to your domain controller and open the DHCP panel (under the tools menu in Server Manager), you should be able to see the lease information for the client you created and joined to the domain. You can also view the client machine in the list of computers found in the Active Directory Users and Computers menu (along with all of the user accounts created).

Congratulations, you've now got a working home lab environment with networking infrastructure and Active Directory! This is an excellent jumping off point for the world of enterprise IT. From here, you can study and implement common administrator tasks like account and password management/policy, static IP address assignments, etc.


<h1>Troubleshooting</h1>

This is a complicated process we've just gone through, and the likelihood of something going wrong is probably pretty high. In this section I'll go through a few of the basic troubleshooting steps you can take in order to (hopefully) solve any issues you may have run into over the course of the lab. Please understand that troubleshooting and learning to "get good" at google are absolutely critical skills when it comes to the world of IT, so don't feel discouraged if you aren't able to solve your problem with the information provided below. Use it as a challenge and an opportunity to develop your understanding of the environment you've built and hone your troubleshooting skills!

**Have you tried turning it off and back on?**

Yes, seriously. We did a lot of different things that may require a restart in order to work. Different settings we changed, roles/features we installed, etc. can sometimes get hung up and will sort themselves out after a quick reboot. There is a reason every IT professional starts here!

Also, if you're having connectivity issues on your client, make sure your DC is up and running. The DC performs all of the networking functionality, so it must be completely booted up and running in order for your client to work.

**More coming soon!**













