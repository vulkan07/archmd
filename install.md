# Installing Arch Linux
This is a simple guide to install Arch Linux from the terminal.
It isn't that difficult as it may seem at first. Just follow the steps.

< **NOTE: This guide is still not complete. Use at your own risk.**
> **NOTE: This guide only applies to UEFI GPT systens. (No BIOS MBR)**


## Getting Started

> NOTE: If you are doing a dual boot system with Windows, install Windows first (if it's not already installed).

> NOTE: If you are doing a dual boot system with Windows, shrink the data partitions to make space for Linux.

> NOTE: If you are doing a dual boot system with Windows, back up all your data! I do not guarantee no data loss.

1. Install an Arch ISO on a USB flash drive. (On Windows: Rufus, on Linux: Etcher)
2. Enter UEFI, and change the boot order to boot into the USB flash drive.

   Note: If you get something like "secure boot violation", disable secure boot or fast boot in the UEFI to boot into the USB.
3. Once the console appears, change the keyboard layout: `$ loadkeys hu` for Hungarian.
4. Verify that you've booted in EFI mode

   If `$ ls /sys/firmware/efi/efivars` returns *No such file or Directory*, you're not in EFI mode.
   Do not continue installing. You may need to change the BIOS from legacy boot to EFI boot.
5. Connect to the internet
   - **Ethernet**: should work without any configuration
   - **Wifi**:
     1. Enter `$ iwctl` (a pre-installed network commandline tool)
     2. `$ device list` lists the available hardware interfaces on your machine. Choose the one you want to use (usually **`wlan0`**) and remember the name.
     4. `$ station <device> scan` scans the nearby Wi-Fi networks.
     5. `$ station <device> get-networks` lists the scanned networks.
     6. `$ station <device> connect <Wi-Fi name>` Connects to the selected Wifi network and will prompt you for the password.
     7. Exit iwctl with `$ exit` 
6. Verify Internet Connection

   `$ ping google.com` If there are packets returned every second, you're good. If nothing happens, the network configuration is not working correctly.

 ## Partitioning
This is a critical part. Be careful and do not mess things up, or you could erase all your data.
Here is a brief overview of common partition types and purposes:
| Partition Name| Format | Purpose    |
|--------------|-----------|------------|
| EFI  System          | FAT32       | Boot files for UEFI systems. Either GPT (better) or MBR (legacy) scheme |
| Microsoft Data       | NTFS/FAT32 | The data partition you can use and see on Windows |
| Microsoft Reserved   | None | Comes with a MS Data partiton |
| Microsoft Recovery   | NTFS/FAT32 | Comes with a MS Data partiton |
| Linux Data   | Ext3/Ext4  | Main data partition for linux |
| Linux Swap   | Swap | Linux writes it's memory here when hibernating. This is optional |

**I. Determine devices & partitions**

- **Drives** are usually named **`sd`** + **`<letter>`** (like `sda`, `sdb`, `sdc`...)

- **Partitions** are named **`<drive name>`** + **`<number>`** (like `sda1`, `sda2`, `sda3`...)

- So for example `sdc2` would mean the second partition on the 3rd drive

- **Run `$ lsblk`** to list the disks and partitions on your system.
   (NOTE: You will also see the USB flash drive)

- **The path to a partition is: `/dev/<partition>`** (like `/dev/sda2`)


**II. Create partitions**

> Note: `cfdisk` won't actually modify the partitions until you say Write, so you can quit and retry if you messed up.
1. Enter `$ cfdisk <drive>` (i.e. `$ cfdisk /dev/sda`) 
2. Delete partitions:
   - If you have Windows installed, do not delete any Microsoft partitions nor the EFI partition!
   - If you want to make a clean install, you can delete all partitions
3. Allocate **500MB** for the **`EFI`** Partition. Then set its type to EFI partition.
4. _(Optional)_ Allocate at least half the size of your RAM to a SWAP partition (if you want hibernation)
5. _(Optional)_ Allocate a separate Linux partition for your `/home` directory.
6. Allocate the rest of the drive (or any size you want) to your main Linux partition, and set its type to Linux partition.
7. Confirm the new layout, then select the option __`write`__ and confrim by typing `yes`. Then Quit.


**III. Format the new partitions**

**Formatting a partition looks like this: `mkfs.<type> <partition>`** (example: `mkfs.ext4 /dev/sda5` to make sda5 an ext4 partition)

1. Format EFI partition: `mkfs.fat -F 32 <partition>` **(Don't do this if you have another OS installed)**
2. Format Swap Partition: `mkswap <partition>`, then run `swapon <partition>` to enable it **(Only if you created a swap partition)**
3. Format Linux partition(s): `mkfs.ext4 <partition>` 


**IV. Mount the partitions**

In order to access a partition in Linux, you must mount it to your filesystem somewhere.
**Example:** You want to mount a **sda3** to **/data**, you run `mount /dev/sda3 /data`. This will make the contents of **sda3** accessible from **/data**.

1. Mount EFI partition: `mount <partition> /mnt/boot/efi`
2. Mount Linux system partition: `mount <partition> /mnt`
3. We will do the mounting of Windows and Network partitions after installing.


## Installing packages

**V. Install base Linux packages to /mnt**
Run `pacstrap -K /mnt base base-devel linux linux-firmware`
Additional packages you may want to install now:
- `vi`/`vim`/`nano` (or any text editor)
- `networkmanager` (or you won't have internet if you're using WiFi)
- `sudo`
- `git`
- `grub` + `efibootmgr` for a bootloader
- `os-prober` if you have other OSs installed and want to dual boot
- `intel-ucode`/`amd-ucode` for Intel/AMD processors


**VI. Generate fstab**

`/etc/fstab` is a text file that contains all the partitions that get mounted when starting a Linux system. You can edit it at any time by hand later, 
but now we will generate it from the mounts we've just did.
To do that, run `$ genfstab -U /mnt >> /mnt/etc/fstab`. You can confirm its correctness by `$ cat /mnt/etc/fstab`


**VII. Change Root**

Run `$ arch-chroot /mnt` to 'enter' the actual Linux in your system and continue setup from there. (You're not actually running the linux on your machine yet, just setting it up)


**VIII. Set Timezone**

Run `$ ln -sf /usr/share/zoneinfo/Europe/Budapest /etc/localtime` (for Budapest)


**IX. Synchronize hardware clock**

Run `$ hwclock --systohc`


**X. Set locale**

1. Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` (or whatever you prefer)
2. Run `$ locale-gen`
3. Run `$ echo "LANG=en_US.UTF-8" > /etc/locale.conf`


**XI. Set keyboard layout for console**

Run `$ echo "KEYMAP=hu" > /etc/vconsole.conf` (for Hungarian)


**XII. Set host name**

This is your machine's name which will appear in the console and on networks.

Run: `$ echo "<name>" > /etc/hostname` (replace <name> with your name of choice)


**XIII. Mkinitcpio**

This shouldn't be required, but run it anyway: `$ mkinitcpio -P`


**XIV. Set root Password**

Set root user password (or you won't be able to log in)

Run `$ passwd`, you will be prompted to enter the password twice (you can't see the characters you type, this is for security)

**XV. Set up Bootloader**
**GRUB:**
- If you want to dual boot, edit `/etc/default/grub` and uncomment this line: `$ GRUB_DISABLE_OS_PROBER=false`,this will enable os-prober
- Additionally you can edit other stuff in that file, like GRUB_TIMEOUT, and GRUB_DEFAULT...
- `$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`
- `$ grub-mkconfig -o /boot/grub/grub.cfg`

**XVI. Reboot**
1. Type `$ exit` to exit from the chroot, then type `reboot`, and cross your fingers.
2. Unplug the USB flash
3. If you installed the system successfully, you'll see GRUB and choose 'Arch Linux'.
4. Then log in from as 'root' with the password you set.
5. Done!


## Post-Install
### Add your user
1. Create your user with a home directory:
   `$ useradd <name> -m` (replace with your name, usually not longer than 5 letters, like 'barni')
2. Set password for user
   `$ passwd <name>`
3. Add user to groups
   `$ usermod -aG <groups>` (replace <groups> with the groups you want to add, separated by space)
   Common groups you might consider:
      - `wheel`: makes your user sort of an 'administrator'
      - `seat`: allows you to run desktop sessions
      - `video`: allows you to do graphical stuff
      - `audio`: allows you to do audio stuff
      - `optical`: allowss you to do disks & removable media related stuff
      - `storage`: allowss you to do storage related stuff
4. Make yourself a sudoer (administrator):
   - `$ EDITOR=vim sudoedit /etc/sudoers`
   -
   ```
     %wheel ALL=(ALL:ALL) ALL
     <your username> ALL = (ALL:ALL) ALL
   ```

### Graphical Interface
- Install a greeter like `lightdm` or `sddm`
- Enable your greeter (i.e. `systemctl enable sddm`)
- **Sway on Wayland**
   1. Install `wayland` + `sway`
- Restart

### Audio
Install either `pipewire` + `wireplumber`? or `pulseaudio`

### Bluetooth
1. install `bluez` and `bluez-utils`
2. enable/start `bluetoothd.service`

TODO:
users
sudo
audio
yay
...
