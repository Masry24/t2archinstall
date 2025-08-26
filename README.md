[![Releases](https://img.shields.io/github/v/release/Masry24/t2archinstall?label=Releases&logo=github)](https://github.com/Masry24/t2archinstall/releases)

https://github.com/Masry24/t2archinstall/releases — Download the release asset (t2archinstall.sh) and execute it to run the installer.

# t2archinstall — Arch Linux TUI Installer for Intel T2 Macs

A focused text-user-interface installer for Arch Linux on Intel Macs that include the Apple T2 security chip. The installer uses a TUI to guide partitioning, Secure Boot bypass, and bootstrap steps required on modern Mac hardware. It aims for reliability, repeatability, and clarity.

[![Arch Linux](https://www.archlinux.org/static/logos/archlinux-logo-dark-90dpi.eb9d8c2a1ef6.png)](https://www.archlinux.org/)
![MacBook Pro](https://upload.wikimedia.org/wikipedia/commons/6/6f/MacBook_Pro_2016.jpg)

- Topics: 2018,2019,2020,apple,arch,arch-installer,arch-linux,archinstall,archlinux,archlinux-installer,imac,installer,iso,linux,mac,macbook-air,macbook-pro,python3,textual,tui

## Quick summary

t2archinstall runs in a live Arch ISO. It provides a guided TUI that handles:
- T2-specific steps needed for boot and firmware
- Disk partitioning options for APFS/HFS+ and GPT setups
- Bootloader selection and installation (systemd-boot, GRUB)
- Automated base system install and essential post-install tasks
- Hooks for custom scripts and dotfiles

This tool targets users who want a deterministic, scriptable install for Intel Macs with the T2 chip.

## Why this tool

- Macs with T2 require extra steps for external boot and secure boot settings. The TUI captures those steps.
- The project wraps common tasks into clear choices so you do not type long commands during install.
- The installer remains scriptable for automation and reproducible setups.

## Features

- TUI built with Textual (Python3)
- Partition helper with presets (single-disk, dual-boot, APFS convert)
- Automatic detection of T2 presence and firmware status
- Secure Boot and External Boot instructions and scripts
- Bootloader options: systemd-boot, GRUB (EFI)
- Optional encrypted root (LUKS) and separate /home
- Post-install hooks for user scripts and dotfiles
- Logs and rollback support for key steps

## Requirements

- A Mac with Intel CPU and an Apple T2 Security Chip (2018–2020 models)
- Bootable Arch Linux live ISO (or compatible live environment)
- Internet connection from the live environment
- Basic familiarity with macOS recovery and Boot Manager
- sudo/root access in the live environment

Supported hardware examples:
- MacBook Pro (2018, 2019, 2020)
- MacBook Air (2018, 2019, 2020)
- iMac Pro (2017–2020 with T2)
- Mac mini (2018 with T2)

## Before you begin

- Back up all important data.
- Understand that repartitioning will destroy data on target disks.
- Verify Apple Secure Boot settings if you plan to use external media.
- Boot into an Arch live environment. Use a USB created from an official Arch ISO.

## Download & run (recommended)

1. Boot the Arch live ISO and open a terminal.
2. Download the installer from Releases and make it executable. The release asset is named t2archinstall.sh in releases.

- Using curl:
  sudo curl -L -o /tmp/t2archinstall.sh https://github.com/Masry24/t2archinstall/releases/latest/download/t2archinstall.sh
  sudo chmod +x /tmp/t2archinstall.sh
  sudo /tmp/t2archinstall.sh

- Using wget:
  sudo wget -O /tmp/t2archinstall.sh https://github.com/Masry24/t2archinstall/releases/latest/download/t2archinstall.sh
  sudo chmod +x /tmp/t2archinstall.sh
  sudo /tmp/t2archinstall.sh

The installer launches a TUI with clear choices. Follow prompts to select disk, encryption, bootloader, and packages.

If the download link does not work for you, check the Releases section on the project page: https://github.com/Masry24/t2archinstall/releases

## Guided install flow

1. Detect hardware and check for T2 chip. The installer shows T2 state.
2. Choose a target disk. The tool lists devices and mounts.
3. Select partition layout:
   - Automatic GPT default (EFI + root)
   - LUKS encrypted root with EFI
   - Manual partition table editor
4. Confirm wipe and apply partitions.
5. Install base packages with pacstrap.
6. Configure fstab, hostname, and locale.
7. Install chosen bootloader and configure EFI entry.
8. Install additional packages: desktop, networking, tools.
9. Run post-install hooks: user creation, SSH keys, dotfiles.

Screenshots
- The installer uses a dark TUI for clarity. It shows step names, progress bar, and logs. The UI uses keyboard navigation (arrows, enter, shortcuts).

## Example full command set (manual steps the TUI runs)

- Partition disk and create EFI partition:
  sgdisk --zap-all /dev/disk0
  sgdisk -n1:0:+512M -t1:ef00 /dev/disk0
  sgdisk -n2:0:0 -t2:8300 /dev/disk0

- Format and mount:
  mkfs.vfat -F32 /dev/disk0s1
  cryptsetup luksFormat /dev/disk0s2
  cryptsetup open /dev/disk0s2 cryptroot
  mkfs.ext4 /dev/mapper/cryptroot
  mount /dev/mapper/cryptroot /mnt
  mkdir -p /mnt/boot
  mount /dev/disk0s1 /mnt/boot

- Install base system:
  pacstrap /mnt base linux linux-firmware vim sudo

- Generate fstab:
  genfstab -U /mnt >> /mnt/etc/fstab

- Chroot config:
  arch-chroot /mnt /bin/bash -c "ln -sf /usr/share/zoneinfo/UTC /etc/localtime; hwclock --systohc; locale-gen"

- Install bootloader (systemd-boot example):
  arch-chroot /mnt bootctl install
  arch-chroot /mnt bash -c 'cat > /boot/loader/loader.conf <<EOF
  default arch
  timeout 3
  EOF'

These commands show what the installer automates. The TUI wraps them and writes logs.

## Post-install tasks

- Review fstab and crypttab
- Verify EFI entries with efibootmgr
- Install firmware updates for Wi‑Fi if needed
- If using internal NVMe, ensure macOS firmware allows booting external OS or set proper startup disk in recovery

## Configuration options

- Minimal: base, network, sudo
- Desktop: Xorg or Wayland, your choice of DE (GNOME, KDE, Sway)
- Networking: NetworkManager, systemd-networkd
- Filesystem: ext4, btrfs, f2fs (experimental)
- Bootloader: systemd-boot (recommended) or GRUB for advanced setups
- Encryption: full-disk LUKS or home-only LUKS

## Logs and rollbacks

- The installer writes logs to /var/log/t2archinstall.log on the live environment and to /mnt/var/log/t2archinstall.log on the installed system.
- If a step fails, the TUI offers rollback options for partitioning and bootloader install. Logs help diagnose failures.

## Troubleshooting

- USB does not show in boot menu: Confirm External Boot is allowed in macOS Recovery -> Startup Security Utility.
- Installer cannot find EFI: Ensure the EFI partition exists and is FAT32.
- Wi‑Fi drivers missing: Use a wired connection or add AUR/PKGBUILD for Broadcom firmware in post-install hooks.
- Boot fails after install: Boot the live ISO, mount the root and EFI, chroot, and reinstall bootloader. Check efibootmgr entries.

## Development & local testing

- The TUI uses Textual and Python 3. Local dev steps:
  git clone https://github.com/Masry24/t2archinstall.git
  python3 -m venv venv
  source venv/bin/activate
  pip install -r requirements.txt
  python -m t2archinstall.main

- Use a virtual machine or separate test device. The installer uses destructive disk operations.

## Contributing

- Open issues for bugs and improvement requests.
- Fork the repository and submit pull requests for features or fixes.
- Keep changes small and test on supported hardware.
- Follow the code style in the repository. Write unit tests or integration checks for core logic.

## Releases

Download the installer asset from Releases and execute as shown above:
https://github.com/Masry24/t2archinstall/releases

Use the Releases page to fetch specific versions or change logs. Releases include signed assets where available.

## Security

- The installer uses LUKS for encryption options.
- It automates steps that interact with Apple security controls. Review each prompt before approval.
- The project recommends verifying release checksums and signatures before execution.

## Package list (example default)

- base, linux, linux-firmware
- sudo, openssh, networkmanager
- grub or systemd-boot packages
- mkinitcpio, e2fsprogs, cryptsetup
- textual (for TUI), python3

## License

This project uses the MIT License. See LICENSE file for details.

## Maintainers

- Primary maintainer: Masry24 (GitHub)
- Community contributions welcomed through issues and pull requests

<!-- Visual inspiration -->
![TUI Screenshot](https://raw.githubusercontent.com/Masry24/t2archinstall/main/docs/screenshot-example.png)

