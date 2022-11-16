
## Installation Guide for Arch Linux with GNOME

This instructional guide will walk you through the steps to installing Arch Linux on the VMware Workstation as well as adding users with sudo permissions and the installation of a desktop environment, in this case we use GNOME. This guide will also include errors that are possible during installation in order to document and help users if they attempt to install it on their own using this guide who may run into issues and to help narrow down solving any problems they may have. With that said, anything that has the comment **NOTE** in these steps are issues I ran into during installation, so if any problems occur please refer to the notes for they may be similar issues you could run into.

For more information, please visit the wiki: https://wiki.archlinux.org/title/installation_guide

*Before installing a virtual machine, reboot your PC in BIOS or UEFI mode and verify that virtualization for your CPU is enabled, otherwise your virtual machine will not boot on launch.*

> # Installing Arch Linux
1. Open VMware Workstation and create a new workstation, select **“Installer disc image file (iso)”** and set the path to the location of your **archlinux.iso** file which you can download from the wiki page. Hit next and select **“Linux”** as the Guest operating system, and **“Other Linux 5.x and kernel 64-bit"** for version and hit next. Give your Virtual Machine a name and hit next again.

2. Set your disk capacity to 20GB of storage, or however much space you wish to use, and keep the setting **“Split virtual disk into multiple files”** highlighted and hit next. Then select **“Customize Hardware”** on the settings window and set your RAM to 2GB, afterwards hit finish. Remember, your selected RAM will be double the space for swap memory.
   > **NOTE, depending on the RAM size in your PC, it will allow you to run more RAM memory than suggested. In case a mistake is made you can always increase or decrease the size of RAM in the options settings after installation.**
3. Start the virtual machine and select **‘Arch Linux install medium (x86_64, BIOS)’** when prompted on the splash screen, otherwise it will automatically pick the first selection if you're idle for 15 seconds, it will then open a terminal screen prompt using bash. Typing `lsblk` will show your reserved hard drive not mounted or partitioned indicating you are currently running in memory space. Leave the console keyboard layout as is because by default it is set to US. You can also verify the boot mode by listing the **‘efivars‘** directory by typing `ls /sys/firmware/efi/efivars`.
   > **NOTE, for my instance, the directory does not exist because my computer runs the VM in BIOS mode, however if it shows the directory then it is in UEFI mode. This could vary depending on which bootloader is better for you.**
   
   > **NOTE 2, if you choose to use a different keyboard layout, it can be done with by typing `ls /usr/share/kbd/keymaps/**/*.map.gz` to list the keyboard layouts which can then be set by typing `loadkeys <selected language>`**
4. To partition the disks, type `cfdisk /dev/sda`. Select **‘gpt’** for label type and in the partition window, select **‘new’** and make its partition size `1M` **(1 Megabyte)** and scroll to **‘type’** on the right and pick **‘BIOS Boot’** partition in the list. Create another new partition for the swap partition and set its size to double the RAM space *(if RAM set to 2GB, then it’ll be 4GB)* which will be `4G` and set the partition **‘type’** as **‘Linux Swap’**. With your remaining space, select **‘new’** and leave the remaining size as it is and hit enter to set it and confirm its type is **‘Linux Filesystem’**,then scroll to the right and select **‘Write’**, typing `yes` when prompted and then quit out back to the bash prompt.
   > **NOTE, verify before continuing your disks were partitioned by typing `lsblk`. Not verifying and starting it with errors can cause the partition table to be unreadable forcing you to repartition everything again or have no space left when installing packages.**
5. To format the partitions, list them out with `lsblk` and create an *ext4* format to **sda3** by typing `mkfs.ext4 /dev/sda3`. Next, format the swap partition by typing out `mkswap /dev/sda2` and enable it by typing `swapon -a`. Change directory to the **/mnt** folder once again and lastly mount the file system by typing out `mount /dev/sda3 /mnt`. Change back to the home directory and again back in **/mnt** to verify it is properly formatted which should now have a folder called *“lost+found”*.
6. Enable the mirror list by editing it with **nano**, type `nano /etc/pacman.conf` to access the config and then scroll all the way down and uncomment the lines **‘[Multilib]’** and **‘Include = /etc/pacman.d/mirrorlist’**. Once done, save and close and then synchronize pacman’s data packages by typing in `pacman –Syy`. While still in the **/mnt** folder, you then want to install the basic Linux essential packages, firmware, and apps you would like to use. For my case, I ended up with installing the following by typing the command: `pacstrap /mnt base linux linux-firmware vim grub zsh openssh sudo`. Let it download and install, this should take a few minutes.
   > **NOTE! If you forget to uncomment the mirrorlist in the previous step, some packages that are 32bit will fail to retrieve from any mirror you are downloading from. Base linux and linux-firmware are your two most essential for the kernel to work properly, vim is not installed automatically nor’ is openssh.**

   > **NOTE 2, if you are unable to install any of the packages provided, move on to step 7 and retry step 6 after configuring your dhcpcd ports.**
