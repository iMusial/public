# How to Setup a Basic  Network with a Firewall in VMware Workstation 
This guide assumes you already have VMware Workstation installed.

## Requirements:
### Operating System
Windows or Linux is preferred, this guide will not detail steps for MacOS.
### Recommended Hardware
* 16 GB of RAM
* 8 Core CPU
* at least 100 GB of available storage

It's worth noting that hardware requirements can be tweaked a little within VMware Workstation, at the potential cost of performance. I will also detail specific options during the step-by-step in case it's necessary.

### Files
* pfSense 2.6.0 is used in this lab, make sure to use a trusted/official source when downloading pfSense.
* Debian 13 AMD64 offline installer is available on Debian's official website: https://cdimage.debian.org/debian-cd/current/amd64/iso-dvd/debian-13.5.0-amd64-DVD-1.iso

Make sure to save files somewhere you'll remember, like the Documents folder.

## Let's Begin!
### Setting up  our Virtual Machines 
1. Open VMware Workstation 
2. Right click the left pane and select "New Virtual Machine"
3. Select "Typical" in the Virtual Machine Configuration prompt, then click Next
4. Select "Use ISO image"
5. Click "Browse" next to the "Use ISO image" option
6. In the file manager window that opens, navigate to the location you saved the files from earlier. Check your Downloads folder if in doubt
7. Select the Debian 13 .iso file
The latest version of VMware workstation should detect Debian 13
8. Click Next
9. Name your VM! I'll be naming mine Deb01 and Deb02, relative to the order I create them. Click Next.
10. If you plan on needing more storage for your VMs, increase the Maximum disk size. Otherwise, just click Next.
11. Click "Customize Hardware"

If you happen to be below the recommended hardware specifications, follow these steps, otherwise skip to step 12.
* Click "Memory" in the left pane, then click or drag to "1 GB" on the slider, or type "1024" in the "Memory for this machine:" text box
* Click "Processors" in the left pane, then type "1" in both text boxes.

12. Click "Network Adapter" in the left pane, then select Host-only
13. Click Close
14. Click Finish

### Setting up and installing Debian

15. You may hear a loud beep when Debian 13 boots, this is normal although a bit startling
16. Quickly, select "Install" or "Graphical Install" using the arrow keys to navigate and Enter key to select
If Debian begins speaking to you audibly (literally), shut down the VM, turn it on again, then select "Install"
17. Continue by using the arrow keys and pressing Enter until you are asked to continue without a default route.
18. Select \<Yes\>
19. No name server address is needed for now, press Enter
20. Set the hostname to whatever you named your VM, or use Deb01 or Deb02
21. No domain name is needed for now, press Enter
22. Create a root password then enter it again
23. Create a user and password, username doesn't matter. I'll be using "user" for my username.
24. Select your timezone
25. Select "Guided - use entire disk"
26. Press Enter until you're asked to write changes to disks, then select \<Yes\>
27. When prompted to scan extra installation media, select \<No\>
28. When prompted to use a network mirror, select \<No\>
29. Participation in the usage survey doesn't matter for the lab, select either option you'd like
30. Using the arrow keys and Spacebar to select items, select the desktop environment you like. I'll be using the defaults for this guide, so make sure the items; Debian desktop environment, GNOME, and standard system utilities are selected, then press Enter to confirm
31. When prompted to install the GRUB boot loader to your primary drive, select \<Yes\> 
32. Select "/dev/sda" in the Device for the boot loader installation prompt
33. Select \<Continue\>

Repeat all steps to create a second Debian installation, naming this one Deb02 or whatever you feel like.

### Setting up the pfSense VM

1. In VMware workstation, right click the left pane and select "New Virtual Machine"
2. Select Typical, click Next 
3. Select Use ISO image if not already selected, then click Browse
4. In the file manager window that opens, navigate to and select your pfSense .iso file
5. Name the VM pfSense or whatever you prefer
6. Click Next unless you need more storage, then specify it here. Otherwise just click Next.
7. Select Customize Hardware
8. Select "Memory" in the left pane, then either type "1024" or "2048" in the "Memory for this virtual machine" text box. Or drag the slider to either 1 GB or 2 GB
9. Select "Processors" in the left pane, optionally change the "Number of cores per processor" to 2
10. At the bottom of the left pane, select "Add" 
11. Click "Network Adapter" then click Finish
12. With the new network adapter selected, usually called "Network Adapter 2", under the Network Connection section select "Host-only"
13. Click Close
14. Click Finish
15. Inside the pfSense VM, press Enter to accept the copyright and distribution notice
16. Press enter until you are asked to select disks for ZFS Configuration
17. Select the only disk with Spacebar, then press Enter.
18. Use the arrow keys to select \<YES\> then press Enter.
19. Select <\No\> then select <\Reboot\>

### Setting up pfSense and the network
In the freshly installed pfSense VM, you should be greeted with a few options. 

1. Type 7 and press Enter
2. Type 8.8.8.8 and press Enter
3. If you are receiving bytes back from 8.8.8.8, the VM's network hardware is set up correctly!
4. Power on a Debian Linux VM
5. Log in 
6. Open a terminal (in gnome, click the top left button and type Terminal, then enter)
7. Type 'ip a' and press Enter
8. Note the IP address for "ens33" or similar. Look for "inet 172.16.\*.\*" colored in purple. In my case, my LAN IP is 172.16.89.133, but this will likely be different for you.

Write that IP address down somewhere, this is very important! The IP should not be 127.0.0.1, that is localhost, not the host-only network that we need.
Note the first three octets of your IP, this is your subnet. So in my case, since my LAN IP is 172.16.89.133, my subnet will begin with 172.16.89.
Write down your subnet address ending in a number that likely isn't used by the network, like 172.16.89.3, for example.
This will be the pfSense IP.

Now to finally set up pfSense's address so our Debian VMs can use it as the gateway!

1. Go back to (or power on) the pfSense VM.
2. Press Enter to continue if needed
3. Type 2 and press Enter
4. Type the number associated with the LAN interface
5. Type the IP address you noted for pfSense earlier, in my case it is 172.16.89.3
6. Type 24 and press Enter (if this fails, try 16 instead.)
7. Press Enter until you are prompted to enable DHCP server, then type y and press Enter
8. Enter the start address for DHCP to assign IPs from, in my case I'll be using 172.16.89.4
9. To ensure I don't interfere with VMware, the end IP address range should end in 253. I'll be using 172.16.89.253
10. For the sake of convenience, type y and press Enter when prompted to revert to HTTP as the webConfigurator protocol
11. You should see a message saying you can now access the web interface at the LAN IP address you assigned.
12. Press Enter to continue
13. Return to your Debian VM and wait a while for DHCP to assign IPs
14. Open a terminal and ping 8.8.8.8
15. If you receive bytes back, you're done!

You can now manage pfsense from either Debian VM using a web browser. Make sure to specify http in the URL!
For example, http://172.16.89.3
Default pfsense login credentials can be found here: https://docs.netgate.com/pfsense/en/latest/usermanager/defaults.html
Or login using admin, pfsense
