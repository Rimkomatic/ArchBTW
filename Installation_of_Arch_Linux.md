# Installation of Arch Linux

First of all, you need a bootable USB stick. Download the ISO file from [Arch downloads](https://archlinux.org/download/) and burn it to a USB stick using something like **balenaEtcher**.

## Booting Arch Linux

Boot the USB stick and select the Arch Linux installation medium.

### Disable Secure Boot

Arch Linux installation images do not support Secure Boot. You need to [disable Secure Boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Disabling_Secure_Boot) to boot the installation medium. If desired, Secure Boot can be re-enabled and configured after installation.

After booting the installation medium, select the installation type:

* If you used the ISO, select **Arch Linux install medium** and press **Enter**.
* If you used the Netboot image, choose a geographically close mirror from the Mirror menu, then select **Boot Arch Linux** and press **Enter**.

If you see the Arch terminal environment, congratulations — you’ve entered the Archway.

## Setting up the installation environment

Now we prepare the environment for installation.

### Setting up the keyboard layout

The default layout is **US**. To change it:

```bash
ls /usr/share/kbd/keymaps/**/*.map.gz
grep -i "de-latin1" /usr/share/kbd/keymaps/**/*.map.gz
loadkeys de-latin1
```

For more information, visit [Keyboard configuration](https://wiki.archlinux.org/title/Keyboard_configuration_in_console).

### Setting up the fonts

Console fonts are in `/usr/share/kbd/consolefonts/`. To set a larger HiDPI font:

```bash
setfont ter-132b
```

See [Fonts in console](https://wiki.archlinux.org/title/Linux_console#Fonts) for additional font options.

### Updating the system clock

The live environment enables `systemd-timesyncd` by default. To verify time sync:

```bash
timedatectl
```

Refer to [systemd-timesyncd](https://wiki.archlinux.org/title/Systemd-timesyncd) for more details.

## Connecting to the Internet

For Ethernet, connection is automatic. To verify:

```bash
ping archlinux.org
```

For Wi-Fi:

```bash
iwctl
```

Then in the interactive prompt:

```bash
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "SSID"
exit
```

Check with:

```bash
ping archlinux.org
```

More info at [Wireless setup](https://wiki.archlinux.org/title/Iwd).

## Partitioning the Disk

List available disks:

```bash
fdisk -l
```

Select your target disk:

```bash
fdisk /dev/sda
```

### Check for UEFI system

```bash
ls /sys/firmware/efi/efivars
```

If the directory exists, you have a UEFI system.

Read more: [UEFI](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface)

### EFI Partition (UEFI only)

Create a 512MB EFI partition for systems using UEFI:

```bash
n   # new partition
1   # partition number
+512M
```

Change its type to EFI System:

```bash
t
L   # list types
1   # EFI System
```

You may also create a **swap partition** if you want to enable swap space on UEFI systems. Swap acts as an overflow for RAM and supports hibernation. The recommended size depends on your system:

* Equal to your RAM (if you plan to use hibernation)
* Half of your RAM for systems with more than 16GB of RAM

Example for a 16GB swap partition:

```bash
n   # new partition
2   # partition number
+16G
```

Then set its type to Linux swap:

```bash
t
L   # list types
19  # Linux swap
```

Finally, continue with creating the root partition as usual. See [Swap on Arch Wiki](https://wiki.archlinux.org/title/Swap) for more details.

### BIOS/MBR Systems (Non-UEFI)

If your system does not support UEFI and uses BIOS with MBR partitioning, you can simply create a bootable partition and optionally a **swap partition** to act as virtual memory. The swap partition is used when your system runs out of physical RAM and can help with hibernation and heavy workloads. Typically, a swap space equal to your RAM size (up to 8GB) is recommended, or at least half your RAM for systems with 16GB or more. See [Swap on Arch Wiki](https://wiki.archlinux.org/title/Swap) for more guidance.

Create a small BIOS boot partition of about 1MB before other partitions (used by GRUB for BIOS installs):

```bash
n   # new partition
1   # partition number
+1M
```

Change its type to BIOS boot:

```bash
t
L   # list types
4   # BIOS boot partition
```

Then create the root partition occupying the rest of the space:

```bash
n   # new partition
2   # partition number
<enter>
<enter>
```

The small buffer partition ensures the bootloader has a proper area to store boot code on MBR setups. You can read more at [GRUB BIOS installation](https://wiki.archlinux.org/title/GRUB#BIOS_systems) and [Partitioning for BIOS systems](https://wiki.archlinux.org/title/Partitioning#BIOS_boot_partition).

### Create Root Partition

Create root partition for the rest of the disk:

```bash
n   # new partition
2   # partition number
<enter>
<enter>
```

Then write changes:

```bash
w
```

For advanced partitioning, refer to [Partitioning](https://wiki.archlinux.org/title/Partitioning).

## Formatting the Partitions

### UEFI Systems

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

### BIOS Systems

```bash
mkfs.ext4 /dev/sda1
```

## Mount the File Systems

```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## Installing the Base System

```bash
pacstrap -K /mnt base linux linux-firmware
```

Optionally, for NVIDIA GPUs:

```bash
pacstrap /mnt nvidia nvidia-utils cuda
```

For AMD GPUs:

```bash
pacstrap /mnt mesa xf86-video-amdgpu vulkan-radeon
```

For Intel GPUs:

```bash
pacstrap /mnt mesa vulkan-intel intel-ucode
```

GPU drivers info: [NVIDIA](https://wiki.archlinux.org/title/NVIDIA), [AMD](https://wiki.archlinux.org/title/AMDGPU), [Intel](https://wiki.archlinux.org/title/Intel_graphics)

## Generate Fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

## Chroot into the System

```bash
arch-chroot /mnt
```

## Setting Time Zone

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

## Localization

Edit locale file:

```bash
nano /etc/locale.gen
```

Uncomment your locale (e.g., `en_US.UTF-8 UTF-8`), then:

```bash
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Refer to [Locale configuration](https://wiki.archlinux.org/title/Locale).

## Network Configuration

```bash
echo "archlinux" > /etc/hostname
```

Add entries to `/etc/hosts`:

```bash
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
```

## Set Root Password

```bash
passwd
```

## Install Essential Packages

```bash
pacman -S networkmanager grub efibootmgr vim base-devel linux-headers git
```

Enable NetworkManager:

```bash
systemctl enable NetworkManager
```

## GRUB Bootloader Installation

For UEFI systems:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

For BIOS systems:

```bash
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

More at [GRUB](https://wiki.archlinux.org/title/GRUB).

## Finishing Installation

Exit the chroot and unmount:

```bash
exit
umount -R /mnt
reboot
```

Remove the installation media when prompted. Welcome to Arch Linux!

---

## Post-Installation: Desktop Environment or Window Manager

You can now install your preferred desktop environment (DE) or window manager (WM).

### KDE Plasma

```bash
sudo pacman -S plasma kde-applications sddm
sudo systemctl enable sddm
```

[Arch Wiki: KDE](https://wiki.archlinux.org/title/KDE)

### GNOME

```bash
sudo pacman -S gnome gnome-extra gdm
sudo systemctl enable gdm
```

[Arch Wiki: GNOME](https://wiki.archlinux.org/title/GNOME)

### Hyprland (Wayland-based compositor)

```bash
sudo pacman -S hyprland waybar wofi xdg-desktop-portal-hyprland
```

[Hyprland Wiki](https://wiki.archlinux.org/title/Hyprland)

### i3 Window Manager

```bash
sudo pacman -S i3 dmenu i3status i3lock nitrogen picom
```

[i3 Wiki](https://wiki.archlinux.org/title/I3)

You can switch between them using a display manager.

## Display Managers

### SDDM (for KDE or others)

```bash
sudo pacman -S sddm
sudo systemctl enable sddm
```

### GDM (for GNOME)

```bash
sudo pacman -S gdm
sudo systemctl enable gdm
```

### Ly (TUI display manager)

```bash
git clone https://aur.archlinux.org/ly.git
cd ly
makepkg -si
sudo systemctl enable ly.service
```

See [Display manager](https://wiki.archlinux.org/title/Display_manager) for alternatives.

## Audio Setup

For PipeWire:

```bash
sudo pacman -S pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```

[PipeWire Wiki](https://wiki.archlinux.org/title/PipeWire)

## GPU and CUDA Verification

For NVIDIA:

```bash
nvidia-smi
```

For OpenCL:

```bash
clinfo | grep 'Device'
```

---

At this stage, you have a fully functional, GPU-accelerated, customizable Arch Linux setup with a desktop environment or window manager of your choice.

Further reading:

* [General Recommendations](https://wiki.archlinux.org/title/General_recommendations)
* [Arch User Repository (AUR)](https://wiki.archlinux.org/title/Arch_User_Repository)
