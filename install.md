# Installing Arch Linux v0.1
Installing Arch is not complicated (usually). The thing is that, you have to do most things by yourself,
in a console. But it's certainly achieveable.

> **NOTE: This guide only shows how to install on a UEFI system with GPT. No legacy BIOS or MBR.**

## Getting Started

> NOTE: If you are doing a dual boot system with Windows, install Windows first (if it's not already installed).

> NOTE: If you are doing a dual boot system with Windows, shrink the data partitions to make space for Linux.

> NOTE: If you are doing a dual boot system with Windows, back up all your data! I do not guarantee no data loss.

1. Install an Arch ISO on a USB flash drive. (On Windows: Rufus, on Linux: Etcher)
2. Enter UEFI, and change the boot order to boot into the USB flash drive.

   Note: You may need to disable secure boot or fast boot in the UEFI to successfully boot into the ISO.
3. Change Keyboard Layout: `$ loadkeys hu` for Hungarian.
4. Verify that you've booted in EFI mode

   If `$ ls /sys/firmware/efi/efivars` returns *No such file or Directory*, you're not in EFI mode.
   Do not continue installing. You may need to change the BIOS from legacy boot to EFI boot.
5. Connect to the internet
   - Ethernet should work without any configuration
   - Wifi:
     1. Enter `$ iwctl` (a pre-installed network commandline tool)
     2. `$ device list` lists the available hardware interfaces on your machine. Choose the one you want to use (usually **`wlan0`**) and remember the name.
     4. `$ station <device> scan` scans the nearby Wi-Fi networks.
     5. `$ station <device> get-networks` lists the scanned networks.
     6. `$ station <device> connect <Wi-Fi name>` Connects to the selected Wi-Fi network and will prompt you for the password.
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

Use the **`$ cfdisk <device>`** commandline tool to partition easily.
If you only have one storage device, you do not have to specify the \<device\>.
If you have multiple devices, use `lsblk` command to list them, and select the correct device, not the partition! 
(e.g. `sdb`, not `sdb1`) 

Note: cfdisk does not modify the actual partitions until Write is issued. 

### Linux Install Only

**I. Create partitions**

1. Enter `$ cfdisk <device>`.
2. Delete all partitions.
3. Allocate at least 200MB, but preferably **500MB** for the **`EFI`** Partition. Then Assign the EFI partition type to it.
4. (Optional) Allocate at least half the size of your RAM to a SWAP partition (if you want hibernation)
5. (Optional) Allocate a separate Linux partition for your `/home` directory.
6. Allocate the rest (or any size you want) to the main Linux data partition.
7. Double check the layout, then select option __`write`__ and confrim by typing `yes`. Then Quit.

**II. Format the new partitions**

Remember, you can see the partitions with `lsblk`. (Example: on disk `sda`, the partitions are `sda1`, `sda2`...)
The path for a partition is **`/dev/<partition>`**
Formatting a partition looks like this: **`mkfs.<type> <partition>`**, example: `mkfs.ext4 /dev/sda5`

1. Format EFI partition: `mkfs.fat -F 32 <partition>`
2. Format Swap Partition: `mkswap <partition>`, then `swapon <partition>` to enable it
3. Format Linux partition(s): `mkfs.ext4 <partition>` 


**III. Mount the partitions**
Mounting a partition means you're appending it to the root Linux file system (/). Example:
You want to mount a Data partition to /data, you do `mount /dev/sda3 /data`. This will allow you to access anything on partition sda3 from /data.

1. Mount EFI: `mount <partition> /mnt/boot/efi`
2. Mount Linux: `mount <partition> /mnt`

### Dual Boot with Windows

**I. Create partitions**

1. Enter `$ cfdisk <device>`.
2. Do not touch the Microsoft partitions nor the EFI partition. Use the remaining free diskspace for the following steps.
4. (Optional) Allocate at least half the size of your RAM to a SWAP partition (if you want hibernation)
5. (Optional) Allocate a separate Linux partition for your `/home` directory.
6. Allocate the rest (or any size you want) to the main Linux data partition.
7. Double check the layout, and that you've not messed up the existing partitions, then select option __`write`__ and confrim by typing `yes`. Then Quit.

**II. Format the new partitions**

Remember, you can see the partitions with `lsblk`. (Example: on disk `sda`, the partitions are `sda1`, `sda2`...)
The path for a partition is **`/dev/<partition>`**
Formatting a partition looks like this: **`mkfs.<type> <partition>`**, example: `mkfs.ext4 /dev/sda5`

1. Do not format the EFI or Microsoft partitions!
2. Format Swap Partition: `mkswap <partition>`, then `swapon <partition>` to enable it
3. Format Linux partition(s): `mkfs.ext4 <partition>`


**III. Mount the partitions**

## Installing Arch System
