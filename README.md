# Guide to Muxless Hybrid Laptops for Arch, Hyprland, and Prime

**Arch Linux Hybrid Graphics (NVIDIA + Zen + Wayland)**  
This repository documents a high-performance, power-efficient configuration for Arch Linux using the Linux Zen kernel, Hyprland/Wayland, and NVIDIA Prime offloading.

The goal of this setup is to allow the iGPU to handle all routine desktop tasks while keeping the NVIDIA dGPU in a low-power state unless explicitly called via `prime-run`.

---

## 🛠 Prerequisites

### Core Packages
* **Kernel:** `linux-zen` & `linux-zen-headers`
* **Drivers:** `nvidia-open-dkms` *(Open-source kernel modules)*
* **Tools:** `nvidia-prime`, `mesa-utils`

### System State Management
* **Filesystem:** BTRFS

---

## ⚙️ Configuration

### 1. Bootloader (GRUB)
To ensure proper Wayland support and prevent the dGPU from waking up unnecessarily for the framebuffer, use the following kernel parameters:

**File:** `/etc/default/grub`
```
```grub 
GRUB_CMDLINE_LINUX_DEFAULT="splash loglevel=3" #nvidia_drm.fbdev=0" #nvidia.NVreg_PreserveVideoMemoryAllocations=1" 
```
```

[!IMPORTANT]
nvidia_drm.fbdev=0 is critical. Setting this to 1 can prevent the GPU from entering deep sleep states.

2. Power Management Notes 🔋
Suspend/Resume: nvidia.NVreg_PreserveVideoMemoryAllocations=1 is currently disabled as it causes instability during the login phase and suspend cycles on this specific hardware.
[!NOTE]
Low Power State: Avoid forcing global "Low Power" modes via third-party Linux drivers if you notice the GPU waking up unexpectedly. The current Prime-offload method handles the dGPU sleep state more reliably.

3. Display Manager (SDDM)
Force SDDM to use Wayland to avoid X11's legacy power management issues which often keep the dGPU "awake."
File: /etc/sddm.conf.d/wayland.conf
code
```Ini
[General]
DisplayServer=wayland
```

4. Environment Variables (UWSM / Hyprland)
These variables configure how applications interact with your drivers. By commenting out the GLX vendor library, we ensure the iGPU is the primary renderer.
File: ~/.config/uwsm/env-hyprland
code
```
```
```
export TERMINAL=kitty

export AQ_DRM_DEVICES=/dev/dri/card1:/dev/dri/card0
#export AQ_DRM_DEVICES=/dev/dri/by-path/pci-0000:00:02.0-card:/dev/dri/by-path/pci-0000:01:00.0-card

export NVD_BACKEND=direct


#export GBM_BACKEND=nvidia-drm

export __GLX_VENDOR_LIBRARY_NAME=nvidia
export LIBVA_DRIVER_NAME=nvidia
export XDG_SESSION_TYPE=wayland
export QT_QPA_PLATFORMTHEME=qt5ct

export XDG_MENU_PREFIX=arch-

export XCURSOR_SIZE=24
export HYPRCURSOR_SIZE=24
export XCURSOR_THEME=Adwaita
export HYPRCURSOR_THEME=Adwaita
5. Initramfs (mkinitcpio)
Configured for BTRFS and standard encryption hooks.
File: /etc/mkinitcpio.conf
```
```
```

MODULES=(btrfs i915 nvidia nvidia_modeset nvidia_uvm nvidia_drm)

HOOKS=(base udev autodetect microcode modconf kms keyboard consolefont block encrypt filesystem fsck) # this setup is for encrypted drive
```
🚀 Usage
Everything renders on the iGPU by default. To launch a specific application (like a game or Blender) on the NVIDIA GPU, use:
code
```Bash
prime-run <application-name>
```
To verify which GPU is being used:
code
```Bash
glxinfo | grep "OpenGL renderer"           # Should show iGPU 
prime-run glxinfo | grep "OpenGL renderer" # Should show NVIDIA
```