7. Verify you are connected to the internet through the VMware by checking the bottom window’s connection icons on the VM window itself or by pinging to “archlinux.org” by typing `ping archlinux.org`. If you are plugged into the internet via ethernet then you will automatically receive a connection. However, if not, change directory to the **/mnt** folder and install the ‘DHCP’ client by typing `pacstrap /mnt dhcpcd`.
   >**NOTE! Originally I was not getting internet connection UNTIL installing dhcpcd, this allowed my dynamic host client to configure automatically and then ping to the archlinux website.**
8. Edit the configuration files by generating the **fstab** file by creating a new folder called `genfstab /mnt` then follow up by `genfstab  /mnt >> /mnt/etc/fstab` to configure the mount partition. Once done, now turn the mount directory to the root folder by typing in `arch-chroot /mnt` and it will now run the system as if you were the root user. While in this mode, configure the **GRUB** boot by installing it in bash using the command `grub-install /dev/sda` which installs it to the 1M partitioned disk that we assigned as the boot loader in step 4, then generate its config file with the line `grub-mkconfig -o /boot/grub/grub.cfg`.
   > **NOTE. The bootloader GRUB does not automatically install upon download and requires these inputs in order for Linux to post upon reboot, failing to do so can cause the machine to not post and get stuck in a blackscreen forcing you to reinstall.**
9. Set up your root password by entering `passwd` and enter it twice to confirm. Check your time zone and set it up by typing `timedatectl set-time "yyyy-MM-dd hh:mm:ss"` and type `hwclock --systohc` to confirm its set properly. Now you want to then type `exit` to leave the root user and then restart it by typing in `reboot`. Once it has restarted it'll prompt a username and password. Login as `root` with your new password and start the dynamic IP by typing in `systemctl start dhcpcd` and then retype the same command replacing `start` with `enable` instead. After this, your Arch Linux should be completely configured and ready for the installation of a Desktop Environment.
    > **Note! Forgetting to type `systemctl enable dhcpcd` will cause it to not remain persistent after rebooting the kernel which in return will disconnect you from the internet upon startup, forcing you to start dhcpcd again before enabling it.**


> # Installing GNOME Desktop Environment
   For more information about GNOME, please visit the website: https://phoenixnap.com/kb/arch-linux-gnome 

  1.  Begin installing your Desktop Environment, in this case using ‘GNOME’, type out in the bash terminal while still signed in as root `sudo pacman –S xorg gnome gnome-extra gdm` and accept all the default settings. If you wish to make any changes to them afterwards, you can do so in the desktop environment once finished installing. In case you wish to make changes however, read through each line and manually set them when prompted. While entering changes and once finished, then type enter `Y` at the end to download and install the DE. After it finishes installing, you will lastly be required to enable the display manager by typing out `systemctl enable gdm`. Do not reboot just yet.
   
  2.  Before rebooting, you need to add any users you plan on having with sudo permissions and a home directory by first using the command `useradd -m <username>` to create a user, and then set their root permissions by adding them to the wheel group with the command `usermod -aG wheel <username>`. Finish this step by setting up the password for each user with command `passwd <username>`. After adding and setting the group permissions, exit and reboot the VM and upon restart it will now open up in the Desktop Environment of your choosing, hence Gnome for this example.

1. Sign in on the user you created and open the **terminal** from the *activities* page. Install ‘vi’ to download visudo by typing `sudo pacman -S vi` and in the terminal it will ask for the users password to use sudo permissions, type `visudo` to access the permissions file and scroll down until you find the line **#%wheel ALL=(ALL) ALL** and uncomment it, then save and exit by pressing ‘ESC’ and typing `:wq`. Reboot the VM one last time and the installation is fully complete with root permissions available to all users in the **‘wheel’** group. 
   > **NOTE! A symbolic link will be required in order to access VIM and VISUDIO since the program is not installed by default and will require a direct link to the path ‘/usr/bin/vi’ directory.**

This marks the end of the installation for Arch Linux with its DE, please refer to the author if you have any questions in regards to the guide and I will answer them as best as I can. Thank you and I hope you enjoy your now new Arch-Linux machine.
![Screenshot](https://i.postimg.cc/cHYrYhbV/proofconcept.jpg "Arch Linux connected to Harvey at Utulsa.")
