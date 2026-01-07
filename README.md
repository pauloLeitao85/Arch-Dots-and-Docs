A battle-tested blueprint for a resilient, self-healing Arch Linux workstation. This guide focuses on a Flat Subvolume Layout and a Surgical Rollback methodology, ensuring you can recover from a broken system in seconds—even without a Live USB.

Disclaimer: I am a Linux enthusiast, not a professional. This guide is a record of what works for my system (Surface Pro). Use it as a reference, but always double-check the Arch Wiki.

✨ Key Features
Architectural Resilience: A flat Btrfs layout (@, @home, @snapshots) ensures data safety even during root restoration.

The "Manual Hero" Method: Step-by-step physical subvolume swapping for guaranteed recovery.

GRUB Emergency Exit: Boot into read-only snapshots via OverlayFS directly from the boot menu.

Automated Protection: Integrated Snapper timers and snap-pac hooks for effortless system safety.

Phase 1: Preparation
1. Verification
Ensure your ISO is authentic before flashing.

Bash

# Integrity Check
sha256sum -c sha256sums.txt

# Authenticity Check
gpg --keyserver-options auto-key-retrieve --verify archlinux-version.iso.sig
2. Create Bootable USB
Identify your drive with lsblk, then flash:

Bash

sudo dd bs=4M if=archlinux.iso of=/dev/sdX conv=fsync oflag=direct status=progress
Phase 2: Partitioning & Encryption
1. Layout
Use cfdisk /dev/nvme0n1 to create:

p1: 1 GiB (EFI System)

p2: Remainder (Linux root x86-64)

2. Encryption (LUKS2)
Bash

# Initialize encryption
cryptsetup luksFormat /dev/nvme0n1p2

# Open the encrypted vault
cryptsetup open /dev/nvme0n1p2 cryptroot
Phase 3: Btrfs Subvolume Layout
We use a Flat Layout where snapshots are siblings to the root, preventing recursive snapshotting and allowing for easy swapping.

1. Create Subvolumes
Bash

mount /dev/mapper/cryptroot /mnt
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg_cache
umount /mnt
2. Optimized Mounting
Bash

# Mount Root with performance flags
mount -o noatime,compress=zstd,ssd,discard=async,subvol=@ /dev/mapper/cryptroot /mnt

# Create mount points
mkdir -p /mnt/{home,.snapshots,var/log,var/cache/pacman/pkg,boot}

# Mount siblings
mount -o noatime,compress=zstd,subvol=@home /dev/mapper/cryptroot /mnt/home
mount -o noatime,compress=zstd,subvol=@snapshots /dev/mapper/cryptroot /mnt/.snapshots
mount -o noatime,compress=zstd,subvol=@log /dev/mapper/cryptroot /mnt/var/log
mount -o noatime,compress=zstd,subvol=@pkg_cache /dev/mapper/cryptroot /mnt/var/cache/pacman/pkg
mount /dev/nvme0n1p1 /mnt/boot
Phase 4: System Installation
1. Pacstrap
Bash

pacstrap -K /mnt base linux linux-firmware linux-firmware-marvell btrfs-progs neovim networkmanager sudo
2. Configuration
Bash

genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
Phase 5: The "Undo" Button (Snapper)
1. Initialization
Bash

# Remove default folder
umount /.snapshots
rm -r /.snapshots

# Create snapper config
snapper -c root create-config /

# Link to our dedicated @snapshots subvolume
btrfs subvolume delete /.snapshots
mkdir /.snapshots
mount -a
chmod 750 /.snapshots
2. Bootable Snapshots (GRUB)
To boot read-only snapshots, you must add the OverlayFS hook.

Edit /etc/mkinitcpio.conf: HOOKS=(... block sd-encrypt filesystems fsck grub-btrfs-overlayfs)

Update grub-btrfsd to watch the correct path: sudo systemctl edit --full grub-btrfsd -> ExecStart=/usr/bin/grub-btrfsd --syslog /.snapshots

🆘 The "Surgical Rollback" (Manual Hero)
If your system breaks, use this procedure to "travel back in time":

Mount Master (ID 5): mount -o subvolid=5 /dev/mapper/cryptroot /mnt

Quarantine: mv /mnt/@ /mnt/@_broken

Restore: btrfs subvolume snapshot /mnt/@snapshots/NUMBER/snapshot /mnt/@

Reboot & Clean Up:

Bash

# Fix the "Ghost" Pacman Lock
sudo rm /var/lib/pacman/db.lck
# Fix the "Kernel Gap"
sudo pacman -S linux linux-headers
🏁 Pro-Tip for your GitHub
To get your images working:

In your GitHub repository, create a folder named img.

Upload your screenshots there.

Update the links in this README to: ![Description](img/your-image.png).

Would you like me to help you write a "Daily Maintenance" section for Phase 10 to cover things like Btrfs scrubbing and balancing?
