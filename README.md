# Arch Linux Installation

## Phase 1: Preparation

### 1. Gathering the Ingredients

Navigate to the [Arch Linux Download page](https://archlinux.org/download/) and grab these three files. Place them in the same folder:

- **The ISO:** `archlinux-202X.XX.XX-x86_64.iso`
- **The Checksum:** `sha256sums.txt`
- **The Signature:** `archlinux-202X.XX.XX-x86_64.iso.sig`

---
### 2. Verification (The Security Check)

Before touching your USB drive, ensure the file is perfect.

#### A. Integrity Check

Run this to ensure the file wasn't corrupted during the download:

```bash
sha256sum -c sha256sums.txt
```

 **Note:** You may see errors for other files; as long as it says `archlinux-202x...iso: OK`, you are safe.

#### B. Authenticity Check

Run this to ensure the file is officially from the Arch team:

```bash
gpg --keyserver-options auto-key-retrieve --verify archlinux-202X.XX.XX-x86_64.iso.sig
```

 **Success:** Look for `gpg: Good signature from "[Developer Name]"`. Ignore the warning about "not certified with a trusted signature"—that is standard behavior for GPG.

---
### 3. Creating the Bootable USB

Now we move the data from your computer to the USB drive.

#### A. Identify the Drive

Plug in your USB and identify its path. **Be extremely careful here; choosing the wrong drive will erase it.**

 Run:

```bash
lsblk
```

 ***Note***: Your USB is usually /dev/sdb or /dev/sdc.

#### B. The "Imaging" Command

We use the `dd` (Data Duplicator or Disk Destroyer) command. Replace `/dev/sdX` with your actual USB path (e.g., `/dev/sdb`).

##### Preparing the Drive

First, unmount the partition:

```bash
umount /dev/sdb
```

##### Flashing the ISO

Now, we'll use `dd`. This command will take the input file (`if`)—your new Arch ISO—and write it to the output file (`of`)—your USB drive.

**The command structure looks like this:**

```bash
sudo dd bs=4M if=/full/path/archlinux-202X.XX.XX-x86_64.iso of=/dev/sdX conv=fsync oflag=direct status=progress
```

- `bs=4M`: Sets the block size to 4 Megabytes for faster writing. ⚡
- `conv=fsync`: Ensures all data is physically written to the drive before the command finishes. 💾
- `status=progress`: Shows you a live update of the transfer speed and time. 📊

**Note**: `of=/dev/sdX`: Ensure this is the **drive** (e.g. sdb), not a **partition** (e.g. sdb1).

---
