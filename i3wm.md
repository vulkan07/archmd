# Quality of Life Solutions Using i3WM 
Version **0.1**

# Table of Contents
1. [Laptop Battery Optimization](#battery)
2. [Status Bar](#bar)
3. [Multiple Displays](#multiple-displays)
4. [Window Animations: Compositor](#compositor)
5. [Lockscreen](#lockscreen)
6. [Printing](#printers)
7. [Launching Apps](#launching-apps)
8. [Disk Management](#disk-management)

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
Syntax: `hdmi <l/r/u/d/off>`
```
pkill polybar

xrandr --output HDMI-1 --auto
if [ $# -eq 0 ]; then
	xrandr --output HDMI-1 --same-as eDP-1
elif [ $1 == "r" ]; then
	xrandr --output HDMI-1 --right-of eDP-1
elif [ $1 == "l" ]; then
	xrandr --output HDMI-1 --left-of eDP-1
elif [ $1 == "u" ] || [ $1 == "t" ] || [ $1 == "a" ]; then
	xrandr --output HDMI-1 --above eDP-1
elif [ $1 == "d" ] || [ $1 == "b" ]; then
	xrandr --output HDMI-1 --below eDP-1
elif [ $1 == "off" ]; then
	xrandr --output HDMI-1 --off
fi

if type "xrandr"; then
  for m in $(xrandr --query | grep " connected" | cut -d" " -f1); do
    MONITOR=$m polybar --reload & > /dev/null
  done
else
  polybar --reload &
fi
feh --bg-scale /home/olahb/Pictures/Wallpaper.jpg
echo
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

### 1. cups
The main program required for printing on linux is **cups**.
1. install **cups**.
2. Enable **`cups.service`** or **`cups.socket`** (The latter will only start cups when a program wants to print)

### 2. Drivers
Before you can print, you need a driver for all printers.

- **HP** printers (for most of them atleast):
  1. Install **hplip**.
  2. Run `hp-setup` (requires **python-pyqt5** to run).

- Other printer drivers: refer to  [this ArchWiki page](https://wiki.archlinux.org/title/CUPS/Printer-specific_problems#HP).

After installing and setting up the driver, and enabling cups, you should be able to select and use your printer.

## <a name="launching-apps"/> Launching apps from i3
Use **dmenu** or **rofi**. Dmenu will show the selector on the top of the screen, while rofi will be in the middle of the screen. 
### dmenu
1. Install **dmenu**
2. Run `dmenu_run` to check if it works.
3. Bind it in *.config/i3/config* if you want.

> Note: To access **.desktop** applications with dmenu, install **j4-dmenu-desktop**(aur) alongside **dmenu** package, and use the **j4-dmenu-desktop** command instead.

## <a name="disk-management"/> Disk Management (Free up space)
Use **ncdu**, a very easy to use command line utility for interactive disk space management.
Traditional commands you can also use: `du` and `df` (these are not CLI)
1. Install **ncdu**.
2. Run `$ ncdu` for your home directory, or run `$ sudo ncdu /` for the whole filesystem-
3. Navigate with arrows, backspace to go up, press **d** to delete stuff.
