# Guide-to-muxless-hybrid-laptops-for-arch-hyprland-and-prime

Arch Linux Hybrid Graphics (NVIDIA + Zen + Wayland)
This repository documents a high-performance, power-efficient configuration for Arch Linux using the Linux Zen kernel, Hyprland/Wayland, and NVIDIA Prime offloading.

The goal of this setup is to allow the iGPU to handle all routine desktop tasks while keeping the NVIDIA dGPU in a low-power state unless explicitly called via prime-run.

🛠 Prerequisites
Core Packages
Kernel: linux-zen & linux-zen-headers

Drivers: nvidia-open-dkms (Open-source kernel modules)

Tools: nvidia-prime, mesa-utils

System State Management
Filesystem: BTRFS

⚙️ Configuration
1. Bootloader (GRUB)
To ensure proper Wayland support and prevent the dGPU from waking up unnecessarily for the framebuffer, use the following kernel parameters:

File: /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="... nvidia_drm.fbdev=0"
[!IMPORTANT]
nvidia_drm.fbdev=0 is critical. Setting this to 1 can prevent the GPU from entering deep sleep states.
🔋 Power Management Notes
Suspend/Resume: nvidia.NVreg_PreserveVideoMemoryAllocations=1 is currently disabled as it causes instability during the login phase and suspend cycles on this specific hardware.

2. Display Manager (SDDM)
Force SDDM to use Wayland to avoid X11's legacy power management issues which often keep the dGPU "awake."

File: /etc/sddm.conf.d/wayland.conf

Ini, TOML
[General]
DisplayServer=wayland

3. Environment Variables (UWSM / Hyprland)
These variables configure how applications interact with your drivers. By commenting out the GLX vendor library, we ensure the iGPU is the primary renderer.

File: .config/uwsm/env-hyprland


export GBM_BACKEND=nvidia-drm
# export __GLX_VENDOR_LIBRARY_NAME=nvidia # Commented to allow iGPU rendering by default
export LIBVA_DRIVER_NAME=nvidia
export XDG_SESSION_TYPE=wayland
export QT_QPA_PLATFORMTHEME=qt5ct

# Multi-GPU Priority (iGPU:dGPU)
export AQ_DRM_DEVICES=/dev/dri/card1:/dev/dri/card0

# Cursor Theming
export XCURSOR_SIZE=24
export HYPRCURSOR_SIZE=24
export XCURSOR_THEME=Adwaita
export HYPRCURSOR_THEME=Adwaita
4. Initramfs (mkinitcpio)
Configured for BTRFS and standard encryption hooks.

File: /etc/mkinitcpio.conf

MODULES=(btrfs)
HOOKS=(base udev autodetect microcode modconf kms keyboard consolefont block encrypt filesystem fsck) #this setup is for encrypted drive


Low Power State: Avoid forcing global "Low Power" modes via third-party linux-drivers if you notice the GPU waking up unexpectedly; the current Prime-offload method handles the dGPU sleep state more reliably.

🚀 Usage
Everything renders on the iGPU by default. To launch a specific application (like a game or Blender) on the NVIDIA GPU, use:

`
prime-run <application-name>
`
To verify which GPU is being used:

`
Bash
glxinfo | grep "OpenGL renderer"          # Should show iGPU
prime-run glxinfo | grep "OpenGL renderer" # Should show NVIDIA
`
