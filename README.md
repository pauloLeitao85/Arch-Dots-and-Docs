# Arch-Dots-and-Docs
A battle-tested guide for Arch Linux on Btrfs with LUKS encryption, featuring the "Surgical Rollback" method for bulletproof system recovery.

Arch-Surgical-Btrfs 🚀
Welcome to my personal blueprint for a resilient, self-healing Arch Linux workstation. This guide focuses on a Flat Subvolume Layout and a Surgical Rollback methodology that ensures you can recover from a broken update or a configuration error in seconds, even without a Live USB.

✨ Key Features
Architectural Resilience: Uses a flat Btrfs layout (@, @home, @snapshots) so your data remains safe even if the OS is wiped.

The "Manual Hero" Method: A reliable, step-by-step procedure to physically swap broken subvolumes with healthy snapshots.

GRUB Emergency Exit: Fully integrated boot menu allowing you to boot into read-only snapshots via OverlayFS.

Automated Protection: Pre-configured Snapper timers and snap-pac hooks for "set and forget" safety.

Surface Pro Optimized: Tailored for the Microsoft Surface Pro (LUKS encryption and kernel synchronization).

🛠️ Hardware Context
This guide was developed and tested on a Microsoft Surface Pro, though the Btrfs logic applies to any modern Arch Linux machine.

⚖️ Disclaimer
I am a Linux enthusiast, not a professional. This guide is a record of what works for my system. Use it as a reference, but always double-check the Arch Wiki.
