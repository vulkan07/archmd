
# Installing Arch Linux

This is the short guide to install Arch Linux from the terminal made by Barni.
Only for users who know what they're doing.

----

## Getting Started

### Preparation

1. Install an Arch ISO on a USB flash

3. Change boot order (you may need to disable _secure boot_ or _fast boot_)

4. Change keyboard layout: `$ loadkeys hu`

5. Verify EFI mode: `$ ls /sys/firmware/efi/efivars`
   
7. Wi-Fi
     1. Enter `$ iwctl`
     2. `$ device list` Choose the wireless interface (something-like **`wlan0`**)
        Note: if wlan0 is disabled either try `device <device> set-property Powered on` or `rfkill unblock <device>`
     4. `$ station <device> scan`
     5. `$ station <device> get-networks`
     6. `$ station <device> connect <Wi-Fi name>` (will prompt for password)

----

## Partitioning

### I. Create partitions
1. Enter `$ cfdisk <drive>` 
2. Delete old partitions (if doing clean install)
3. 500MB -> EFI partition
4. _(Optional)_ Size of RAM -> SWAP partition
5. _(Optional)_ Separate Linux partition -> `/home`
7. Main Linux partition
8. Confirm new layout, then __`write`__ and confrim by typing `yes`.

### II. Format the new partitions
1. EFI: `mkfs.fat -F 32 <partition>` **(Don't do this if you have another OS installed)**
2. Swap: `mkswap <partition>`, then `swapon <partition>` to enable it
3. Linux: `mkfs.ext4 <partition>` 

### III. Mount the partitions
1. Mount EFI: `mount <partition> /mnt/boot/efi`
2. Mount Linux system: `mount <partition> /mnt`
3. Mount Windows and Network partitions after installing.

----

## Installing packages
### 1) Install Linux to /mnt
Run `$ pacstrap -K /mnt base base-devel linux linux-firmware <and other packages you want>`
<br>
Additional packages you probably want to install now:
- `vi`/`vim`/`nano` (or any text editor)
- `networkmanager` (if using WiFi)
- `sudo`
- `git`
- `grub` + `efibootmgr` for a bootloader
- `os-prober` for dual boot
- `intel-ucode`/`amd-ucode` for Intel/AMD processors


### 2) Generate fstab
Run `$ genfstab -U /mnt > /mnt/etc/fstab`. You can confirm it by `$ cat /mnt/etc/fstab`

### 3) Change Root
Run `$ arch-chroot /mnt` 

### 4) Set Time
1. Run `$ ln -sf /usr/share/zoneinfo/Europe/Budapest /etc/localtime` (for Budapest timezone)
2. Run `$ hwclock --systohc` to update your hardware clock

### 5) Set locale
1. Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` (or whatever you prefer)
2. Run `$ locale-gen`
3. Run `$ echo "LANG=en_US.UTF-8" > /etc/locale.conf`

### 6) Set keyboard layout
`$ echo "KEYMAP=hu" > /etc/vconsole.conf` (for Hungarian)

### 7) Set your machine's name
Run: `$ echo "<name>" > /etc/hostname` (replace \<name> with your name of choice)

### 8) Mkinitcpio
This shouldn't be required, but run it anyway: `$ mkinitcpio -P`

### 9) Set root Password
Run `$ passwd`

### 10) Set up Bootloader
**For GRUB:**
1. Edit `/etc/default/grub` 
	- If you want to dual boot, uncomment this line: `$ GRUB_DISABLE_OS_PROBER=false`
	- Additionally configure other stuff like GRUB_TIMEOUT, and GRUB_DEFAULT...

2. Run `$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`

3. Run `$ grub-mkconfig -o /boot/grub/grub.cfg`

**For bootctl**:
https://wiki.archlinux.org/title/Systemd-boot#Installing_the_UEFI_boot_manager

### 11. Reboot
1. Type `$ exit` to exit from the chroot, then type `reboot`, and cross your fingers
2. Unplug the USB flash
3. If you installed the system successfully, you'll see GRUB
4. Log in as 'root' with the password you set
5. Enjoy!

----

## Post-Install

### Check Internet connection
For NetworkManager, use `$ nmtui` or `$ nmcli` to connect to a network.

### System update
`$ sudo pacman -Syu`

### Install an AUR helper
**yay**
1. Install `git` and `base-devel` if you don't have them
2. `$ git clone https://aur.archlinux.org/yay.git`
3. `$ cd yay`
4. `$ makepkg -si`
5. Check if yay works by `$ yay --version`
6. You can delete the `yay` folder 

### Add your user
1. Create your user with a home directory:
   `$ useradd <name> -m`
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
4. Make yourself a sudoer:
   - `$ EDITOR=vim sudoedit /etc/sudoers`
   ```
     %wheel ALL=(ALL:ALL) ALL
     <your username> ALL = (ALL:ALL) ALL
   ```

### Graphical Interface
- Install a greeter like `lightdm` or `sddm`
- Enable your greeter (i.e. `$ systemctl enable sddm`)
- **Sway on Wayland**
   1. Install `wayland` + `xorg-xwayland` + `sway`
   2. Your greeter should recognize it as a session after a reboot
   > NOTE: sway won't launch with Nvidia GPUs unless passing it `--unsupported-gpu` (set it in `/usr/share/wayland-sessions/sway.desktop`)
- Reboot

### Audio
1. Install `pipewire pipewire-pulse wireplumber`
2. Enable WirePlumber: `systemctl --user --now enable wireplumber`
> NOTE: if no sound devices are detected, you may need `sof-firmware` (for newer soundcards)

### Bluetooth
1. install `bluez` and `bluez-utils`
2. enable/start `bluetoothd.service`

### Laptops & Battery
Install `tlp`, it drastically optimizes battery
TODO: upower?
