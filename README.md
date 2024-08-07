<h1>Active Directory Home Lab</h1>

The purpose of this lab is to simulate a small-scale enterprise environment. It is written assuming that you have very little or no prior experience using these tools. I also assume you are using a Windows computer, but should work for MacOS as well.
The majority of time in this lab will be spent configuring a domain controller running Windows Server 2019. The domain controller will fill a number of roles, such as a DNS server, a remote access server, and handling Active Directory Domain Services.
In addition to the domain controller, we will also create a Windows 10 Pro client machine to emulate the end-user experience and verify that our domain accounts and Active Directory policies are working as expected.


Upon completion of this lab, you will have:

- Created and configured two virtual machines from ISO images
- Assigned static IP addresses and DNS server addresses to the machine's network interfaces
- Set up Active Directory admin account(s)
- 


<h2>Prerequisites</h2>

- [VirtualBox + Extension Pack](https://www.virtualbox.org/wiki/Downloads)
- [Windows Server 2019 ISO](https://www.microsoft.com/en-us/evalcenter/download-windows-server-2019)
- [Windows 10 ISO](https://www.microsoft.com/en-us/software-download/windows10)

<h1>Part 1: Virtual Machine Setup</h1>

The first thing we'll need to do is create the virtual machine that will run our Server 2019 operating system.

Click the New button at the top of your VirtualBox window and provide the required information: give the new machine a descriptive name, choose where you'd like the machine to be installed, and show VirtualBox where to find your Server 2019 ISO. After that, check the 'Skip Unattended Installation box.

<i>Note: Unattended Installation is a feature that allows VirtualBox to automate some of the initial setup of a new operating system for you. While this feature is handy, and the tasks it automates are relatively minor, we are skipping it here because we want to learn to do these things manually.</i>

<p align="center"> <img src="https://i.imgur.com/269sDD0.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

On the next screens, specify the amount of hardware resources you'd like your virtual machine to have access to. Ideally you would be able to allocate at least 2GB (2048 MB) of RAM and two processing cores to the machine, but if you're on a relatively modest device or you're unsure about what hardware your host machine has, the default settings should suffice.

It is worth noting that the size of the virtual hard disk is the _upper limit_ of storage for the drive, and not how much space it will occupy on your storage device upon creation (unless you select Pre allocate Full Size, which I would advise against).

<p align="center"> <img src="https://i.imgur.com/KqpOHEZ.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

<p align="center"> <img src="https://i.imgur.com/eDima1A.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

Verify your settings are correct, and then click Finish.

<p align="center"> <img src="https://i.imgur.com/oefdKJL.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

Voilà! You now have a virtual machine that is ready to install a guest OS. But before we do that, there are a few settings we'll need to change. Open the machine settings from the right-click menu:

<p align="center"> <img src="https://i.imgur.com/h8EgzH9.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

The first settings we'll change are technically optional, but are very convenient if we ever need them. Navigate to the Advanced tab of the general settings and change both Shared Clipboard and Drag'n'Drop to Bidirectional. These features allow us to copy/paste things or drag them between our host machine and our guest machine.

<p align="center"> <img src="https://i.imgur.com/7450JvP.png" height="80%" width="80%" alt="VirtualBox Setup"/> </p>

Next, click on the Network tab. Our domain controller will have two network interfaces: one that connects to the internet through our home network, and one that connects to our internal network, like what you would connect to if you were logging in to a machine at work. The adapter that will connect to the internet is already set up for us under Adapter 1, and will automatically be assigned an IP address from your router. Click on Adapter 2, check the Enable Network Adapter box, and use the Attached to dropdown menu to select Internal Network.

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
The way to enter Ctrl+Alt+Delete in a VirtualBox VM is to either press the "host key" (right Ctrl by default) and Delete, or to click the Input menu at the top of the window, then Keyboard, then Insert Ctrl+Alt+Delete.

Using this method, you should now be able to enter your administrator password and log into Windows. Once inside Windows, you should see a prompt appear on the right side of your window asking if you'd like to allow this machine to be discoverable on the network. Click Yes.

<p align="center"> <img src="https://i.imgur.com/heND3fY.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

The last bit of setup for our OS installation is to install the VirtualBox Guest Additions for Windows. Guest Additions improves the functionality of Windows virtual machines and offers a host of quality of life features, such as dynamic window resizing. This step isn't technically required for the lab to work, but it will make your life much easier.

Open the Devices menu at the top of your window and click on "Insert Guest Additions CD image..." This button essentially inserts a virtual CD-ROM into a virtual CD drive on our virtual machine (virtually!).

<p align="center"> <img src="https://i.imgur.com/qORYzQp.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

Once the "CD" has been "inserted", navigate to My Computer and click on the CD drive to open its contents. You should see a list of files and applications. Click the application called "VBoxWindowsAdditions-<b>amd64</b>" to run the installer. Once the installer has run, reboot the machine.

<p align="center"> <img src="https://i.imgur.com/e9MjwU5.png" height="80%" width="80%" alt="Server 2019 Setup"/> </p>

After rebooting, your machine should now have the additions installed. You can verify the install worked by resizing your window. The OS should automatically adjust the display resolution to fit the size of the window. If it doesn't, you can run the installer again and this time select "I will manually reboot later" at the last step, and then shut down and restart the virtual machine.

With the Guest Additions installed, we are finally ready to start diving into our domain controller configuration! The first step is to set up our network interfaces on this machine.

