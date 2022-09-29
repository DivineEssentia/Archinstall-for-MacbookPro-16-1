# Welcome to 16,1 Arch!

This is a guide on how to install Arch Linux on a Macbook Pro 16 inch 2019 (16,1) using Archinstall. 


## Requirements

- Internet Connection
- The laptop
- A USB Drive with at least 1GB
- An external keyboard and mouse
- Join the T2Linux discord: [discord.gg/Jayz5f5](https://discord.gg/Jayz5f5 "https://discord.gg/Jayz5f5")

# Part 1: Installing the OS

## I. Creating a Partition
even if you may wish to remove MacOS completely, i learned the hard way that it will break everything. 
- Open Disk Utility
- Select your drive
- Press “Partition” in the top of the window and set the size you would like. The name doesnt matter because it will be overwritten.
- ### IMPORTANT:  MAKE SURE TO REMEMBER THE SIZE OF THE PARTITION SO YOU CAN INSTALL ARCH TO THE CORRECT ONE! 

## II. Creating the Bootable Drive

- Download the ISO of your choice from https://github.com/t2linux/archiso-t2/releases
- Download Balena Etcher https://www.balena.io/etcher/‘
- Use the instructions in the app to flash your drive with the ISO


## III. Disabling Secure Boot
- Reboot your mac and before you see the apple logo hold `Cmd+R` until the logo goes away. 
- log into recovery mode and press utilites on the menu bar and select Startup Security Utility.
	- Set Secure Boot to “no” and enable external boot. 
- Then press CMD+Q and select Terminal from Utilities.
	- type `csrutil disable` for good measure
	- then type `reboot`

## IV. Booting into the installer

- Before you see the Apple logo, hold down `Alt/Option` until you see a screen with two disks. 
- use your arrow keys to select the orange drive named EFI and press Enter
- when it gives you options for an installer select the one that includes `16,1/4`
## V. Setting up required services 

This sets up WiFi for now and other required utilities
- once you see a terminal prompt type `iwctl`
	- type `device list` to see your devices. wlan0 will likely be the one you need. 
	- then type `station wlan0 device scan` to find networks around you. 
	- type `station wlan0 connect [YOUR NETWORK NAME]` and input the password 
	- press `ctrl+c`
- Now type `ping -c 3 archlinux.org` to make sure you have connection.
	- If you have 3 lines that say `RECIEVED` on them it means you are connected

## VI. Installing Arch with Archinstall
This tutorial uses the Archinstall method to setup and install the OS quickly and easilly. If you would like to install this the hard way where you learn how to create a custom install, follow https://wiki.t2linux.org/distributions/arch/installation/
- To start the installation process, type `archinstall` .
- After a few seconds, a menu should show up
- Move down to Drives and press enter and select the largest storage device
- Then select Disk Layout and press enter 
- Select “Select what to do with the indiviual drives”
	- Boot Drive
		- Then `Create new partition`
			- Type: FAT32
			- Start: 3MB
			- End: 203MB
		- Select  `assign mount point for a partition` and use the one you just created. Mount it at /boot 
		- Select `Mark/unmark a partition as bootable`and set this to be true for this partition
		- Select `Mark/unmark a partition to be formatted` and set this to be true for this partition
	- OS Partition
		- This will be using the partition that you created at the beginning
		- Select  `assign mount point for a partition` and mount it at `/`.
		- Select `Mark/unmark a partition to be formatted` and set it to True.
	- Go to the bottom and select `Save and Exit`
- Set Bootloader to `grub` 
- Set Hostname to whatever you would like your computer to be called. I used “archbook-pro”
- Set root password to something you will easilly remember. I recommend using `root` because you will not be using it very often and its extremely easy to remember.
- Select User Account and create a new user. Enter the username and password and make the user a sudo user, so it can run root commands and other things of the like. 
- Set Profile to “Desktop Environment” and then your DE of choice. 
	> I personally recommend `GNOME` because it is very similar to MacOS and looks great. 
- Set Audio to Pulseaudio. Pipewire will break most things.
- Add additional packages that you would like to use. I STRONGLY recommend `firefox yay qt5`.
- Set Network Configuration to Installer Configuration.
- Set Timezone to your local timezone.
- Finally, you are done. Select Install and do what it asks you to do. You do not need to create a config because this will (hopefully) be the only time you need this. 
- Now go touch some grass or get a snack and wait for it to install.
- once it is done, type `y` when it asks you if you would like to chroot into the OS 
- Chroot into the OS: `arch-chroot /`
- Mount the boot drive: `mnt /boot`
- Remove the linux kernel: `pacman -R linux`
- Install the new kernel and the wifi firmware: `pacman -S linux-t2 apple-bcm-wifi-firmware`
- Regenerate the GRUB config: `grub-mkconfig -o /boot/grub/grub.cfg`
- Now type `pacman -Syu` and `reboot`


# Part II: Setup OS
Congratulations! If you followed this tutorial correctly, you have installed Arch on your Mac. Now, you need to to some more setup because many things are not installed or working by default. 
### WiFi Drivers
- To setup the Wifi drivers, you need to reboot into MacOS and download [this](https://wiki.t2linux.org/tools/firmware.sh) file.
- Open Terminal and run:
	- `chmod +x ~/Downloads/firmware.sh`
	-  `bash ~/Downloads/firmware.sh`
- Reboot back into Arch.
- run these commands:
	- `sudo umount /dev/nvme0n1p1`
	- `sudo mkdir /tmp/apple-wifi-efi`
	- `sudo mount /dev/nvme0n1p1 /tmp/apple-wifi-efi`
	- `bash /tmp/apple-wifi-efi/firmware.sh`
- Reboot.
- Go into GNOME WiFi settings and select your network and input the password. 
- You are now connected!
### Fan Drivers
- Install Git: `sudo pacman -S git`
- Clone the fan repo:  `git clone https://github.com/networkException/mbpfan`
- Run `cd mbpfan` 
- Compile it using: `make`
- Install it with: `sudo make install`
- To make it start at boot:
	- `sudo cp mbpfan.service /etc/systemd/system/`
	- `sudo systemctl enable mbpfan.service`
	- `sudo systemctl daemon-reload`
	- `sudo systemctl start mbpfan.service`
		> This will not always work (in my experience). If you see in the startup logs `[ERROR] Failed to start A Fan Daemon For Macbook Pro`, you must immediatly run `mbpfan` after logging in to prevent overheat. 
- Next, install nano using `sudo pacman -S nano` to be able to edit files.
- To edit the config, run `sudo nano /etc/mbpfan.conf`
	- edit the lines to match`min_fan1_speed = 3800` and `max_fan1_speed = 5200`.
	- add the lines `min_fan2_speed = 3800` and `max_fan2_speed = 5200`.
- Run `reboot` and if you see `[ERROR] Failed to start A Fan Daemon For Macbook Pro`, run `mbpfan` after logging in. 
- You now have a working fan manager!
## DKMS (Mouse, Keyboard, and Audio Drivers)
- To start, install the `dkms` package from pacman (`sudo pacman -S dkms`)
- Run `sudo nano /etc/pacman.conf` and add this to the bottom of the file: 
>`[Redecorating-t2]`
>`Server = https://github.com/Redecorating/archlinux-t2-packages/releases/download/packages`
- Then run `sudo pacman -S apple-bce-dkms-git apple-ibridge-dkms-git apple-ibridge-dkms-git` to install the required packages 
- Now, reboot.
- If you have any problems with this, Ask @Redecorating on the discord server
## Finishing up
Congradulations! If everything worked, you have installed Arch Linux on your Macbook Pro!

If you have any issues, ask on the discord server for t2linux since I wont really be able to help :D

#### Credits:
Written by: [DivineEssentia](https://github.com/DivineEssentia)
Some commands and instructions from https://wiki.t2linux.org
16,1 packages by [Redecorating](https://github.com/Redecorating)
