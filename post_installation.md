# Arch Linux Full Setup Guide (with Yay and Common Packages)

This guide covers everything from installing `yay` to setting up your core environment — grouped by purpose for clarity.

---

## 1. Install `yay`

`yay` helps you install both official and AUR packages easily.

### Step 1: Prepare build tools

```bash
sudo pacman -S --needed base base-devel git
```

### Step 2: Build and install yay

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

After this, `yay` will be ready to install all other software.

---

## 2. Terminals & Shell Enhancements

```bash
yay -S alacritty kitty wezterm-git starship tmux thefuck
```

**Purpose:** Terminals and shell improvements.

* GPU-accelerated terminals for performance.
* `starship` gives a fast, pretty shell prompt.
* `tmux` lets you manage multiple sessions.
* `thefuck` corrects mistyped commands.

Set **Kitty** as your default terminal (optional):

```bash
xdg-mime default kitty.desktop x-scheme-handler/terminal
```

---

## 3. Development Tools

```bash
yay -S git lazygit rust nodejs npm python-pip cmake nasm
```

**Purpose:** Languages, compilers, and utilities.

* Git for version control; LazyGit for TUI.
* Rust, Node, Python for development.
* CMake and NASM for building and compiling.

---

## 4. System Utilities

```bash
yay -S bat btop eza fastfetch htop lsd 7zip wget ufw power-profiles-daemon
```

**Purpose:** Everyday utilities.

* Enhanced file listing, system info, compression, and firewall tools.
* `ufw` is the Uncomplicated Firewall.

Enable firewall:

```bash
sudo systemctl enable --now ufw
sudo ufw enable
```

---

## 5. Multimedia & Creative Tools

```bash
yay -S vlc pavucontrol spotify pinta ffmpegthumbnailer
```

**Purpose:** Video, audio, and media tools.

* `vlc`: Default video player.
* `pavucontrol`: Manage audio devices.
* `spotify`: Stream music.
* `pinta`: Simple image editor.
* `ffmpegthumbnailer`: Generate thumbnails.

Set defaults:

```bash
xdg-mime default vlc.desktop video/*
```

---

## 6. File Management & GUI Tools

```bash
yay -S dolphin dolphin-plugins nautilus nautilus-open-any-terminal
```

**Purpose:** File managers and GUI helpers.

* `nautilus` (GNOME) or `dolphin` (KDE) for browsing files.
* Integrate terminal access in Nautilus.

Set Nautilus as default:

```bash
xdg-mime default org.gnome.Nautilus.desktop inode/directory
```

---

## 7. Wayland / Hyprland Environment

```bash
yay -S hyprland rofi-wayland waybar swaync wlogout wofi xdg-desktop-portal-hyprland xdg-desktop-portal xdg-desktop-portal-gtk xdg-desktop-portal-wlr wireplumber pipewire pipewire-alsa pipewire-pulse pipewire-jack xorg-xwayland qt5-wayland qt6-wayland grim slurp wl-clipboard wf-recorder brightnessctl polkit-gnome swaybg swaylock-effects kanshi mako wlsunset gammastep
```

**Purpose:** Full Wayland desktop environment and all dependencies for Hyprland.

* `hyprland`: Wayland compositor.
* `rofi-wayland`, `wofi`: Application launchers.
* `waybar`, `swaync`: Status bar and notifications.
* `pipewire`, `wireplumber`: Audio and multimedia backend.
* `xdg-desktop-portal-*`: Portal support for apps and screensharing.
* `xorg-xwayland`: Compatibility layer for X11 apps.
* `grim`, `slurp`, `wl-clipboard`: Screenshots and clipboard tools.
* `wf-recorder`: Screen recording.
* `brightnessctl`: Control screen brightness.
* `polkit-gnome`: Authentication dialogs.
* `swaybg`, `swaylock-effects`, `mako`: Background, lock screen, and notification utilities.
* `kanshi`: Auto display profile manager.
* `wlsunset`, `gammastep`: Screen temperature adjustment for comfort.

---

## 8. Networking Tools

```bash
yay -S networkmanager network-manager-applet protonvpn-cli-community-git
```

**Purpose:** Wi-Fi and VPN management.

* `networkmanager` for managing connections.
* `network-manager-applet` adds a GUI.
* `protonvpn` for secure browsing.

Enable network services:

```bash
sudo systemctl enable --now NetworkManager
```

---

## 9. Fonts, Icons, and Theming

```bash
yay -S papirus-icon-theme bibata-cursor-theme-bin noto-fonts otf-font-awesome ttf-fira-code ttf-firacode-nerd ttf-fira-sans ttf-dejavu
```

**Purpose:** Improve visuals and readability.

* Icon and cursor themes.
* Developer-friendly and aesthetic fonts.

---

## 10. Virtualization and Emulation

```bash
yay -S qemu-base qemu-full
```

**Purpose:** Virtual machine and emulation setup.

Enable necessary services:

```bash
sudo systemctl enable --now libvirtd
```

---

## 11. Miscellaneous Enhancements

```bash
yay -S touche touchegg gestures trash-cli vim yazi
```

**Purpose:** Touch gestures, text editing, and CLI utilities.

* `touchegg` for gesture support.
* `trash-cli` provides safe file deletion.
* `vim` and `yazi` for editing and file browsing.

---

## 12. Optional Drivers

### CPU Microcode

```bash
sudo pacman -S amd-ucode   # for AMD
sudo pacman -S intel-ucode # for Intel
```

### GPU Drivers

| GPU Type        | Command                                          |
| --------------- | ------------------------------------------------ |
| AMD             | `sudo pacman -S xf86-video-amdgpu vulkan-radeon` |
| Intel           | `sudo pacman -S mesa vulkan-intel`               |
| NVIDIA (open)   | `sudo pacman -S nvidia-open`                     |
| NVIDIA (legacy) | `sudo pacman -S nvidia nvidia-utils`             |

---

## 13. Finishing Up

Update everything:

```bash
yay -Syu
```

Clean up unnecessary dependencies:

```bash
yay -Yc
```

Reboot your system to apply all settings:

```bash
sudo reboot
```

---

You now have a fully functional Arch Linux environment — developer-ready, multimedia-friendly, and visually refined. Read the Arch Wiki often; it’s your best reference for deeper customization.
