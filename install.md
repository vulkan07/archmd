
# Installing Arch Linux
This is a simple guide to install Arch Linux from the terminal made by Barni.
It may seem long, however it isn't *(that much)*. 

> **NOTE:** This guide is **still not complete**. Use at your own risk.
> **NOTE:** This guide is **only for UEFI GPT** systens. (No BIOS MBR)

## Prequisities
Here are some things that you need to know before attempting an install:
1. How to make an ISO flash drive
2. How to enter your UEFI and change the boot order
3. Basic usage of the Linux console (zsh/bash)
4. How to edit a text file in console (using vim/nano...)
5. How to use `cfdisk`
6. Know your hardware (especially the drives for partitioning) 

----

## Getting Started

### If you want to dual boot with Windows
Installing Windows will overwrite Linux's bootloader and you'll have to fix it from a Linux flash ISO, so if you're doing a fresh installation for Windows, **install Windows before Linux**.
If you already have Windows installed, shrink one of the partitions to make space for your Linux (at least 10GB).
### Preparation

1. Install an Arch ISO on a USB flash drive. (use Rufus *(Windows)* or Etcher *(Linux)*)

3. Change the boot order to boot into the ISO in your UEFI.

   > **NOTE:** you may need to disable secure boot or fast boot in the UEFI or it won't work

4. Once the console appears, change the keyboard layout with `$ loadkeys hu` to Hungarian.

5. Verify that you've booted in EFI mode

   If running `$ ls /sys/firmware/efi/efivars` returns a bunch of files, you're good to go.
6. Connect to the internet
   - **Ethernet**: should work without any configuration
   - **Wifi**:
     1. Enter `$ iwctl` (a pre-installed network commandline tool)
     2. `$ device list` lists the available hardware interfaces on your machine. Choose the one you want to use (usually **`wlan0`**) and remember the name.
     4. `$ station <device> scan` scans the nearby Wi-Fi networks.
     5. `$ station <device> get-networks` lists the scanned networks.
     6. `$ station <device> connect <Wi-Fi name>` Connects to the selected Wifi network and will prompt you for the password.
     7. Exit iwctl with `$ exit` 
7. Verify Internet Connection

   `$ ping google.com` If there are packets returned every second, you're good.

----

 ## Partitioning
This is a critical part. Be careful and do not mess things up, or you could erase all your data.
Here is a brief overview of common partition types and purposes:
| Partition Name| Format | Purpose    |
|--------------|-----------|------------|
| EFI  System          | FAT32       | Boot files for UEFI systems. Either GPT (better) or MBR (legacy) scheme |
| Microsoft Data       | NTFS/FAT32 | The data partition you can use and see on Windows |
| Microsoft Reserved   | None | Required by an NTFS partition, ignore it |
| Microsoft Recovery   | NTFS/FAT32 | Required by an NTFS partition, ignore it |
| Linux Data   | **Ext3/Ext4**  | **Main data partition for linux** |
| Linux Swap   | Swap | Linux writes it's memory here when hibernating. This is optional |

### I. Determine devices & partitions
- **Drives** are  named **`sd`** + *`<letter>`* (like `sda`, `sdb`, `sdc`...)

- **Partitions** are named **`<drive name>`** + *`<number>`* (like `sda1`, `sda2`, `sda3`...)

> **NOTE:** If you have an NVME drive, the drive name may look like `nvme0n1` and the partitions will be like `nvme0n1p1`, `nvme0n1p2`...

- So for example `sdc2` would mean the second partition on the 3rd drive

- Run **`$ lsblk`** to list the disks and partitions on your system.
   (NOTE: You will also see the USB flash drive)

- **The path to a partition is: `/dev/<partition>`** (like `/dev/sda2`)


### II. Create partitions
> Note: `cfdisk` won't actually modify the partitions until you say Write, so you can quit and retry if you messed up.
1. Enter `$ cfdisk <drive>` (example: `$ cfdisk /dev/sda`) 
2. Delete old partitions:
   - If you have Windows installed, do not delete any Microsoft partitions nor the EFI partition!
   - If you want to make a clean install, you can delete all partitions
3. Allocate 500MB for the **EFI partition**. Then set its type to EFI partition.
4. _(Optional)_ Allocate the size of your RAM for a **SWAP partition** (if you want hibernation or if you have <2G of RAM)
5. _(Optional)_ Allocate a separate Linux partition for your `/home` directory.
6. Allocate the rest of the drive (or any size you want) to your main Linux partition, and set its type to Linux partition.
7. Confirm the new layout, then select the option __`write`__ and confrim by typing `yes`. Then Quit.


### III. Format the new partitions
**Formatting a partition looks like this: `$ mkfs.<type> <partition>`** 
(example: `mkfs.ext4 /dev/sda5` to make sda5 an ext4 partition)

