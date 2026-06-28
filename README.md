# abiso

Minimal Arch-based ISO builder for ArchBang. Replaces mkarchiso for the live ISO use case — squashfs, syslinux BIOS, systemd-boot UEFI, no PXE, no signing, no erofs. Profile-compatible with archiso.

## Requirements

Build machine packages:

```
arch-install-scripts   # pacstrap, arch-chroot
libisoburn             # xorriso
squashfs-tools         # mksquashfs
syslinux               # BIOS boot binaries
dosfstools             # mkfs.fat for EFI image
mtools                 # mmd, mcopy for EFI image
```

## Usage

Run from inside the ISO profile directory (where `profiledef.sh` lives):

```bash
sudo ./abiso           # standard build
sudo ./abiso -v        # verbose
sudo ./abiso -r        # remove work dir after build
sudo ./abiso -w /tmp/work -o /tmp/out   # custom dirs
```

## Profile compatibility

Fully compatible with existing archiso profiles. Uses:

- `profiledef.sh` — all variables including `bootmodes`, `airootfs_image_tool_options`, `file_permissions`
- `packages.x86_64` — package list
- `airootfs/` — overlay copied into the squashfs
- `syslinux/` — BIOS bootloader config and splash
- `efiboot/` — systemd-boot loader config and entries

## Boot modes

| Mode | Bootloader |
|------|-----------|
| `bios.syslinux` | SYSLINUX / ISOLINUX |
| `uefi.systemd-boot` | systemd-boot |
| `uefi.grub` | GRUB standalone EFI binary |

## Squashfs layout

Uses the standard archiso layout:

```
arch/x86_64/airootfs.sfs
arch/x86_64/airootfs.sha512
```

The initramfs is stripped from the squashfs before packing (rebuilt by mkinitcpio at install time). The kernel and ucode images are kept — the installer needs them.

## Airootfs customisation

Place a `customize_airootfs.sh` at `airootfs/root/customize_airootfs.sh` in your
profile. abiso copies it into the chroot via the overlay stage, runs it under
`arch-chroot`, then removes it.

Typical uses: locale, hostname, timezone, users, services, sudo, pacman tweaks.
The script runs as root inside the target filesystem — standard chroot environment.

If the script is absent the stage is skipped silently.

## Licence

GPL-2.0-only — see [LICENSE](LICENSE).
