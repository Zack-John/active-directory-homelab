<h1>Active Directory Home Lab</h1>

The purpose of this lab is to simulate a small-scale enterprise environment. There are many tutorials that walk you through the steps to set up an Active Directory lab at home, so I'm trying to provide a little bit more information about the _how_ and _why_ with this guide. The instructions should work for Windows or MacOS, and I'm assuming that you have very limited or no prior experience using these tools.

The majority of time in this lab will be spent configuring a domain controller running Windows Server 2019. The domain controller will fill a number of roles, such as a DNS server, a remote access server, and handling Active Directory Domain Services.
In addition to the domain controller, we will also create a Windows 10 Pro client machine to emulate the end-user experience and verify that our domain accounts and Active Directory policies are working as expected.


Upon completion of this lab, you will have:

- Created and configured two virtual machines from ISO images
- Installed drivers and software using an installation media (guest additions)
- Assigned static IP addresses and DNS server addresses to the machine's network interfaces
- ...


<h2>Technologies Covered</h2>

- _**Virtualization**_ (havent written explanation yet; will do in revision)
- DHCP
- APIPA
- _**DNS**_ (havent reached this point yet)
- ...


<h2>Prerequisites</h2>

- [VirtualBox + Extension Pack](https://www.virtualbox.org/wiki/Downloads)
- [Windows Server 2019 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10)

<h1>Part 1: Virtual Machine Setup</h1>

The first thing we'll need to do is create the virtual machine that will run our Server 2019 operating system.

Click the New button at the top of your VirtualBox window and provide the required information: give the new machine a descriptive name, choose where you'd like the machine to be installed, and show VirtualBox where to find your Server 2019 ISO. After that, check the Skip Unattended Installation box.

<i>Note: Unattended Installation is a feature that allows VirtualBox to automate some of the initial setup of a new operating system for you. While this feature is handy, and the tasks it automates are relatively minor, we are skipping it here because we want to learn to do these things manually.</i>

<p align="center"> <img src="https://i.imgur.com/269sDD0.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

On the next screens, specify the amount of hardware resources you'd like your virtual machine to have access to. Ideally you would be able to allocate at least 2GB (2048 MB) of RAM and two processing cores to the machine, but if you're on a relatively modest device or you're unsure about what hardware your host machine has, the default settings should suffice.

It is worth noting that the size of the virtual hard disk is the _upper limit_ of storage for the drive, and not how much space it will occupy on your storage device upon creation (unless you select 'Pre allocate Full Size', which I would advise against).

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

<h2>Server 2019 Operating System Installation</h2>

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

This will open a detailed breakdown of that adapters network configuration. If the adapter has a "normal" IP address (usually something like 10.0.2.15) and most of the fields (default gateway, DNS server, etc) filled out, then it is your external adapter. If the adapter has many blank fields and an IP address that starts with 169.254, then it is your internal adapter.

<p align="center">
  <img src="https://i.imgur.com/MvhUxEz.png" height="80%" width="80%" alt="Server 2019 Setup"/></br>
  <i>*external adapter - notice the "Lease obtained" and "Lease expires" fields with information in them</i>
</p>

<p align="center">
  <img src="https://i.imgur.com/Ie0InIi.png" height="80%" width="80%" alt="Server 2019 Setup"/></br>
  <i>*internal adapter - notice the empty fields, no lease info, and the 169.254.x.x IP address</i>
</p>


**__________________**

**Aside - APIPA Addressing:**

Automatic Private IP Addressing (APIPA) is a feature of Windows that allows a device to assign itself an IP address (sometimes called a link-local address) if it does not receive one from a DHCP server (i.e., your router). APIPA always uses the IP address range of 169.254.0.1 - 169.254.255.254, so it is easy to tell at a glance if a machine failed to received an IP address from your DHCP server / router. With a link-local address, a device can still communicate on the local network it is connected to, but is unable to reach the internet.

<i>So why does our internal adapter have a link-local address but our external adapter have a DHCP address?</i>

Our external adapter can communicate with our router, so our router was able to provide it with an IP address just like any other "regular" device in your home. Our internal adapter is _only connected to our internal VirtualBox network_, which only has our single virtual machine on it at the moment. Since there is no DHCP server on our internal network, the adapter was unable to lease an IP address and therefore defaulted to assigning itself one via APIPA.

**__________________**



