1. Format EFI partition: `mkfs.fat -F 32 <partition>` **(Don't do this if you have another OS installed)**
2. Format Swap Partition: `mkswap <partition>`, then run `swapon <partition>` to enable it **(Only if you created a swap partition)**
3. Format Linux partition(s): `mkfs.ext4 <partition>` 


### IV. Mount the partitions
In order to access a partition in Linux, you must mount it to your filesystem somewhere.
**Example:** You want to mount a **sda3** to **/data**, you run `$ mount /dev/sda3 /data`. This will make the contents of **sda3** accessible from **/data**.

1. Mount EFI partition: `mount <partition> /mnt/boot/efi`
2. Mount Linux system partition: `mount <partition> /mnt`
3. Mounting Windows and Network partitions should wait after installing.

----

## Installing packages
### 1) Install Linux to /mnt
Run `$ pacstrap -K /mnt base base-devel linux linux-firmware <and other packages you want>`
Additional packages you may want to install now:
- `vi`/`vim`/`nano` (or any text editor)
- `networkmanager` (or you won't have internet if you're using WiFi)
- `sudo`
- `git`
- `grub` + `efibootmgr` for a bootloader
- `os-prober` if you have other OSs installed and want to dual boot
- `intel-ucode`/`amd-ucode` for Intel/AMD processors

### 2) Generate fstab
`/etc/fstab` is a text file that contains all the partitions that get mounted when booting a Linux system. You can edit it any time by hand later, but now we will generate it from the mounts we've just did.
To do that, run `$ genfstab -U /mnt > /mnt/etc/fstab`. You can confirm its correctness by `$ cat /mnt/etc/fstab`

### 3) Change Root
Run `$ arch-chroot /mnt` to "enter" the actual Linux on your machine and continue setup from there. (You're not actually running the linux on your machine yet, just setting it up)

### 4) Set Time
1. Run `$ ln -sf /usr/share/zoneinfo/Europe/Budapest /etc/localtime` (for Budapest timezone)
2. Run `$ hwclock --systohc` to update your hardware clock

### 5) Set locale
1. Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` (or whatever you prefer)
2. Run `$ locale-gen`
3. Run `$ echo "LANG=en_US.UTF-8" > /etc/locale.conf`


### 6) Set keyboard layout
To make a keyboard layout persistent after rebooting, run 
`$ echo "KEYMAP=hu" > /etc/vconsole.conf` (for Hungarian)


### 7) Set your machine's name
Run: `$ echo "<name>" > /etc/hostname` (replace \<name> with your name of choice)


### 8) Mkinitcpio
This shouldn't be required, but run it anyway: `$ mkinitcpio -P`


### 9) Set root Password

Set root user's password (or you won't be able to log in)

Run `$ passwd`, you will be prompted to enter the password twice (you can't see the characters you type, this is for security)

### 10) Set up Bootloader
**For GRUB:**
1. Edit `/etc/default/grub` 
	- If you want to dual boot, uncomment this line: `$ GRUB_DISABLE_OS_PROBER=false`
	- Additionally configure other stuff like GRUB_TIMEOUT, and GRUB_DEFAULT...

2. Run `$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`

3. Run `$ grub-mkconfig -o /boot/grub/grub.cfg`

**For bootctl**:
https://wiki.archlinux.org/title/Systemd-boot#Installing_the_UEFI_boot_manager

### 11.  Reboot
1. Type `$ exit` to exit from the chroot, then type `reboot`, and cross your fingers.
2. Unplug the USB flash
3. If you installed the system successfully, you'll see GRUB, then choose 'Arch Linux'.
4. Log in as 'root' with the password you set.
5. Enjoy!


## Post-Install

### Check Internet connection
 Use `$ ping google.com` to check if your connection is working.
 It should work out of the box if you are using LAN.
 If you're using WiFi, use `$ nmtui` or `$ nmcli` to connect to a network.

### System update
Once you have internet connection, update your system by running `$ sudo pacman -Syu`

### Install an AUR helper
AUR (Arch User Repository) is where community-made packages are. `pacman` wont allow you to install them, so you should install an AUR helper, like **`yay`**:
1. Install `git` and `base-devel` if you don't have them
2. `$ git clone https://aur.archlinux.org/yay.git`
3. `$ cd yay`
4. `$ makepkg -si`
5.  Check if yay works by `$ yay --version`
6. You can delete the `yay` folder 

### Add your user
1. Create your user with a home directory:
   `$ useradd <name> -m` (replace with your name, like 'barni')
2. Set password for user
   `$ passwd <name>`
3. Add user to groups
   `$ usermod -aG <groups>` (replace \<groups> with the groups you want to add, separated by space)
   Common groups you might consider:
      - `wheel`: makes your user sort of an 'administrator'
      - `seat`: allows you to run desktop sessions
      - `video`: allows you to do graphical stuff
      - `audio`: allows you to do audio stuff
      - `optical`: allows you to do disks & removable media related stuff
      - `storage`: allows you to do storage related stuff
4. Make yourself a sudoer (administrator):
   - `$ EDITOR=vim sudoedit /etc/sudoers` (replace vim with your editor of choice)
   - TODO (do that what you see below):
   ```
     %wheel ALL=(ALL:ALL) ALL
     <your username> ALL = (ALL:ALL) ALL
   ```

### Graphical Interface
- Install a greeter like `lightdm` or `sddm` that will show up after booting Linux and allow you to log in
- Enable your greeter (i.e. `$ systemctl enable sddm`)
- **Sway on Wayland**
   1. Install `wayland` + `xorg-xwayland` + `sway`
   2. `sddm` should  recognize it as a session after a reboot
   > NOTE: sway won't launch with Nvidia GPUs unless passing it `--unsupported-gpu`
- Restart

### Audio
Install either `pipewire` + `wireplumber` (newer) or `pulseaudio` (older)

### Bluetooth
1. install `bluez` and `bluez-utils`
2. enable/start `bluetoothd.service`

### Laptops & Battery
Install `tlp`, it drastically optimizes battery
TODO: upower?
