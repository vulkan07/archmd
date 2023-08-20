# Quality of Life Solutions Using i3WM 
Version **0.2** By **Barni**

i3WM doesn't come pre-packaged with all the utilities like a desktop environment. This document shows how I
set up most things on my MSI Laptop. These are not universal, some use my scripts, but most of them should work fine.

# Table of Contents
1. [Laptop Battery Optimization](#battery)
2. [Status Bar TODO](#bar)
3. [Multiple Displays TODO](#multiple-displays)
4. [Window Animations: Compositor TODO](#compositor)
5. [Lockscreen](#lockscreen)
6. [Printing](#printers)
7. [Launching Apps](#launching-apps)
8. [Disk Management](#disk-management)
9. [Blue Light Filter](#red-filter)
9. [My i3 Keybinds](#keybinds)

## <a name="battery"/> Laptop Battery Optimization
Linux by default is not optimized for energy consumption, and will drain your battery quick.

### tlp
Luckily, simply installing **tlp** will go a long way. This module disables unused hardware and optimizes stuff in the OS.
1. Simply install **tlp**. Done.

### cpupower
If you want further battery lifetime, you need to tune down your CPU. Linux and tlp won't slow it down by default.
1. Install **cpupower**
2. Usage `cpupower frequency-set --g <governor>` (Needs root privileges)

   A **governor** is basically a power-plan for a CPU. Most CPUs will have `performance` and `powersave`.
   You can check the governors of you CPU with `cpupower frequency-info --governors`.
    
4. You can set a udev rule to use a power saving governor when unplugged, or run this command manually, or in a power management script.

## <a name="bar"/> Status Bar
TODO polybar

## <a name="multiple-displays"/> Multiple displays
TODO

This .sh script works with **polybar** and **feh**, and simply allows to set the side the HDMI is on.
Syntax: `hdmi <l/r/u/d/m/off>`
> Note: You may have different display names, check with xrandr. 
```sh
if [ $# -eq 0 ]; then
    echo "Usage: hdmi [position] <primary> <secondary>"
    echo "- position: l (left), r (right), t/u (top), b/d (bottom), m/s (mirror), o/off (off)"
    echo "- primary (optional):   Name of primary screen"
    echo "- secondary (optional): Name of secondary screen"
    exit 1
fi

# Set Monitor Names
if [ $# -eq 3 ]; then
	# Names from cmd arguments
	PRIMARY=$2
	SECONDARY=$3
else
	# Default names
	PRIMARY=eDP1
	SECONDARY=HDMI1
fi

# If 1, polybar will be started on all screens
# Mirror mode disables this
POLYBAR=1

# Kill polybar
pkill polybar


# Map single letters to full commands
case $1 in
    l)
        xrandr --output $SECONDARY --auto --left-of $PRIMARY
		echo -e "\e[1;34m$SECONDARY \e[3mleft\e[0m"
        ;;
    r)
        xrandr --output $SECONDARY --auto --right-of $PRIMARY
		echo -e "\e[1;34m$SECONDARY \e[3mright\e[0m"
        ;;
    t | u)
        xrandr --output $SECONDARY --auto --above $PRIMARY
		echo -e "\e[1;34m$SECONDARY \e[3mtop\e[0m"
        ;;
    b | d)
        xrandr --output $SECONDARY --auto --below $PRIMARY
		echo -e "\e[1;34m$SECONDARY \e[3mbottom\e[0m"
        ;;
    o | off )
        xrandr --output $SECONDARY --off
		echo -e "\e[1;34m$SECONDARY \e[3moff\e[0m"
		POLYBAR=0
        ;;
	m | s)
        xrandr --output $SECONDARY --auto --same-as $PRIMARY
		echo -e "\e[1;34m$SECONDARY \e[3mmirrored\e[0m"
		POLYBAR=0
		;;
    *)
		echo -e "\e[0;91mInvalid Option: \e[1;91m$1\e[0m"
        exit 1
        ;;
esac

# Set polybar on all screens (except when mirrored)
if type "xrandr" > /dev/null && [ $POLYBAR -ne 0 ]; then
  for m in $(xrandr --query | grep " connected" | cut -d" " -f1); do
    MONITOR=$m polybar -q --reload &
  done
else
  polybar -q --reload &
fi

# Set wallpaper
feh --bg-scale /home/olahb/Pictures/Wallpaper.jpg

```

## <a name="compositor"/> Window Animations / Compositor
TODO
### picom


## <a name="lockscreen"/> Lockscreen
### i3lock
1. Install **i3lock**, or **i3lock-color** for better customizability.
2. Create a script to feed it with arguments for customization.
3. Use i3lock command to lock your session.
4. Bind a key to it in *.config/i3/config*.


## <a name="printers"/> Printers

### Add user to **lp** group
`$ sudo usermod -aG lp <username>`

### 2. cups
The main program required for printing on linux is **cups**.
1. install **cups**.
2. Enable **`cups.service`** or **`cups.socket`** (The latter will only start cups when a program wants to print)

### 3. Drivers
Before you can print, you need a driver for your printer.

- Most **HP** printers *(This was tested on a SmartTank 530)*:
  1. Install **hplip**.
  2. Run `$ hp-setup` (requires **python-pyqt5** to run).

- Other printer drivers: refer to  [this ArchWiki page](https://wiki.archlinux.org/title/CUPS/Printer-specific_problems#HP).

After installing and setting up the driver, and enabling cups, you should be able to select and use your printer.

## <a name="launching-apps"/> Launching apps from i3
Use **dmenu** or **rofi**. Dmenu will show the selector on the top of the screen, while rofi will be in the middle of the screen. 
### dmenu
1. Install **dmenu**.
2. Run `$ dmenu_run` to check if it works.
3. Bind it in *.config/i3/config* if you want.

> Note: To access **.desktop** applications with dmenu, install **j4-dmenu-desktop**(aur) alongside **dmenu** package, and use the **j4-dmenu-desktop** command instead.

## <a name="disk-management"/> Disk Management (Free up space)
*Traditional commands you can use: `du` and `df` (these are not CLI). Instead,*

Use **ncdu**, a very easy to use command line utility for interactive disk space management.

1. Install **ncdu**.
2. Run `$ ncdu` for your home directory, or run `$ sudo ncdu /` for the whole filesystem.
3. Navigate with arrows, backspace to go up, press **d** to delete stuff.

Also pacman's package caches can take up a few gigabytes. To delete them, run: `$ sudo pacman -Sc` (-Scc for deleting caches of all packages)



## <a name="red-filter"/> Blue Light Filter (Night mode)
If you want your eyes to not burn out on late night programming sessions, use **redshift** for blue light filtering.
1. Install **redshift**.
2. Create and customize config file: `.config/redshift.conf`
```
[redshift]
location-provider=manual

[manual]
lat=47.497913 		## Location of your area
lon=19.040236 		## This example is Budapest

[screen]
temperature-day=5700   	## Set the temperatues (in Kelvin)
temperature-night=1200
```
3. To autostart redshift in background add this to `.config/i3/config`:

`exec --no-startup-id redshift -c <home>/.config/redshift.conf` (replace \<home\>)

## <a name="keybinds"> My i3 Keybinds (Barni)
In **`.config/i3/config`**:
``` i3
### NAVIGATION ###

# Switch between workspaces with Win+Alt+<left/right>
bindsym $mod+Mod1+Left workspace prev_on_output
bindsym $mod+Mod1+Right workspace next_on_output

# Toggle i3 window layout
bindsym $mod+e layout toggle split

# Cycle current workspace between displays
bindsym $mod+u exec "i3-msg move workspace to output next"

### APP LAUNCH HOTKEYS ###

#Brave Win+C
bindsym $mod+c exec brave

#Screenshot Win+P
bindsym $mod+p exec "flameshot gui"

#File Explorer Win+Q
bindsym $mod+q exec --no-startup-id thunar


### OTHER BINDINGS ###

# Lockscreen Win+L (uses custom script)
bindsym $mod+l exec --no-startup-id /home/olahb/.config/scripts/lock

# Touchpad toggle Win+T (uses custom script)
bindsym $mod+t exec --no-startup-id /home/olahb/.config/scripts/touchpad

# Brightness control (uses custom script) (custom for MSI laptop)
bindsym XF86MonBrightnessUp exec --no-startup-id "/home/olahb/.config/scripts/brightness 8"
bindsym XF86MonBrightnessDown exec --no-startup-id "/home/olahb/.config/scripts/brightness -8"
```
