# Barni Sway guide on Arch
This is not a flushed-out guide, use as a reference only


## Install Sway
1. Install `wayland`, `xorg-wayland`, `sway` base packages, it should be automatically recognized by `sddm`
2. If you're using an **Nvidia** card, launch sway with `--unsupported-gpu` or it won't launch (add that argument in: `/usr/share/wayland-sessions/sway.desktop`)

### 1. Background
Install: `swaybg`
Command from sway config: `output "*" bg <image> fill`

### 2. Screenshots
Install:
- `grim`: for capturing screen
- `slurp`: to select area
- `wl_clipboard`: to copy to system clipboard

Commands:
- `grim -g "$(slurp)" - | wl-copy` (select area)
- `grim -g - | wl-copy` (full screen)

### 3. OBS Screen Capture
Install: `wlrobs-hg`

### 4. HDMI Control \[TODO]
In theory `xrandr` can be replaced by  `wlr-randr`, however I haven't tested it yet

### 5. Redshift
Install: `redshift-wayland-git`

### 6. Status Bar
`Polybar` doesn't really play well with sway, so use `waybar` instead.

### 7. Brave
Brave didn't launch for me out of the box, fixed with these arguments:
`--enable-features=UseOzonePlatform --ozone-platform=wayland`

## Nvidia
> NOTE: this is using laptop dual GPU setup (intel iGPU + Nvidia dGPU)

Currently installed packages
`lib32-nvidia-utils`
`nvidia`
`nvidia-prime`
`nvidia-settings`
`nvidia-utils`
`lib32-vulkan-intel`
`libva-intel-driver`
`xf86-video-intel`

Sway by default uses the iGPU, to run an app with the Nvidia card, use `$ prime-run <command>`.
That simply sets 3 environment variables, check it with `$ cat /bin/prime-run`:
```
#!/bin/bash
__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia "$@"
```
So set these for apps to run them on the Nvidia card.

### Steam
To launch a Steam game with the Nvidia card, open the game's **launch options** and set:
`__NV_PRIME_RENDER_OFFLOAD=1 __VK_LAYER_NV_optimus=NVIDIA_only __GLX_VENDOR_LIBRARY_NAME=nvidia %command%`
(tested on SM and TF2)

