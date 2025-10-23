# Installation of Arch Linux

First of all, you need a bootable USB stick. Download the ISO file from [Arch downloads](https://archlinux.org/download/) and burn it to a USB stick using something like **balenaEtcher**.

## Booting Arch Linux

Boot the USB stick and select the Arch Linux installation medium.

### Disable Secure Boot

Arch Linux installation images do not support Secure Boot. You need to [disable Secure Boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot#Disabling_Secure_Boot) to boot the installation medium. If desired, Secure Boot can be re-enabled and configured after installation.

After booting, select the installation type:

* **Arch Linux install medium** if using the ISO.
* **Boot Arch Linux** after choosing a mirror if using the Netboot image.

If you see the Arch terminal prompt, congratulations — you’ve entered the Archway.

---

## Disk Partitioning Overview

Before diving in, here’s how the `fdisk` tool works. It’s a command-line utility for managing disk partitions. When you start it with `fdisk /dev/sda`, you’ll enter an interactive session. You can press `m` at any time to display a list of all available commands. A simple step-by-step flow to create partitions is:

1. Type `n` to create a new partition.
2. Enter the partition number (start with 1 for the first partition).
3. Press Enter to accept the default starting sector.
4. Type `+<size>` (for example `+512M` or `+16G`) to define the partition size.
5. Repeat `n` for additional partitions.
6. Use `t` to change the partition type (for swap, EFI, etc.).
7. Type `p` to print and review the table.
8. Finally, type `w` to write changes and exit.

Use `q` instead of `w` if you want to quit without saving changes. Common commands include:

* `n`: Create a new partition
* `d`: Delete a partition
* `p`: Print current partition table
* `t`: Change a partition type
* `w`: Write changes to disk
* `q`: Quit without saving

You can view disks with:

```bash
fdisk -l
```

Then open your target disk:

```bash
fdisk /dev/sda
```

If you’re unsure about partition schemes, read [Partitioning](https://wiki.archlinux.org/title/Partitioning) on the Arch Wiki.

---

## Connecting to the Internet

Internet access is crucial for installation.

### Ethernet

Usually automatic. Test it:

```bash
ping archlinux.org
```

### Wi-Fi (using `iwctl`)

If using Wi-Fi, launch the interactive prompt:

```bash
iwctl
```

Inside `iwctl`:

```bash
device list                # lists wireless devices
station wlan0 scan         # scans for nearby networks
station wlan0 get-networks # shows available SSIDs
station wlan0 connect "SSID"  # connect to your Wi-Fi
exit
```

Test your connection again:

```bash
ping archlinux.org
```

More information: [Iwd Wi-Fi setup](https://wiki.archlinux.org/title/Iwd#iwctl_interactive_use)

Later, once installed, we’ll set up **NetworkManager** for easier network handling.

---

## Setting up the Installation Environment

### Keyboard Layout

Default is US. To list and change layouts:

```bash
ls /usr/share/kbd/keymaps/**/*.map.gz
grep -i "de-latin1" /usr/share/kbd/keymaps/**/*.map.gz
loadkeys de-latin1
```

### Font

For larger fonts on HiDPI screens:

```bash
setfont ter-132b
```

### System Clock

`systemd-timesyncd` syncs time automatically once connected. Check status:

```bash
timedatectl
```

---

## Partitioning for Installation

### Check System Type (UEFI or BIOS)

```bash
ls /sys/firmware/efi/efivars
```

If it exists, you’re on a **UEFI** system. Otherwise, you’re on **BIOS/MBR**.

### UEFI Partitioning

Create a 512MB EFI partition:

```bash
n
1
+512M
```

Change type:

```bash
t
1
```

Optionally, create a swap partition (recommended if you plan to hibernate):

* Equal to RAM size (for hibernation)
* Half your RAM for >16GB systems

```bash
n
2
+16G
t
19
```

Finally, create root partition:

```bash
n
3
<enter>
<enter>
```

### BIOS/MBR Partitioning

For BIOS systems, you'll be using the traditional Master Boot Record (MBR) scheme instead of GPT. This means your disk will store its partition table differently, but the concept of root and swap partitions remains the same.

When you enter `fdisk /dev/sda`, you'll start the interactive partitioning mode. Each time you press `n`, you're creating a new partition. The tool will ask for a partition number — starting with **1** for the first partition. That’s why we enter `1` first: it becomes `/dev/sda1`.

Next, when you type `+16G`, you’re telling `fdisk` to make this partition 16 gigabytes in size. You can adjust this depending on your available disk space — it can be larger or smaller. This partition will usually serve as **swap** if you set its type to Linux swap later.

Then, you’ll create another partition (by typing `n` again and choosing the next number) to use the remaining space for the **root filesystem**, which holds all your system files.

You don’t need an EFI partition here because BIOS systems don’t use it. The BIOS simply looks for the bootloader code in the first sector of your disk (the MBR), which is where GRUB will be installed.

If you’re unsure about how much swap you need, the general rule is:

* Equal to your RAM size if you want hibernation.
* Half of your RAM if you have more than 16GB.

Refer to [Swap on Arch Wiki](https://wiki.archlinux.org/title/Swap) for guidance on sizing and setup.

```bash
n
1
+16G
t
19
n
2
<enter>
<enter>
```

---

## Formatting Partitions

For UEFI:

```bash
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
mkfs.ext4 /dev/sda3
```

For BIOS:

```bash
mkswap /dev/sda1
mkfs.ext4 /dev/sda2
```

Enable swap:

```bash
swapon /dev/sda2
```

---

## Mount Filesystems

```bash
mount /dev/sda3 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

---

## Base Installation

```bash
pacstrap -K /mnt base linux linux-firmware
```

GPU support:

```bash
# NVIDIA
yay -S nvidia nvidia-utils cuda
# AMD
pacman -S mesa xf86-video-amdgpu vulkan-radeon
# Intel
pacman -S mesa vulkan-intel intel-ucode
```

---

## Generate Fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

---

## Chroot

```bash
arch-chroot /mnt
```

---

## Region, Time, and Locale

Set your time zone:

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
```

Localization:

```bash
nano /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

---

## Network Configuration

Set hostname and hosts:

```bash
echo "archlinux" > /etc/hostname
cat <<EOF > /etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
EOF
```

Enable NetworkManager:

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

---

## Audio Setup (PipeWire or PulseAudio)

Modern Arch uses PipeWire by default:

```bash
pacman -S pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```

If you prefer PulseAudio:

```bash
pacman -S pulseaudio pulseaudio-alsa
```

---

## Display Server and Seat Management

For GUI sessions, you’ll need a **seat management daemon** like `seatd`, used by lightweight compositors (e.g., Sway, Hyprland).

```bash
pacman -S seatd
systemctl enable seatd.service
```

`seatd` handles device access (input, output, etc.) without a full display manager. More: [Seat management](https://wiki.archlinux.org/title/Seat_management)

---

## Bootloader

For UEFI:

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

For BIOS:

```bash
pacman -S grub
grub-install --target=i386-pc /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Post-Installation

Once inside your system, you can install desktop environments like KDE, GNOME, or window managers such as Hyprland or i3. You can install them with pacman as follows:

```bash
# KDE Plasma
yay -S plasma kde-applications sddm
systemctl enable sddm

# GNOME
pacman -S gnome gnome-extra gdm
systemctl enable gdm

# Hyprland (Wayland)
pacman -S hyprland waybar wofi xdg-desktop-portal-hyprland

# i3 Window Manager
pacman -S i3 dmenu i3status i3lock nitrogen picom
```

Refer to their respective [Arch Wiki pages](https://wiki.archlinux.org/title/Desktop_environment) for more detailed instructions and customization tips.

Also, consider enabling services like **SDDM**, **GDM**, or **Ly** for easy session management. You can install them as follows:

```bash
# SDDM (KDE and general use)
pacman -S sddm
systemctl enable sddm

# GDM (GNOME)
pacman -S gdm
systemctl enable gdm

# Ly (lightweight TUI manager)
git clone https://aur.archlinux.org/ly.git
cd ly
makepkg -si
systemctl enable ly.service
```

These display managers let you easily log in, switch between desktop environments, and manage sessions graphically. See [Display manager](https://wiki.archlinux.org/title/Display_manager) and [General recommendations](https://wiki.archlinux.org/title/General_recommendations) for further details and optional configurations.

You now have a working, network-ready, audio-configured Arch Linux base system.
