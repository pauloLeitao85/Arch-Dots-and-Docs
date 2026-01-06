# Arch-Dots-and-Docs
A battle-tested guide for Arch Linux on Btrfs with LUKS encryption, featuring the "Surgical Rollback" method for bulletproof system recovery.

Arch-Surgical-Btrfs 🚀
Welcome to my personal blueprint for a resilient, self-healing Arch Linux workstation. This guide focuses on a Flat Subvolume Layout and a Surgical Rollback methodology that ensures you can recover from a broken update or a configuration error in seconds, even without a Live USB.

✨ Key Features
Architectural Resilience: Uses a flat Btrfs layout (@, @home, @snapshots, @var_logs, @var_pkgs) so your data remains safe even if the OS is wiped.

The "Manual Hero" Method: A reliable, step-by-step procedure to physically swap broken subvolumes with healthy snapshots.

GRUB Emergency Exit: Fully integrated boot menu allowing you to boot into read-only snapshots via OverlayFS.

Automated Protection: Pre-configured Snapper timers and snap-pac hooks for "set and forget" safety.

⚖️ Disclaimer
I am a Linux enthusiast, not a professional. This guide is a record of what works for my system. Use it as a reference, but always double-check the Arch Wiki.

# Installation

## Phase 1: Preparation

### 1. Gathering the Ingredients

Navigate to the [Arch Linux Download page](https://archlinux.org/download/) and grab these three files. Place them in the same folder:

<div _ngcontent-ng-c4186646317="" aria-busy="false" aria-live="polite" class="markdown markdown-main-panel stronger enable-updated-hr-color" dir="ltr" id="bkmrk-the-iso%3A%C2%A0archlinux-2" inline-copy-host="">- **The ISO:** `archlinux-202X.XX.XX-x86_64.iso`
- **The Checksum:** `sha256sums.txt`
- **The Signature:** `archlinux-202X.XX.XX-x86_64.iso.sig`

---

</div>### 2. Verification (The Security Check)

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


<div _ngcontent-ng-c4186646317="" aria-busy="false" aria-live="polite" class="markdown markdown-main-panel stronger enable-updated-hr-color" dir="ltr" id="bkmrk-linux%3A-run-lsblk.-yo" inline-copy-host=""></div><div _ngcontent-ng-c4186646317="" aria-busy="false" aria-live="polite" class="markdown markdown-main-panel stronger enable-updated-hr-color" dir="ltr" id="bkmrk-the-iso%3A-archlinux-2" inline-copy-host=""></div>### 4. Booting the ISO

1. Insert your prepared USB drive into the target machine.
2. Restart and enter your **BIOS/UEFI** (usually by tapping `F2`, `F12`, `Del`, or `Esc`).
3. **Disable Secure Boot** (Arch's official ISO doesn't support it out of the box).
4. Select your USB drive as the primary boot device.
5. Choose **"Arch Linux install medium (x86\_64, UEFI)"** from the menu.

---

### 5. Network &amp; Time

#### A. Connecting to Wi-Fi (if needed)

Once you see the `root@archiso #` prompt, you need internet to download the system files.

If you know your device name and network name run this command:

```bash
iwctl --passphrase "YOUR_PASSWORD" station wlan0 connect "YOUR_SSID"
```

If not do this steps:

1. **Enter the utility:** `iwctl`
2. **Identify your device:** `device list` (Usually `wlan0`).
3. **Scan for networks:** `station wlan0 scan`
4. **List networks:** `station wlan0 get-networks`
5. **Connect to your SSID:** `station wlan0 connect YOUR_SSID`
6. **Enter Password:** Type your Wi-Fi key when prompted.
7. **Exit:** Type `exit`.

**Verification:** Run `ping -c 3 google.com`. If you get replies, you’re online!

#### B. Synchronize the System Clock

*Update system clock:*

```bash
timedatectl set-ntp true
```

*Check status with:*

```bash
timedatectl status
```


<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-275 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="4" id="bkmrk-bash-1" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"></div>- *Explanation:* `timedatectl` synchronizes your clock. If your clock is wrong, SSL certificates (used by HTTPS websites/mirrors) will appear invalid, and package downloads will fail.

---

### 6. Setting up the SSH Bridge (Optional)

Installation is easier when you can copy-paste commands from your main computer.

1. **Set a temporary root password (t**his password is only for the **live session):**```bash
    passwd
    ```
    
    **Note:** This is only for the **live session**. It disappears when you reboot and won't affect your final system.
2. **Start the SSH service:**```bash
    systemctl start sshd
    ```
3. **Find your IP address:**```bash
    ip addr show wlan0
    ```
    
    *Look for the number after `inet` (e.g., `192.168.1.15`).*
4. **Connect from your OTHER computer:** Open your terminal on your daily-driver machine and type:
    
    ```bash
    ssh root@192.168.1.15
    ```

---

### 7. Verify UEFI Mode

We ensure the system is booted in UEFI mode, as `systemd-boot` requires it.

```bash
ls /sys/firmware/efi/efivars
```

*Explanation:* If this directory exists and is populated, you are in UEFI mode. If not, stop and check your BIOS settings.

---

## Phase 2: Partitioning &amp; Encryption

This is the most technical part of the build. We are creating a secure, high-performance foundation.

### 1. Wipe metadata

**Warning:** This will erase all data on `/dev/nvme0n1`(or your drive device).

```bash
sgdisk --zap-all /dev/nvme0n1
```

---

### 2. Partitioning

We will use `cfdisk` to create a simple, robust partition table.

Run `cfdisk /dev/nvme0n1`. Create the following:

1. **Label Type:** Select `gpt`.
2. Create a **New** partitions:

<table data-path-to-node="14" id="bkmrk-partition-size-type-"><thead><tr><td>**Partition**</td><td>**Size**</td><td>**Type**</td><td>**Explanation**</td></tr></thead><tbody><tr><td><span data-path-to-node="14,1,0,0">**`/dev/nvme0n1p1`**</span></td><td><span data-path-to-node="14,1,1,0">**1 GiB**</span></td><td><span data-path-to-node="14,1,2,0">EFI System</span></td><td><span data-path-to-node="14,1,3,0">Large size to hold multiple Kernel images (linux, linux-lts) and rescue images.</span></td></tr><tr><td><span data-path-to-node="14,2,0,0">**`/dev/nvme0n1p2`**</span></td><td><span data-path-to-node="14,2,1,0">**Remainder**</span></td><td><span data-path-to-node="14,2,2,0">Linux root (x86-64)</span></td><td><span data-path-to-node="14,2,3,0">This will hold our Encrypted Container.</span></td></tr></tbody></table>

 3. Select "**Write**" -&gt; "**yes**" -&gt; "**Quit**".

**Pro-Tip: Why "Linux root (x86-64)"?** We use this specific type instead of the generic "Linux filesystem" to follow modern standards. It allows the system to automatically identify your drive as the "root" partition, which adds a layer of redundancy if your configuration files ever have issues.

---

### 3. Format EFI Partition

```bash
mkfs.fat -F 32 /dev/nvme0n1p1
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-276 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="9" id="bkmrk--11" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-276"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-276"></div></div></div>- ***Explanation**:* The UEFI motherboard firmware can only read FAT32 filesystems. This is where the bootloader lives.

---

### 4. Encrypt Root Partition (LUKS2)

We encrypt the raw partition before putting a filesystem on it.

```bash
cryptsetup luksFormat /dev/nvme0n1p2
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-277 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="11" id="bkmrk--14" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-277"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-277"></div></div></div>- ***Explanation**:* Initializes the partition with LUKS2 encryption. You will set a passphrase here.
- ***Note**:* By default, this uses **Argon2id**, the modern memory-hard key derivation function (highly secure). You **must** type `YES` in all capital letters.

---

### 5. Unlock Partition

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-278 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="13" id="bkmrk--15" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-278">  
</div></div>```bash
cryptsetup open /dev/nvme0n1p2 cryptroot
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-278 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="13" id="bkmrk--17" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-278"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-278"></div></div></div>- *Explanation:* Decrypts the drive and maps it to `/dev/mapper/cryptroot`. The system treats `cryptroot` as a standard unencrypted drive from now on.
- **Verification Step:** Run `lsblk`. You should now see `cryptroot` nested under your second partition:

[![image.png](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/scaled-1680-/XLSimage.png)](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/XLSimage.png)

---

## Phase 3: Btrfs Subvolume Layout

This phase is where you define how your data is organized and how your system will handle "time travel" (backups/snapshots). Btrfs allows "Subvolumes"—dynamic partitions that share the same free space. We are going to use the **"Lean Snapshot"** strategy. This ensures that when you take a system backup, it stays tiny by excluding folders that don't need to be saved (like caches and logs).

---

### 1. Format the Unlocked Vault

Now that your LUKS container is open at `/dev/mapper/cryptroot`, we format it with Btrfs.

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-279 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="15" id="bkmrk-bash-5" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-279">  
</div></div><div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-279 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="15" id="bkmrk--21" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-279"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-279"></div></div></div>```bash
mkfs.btrfs -L Arch /dev/mapper/cryptroot
```

---

### 2. Create the Subvolume Layout

To create subvolumes, we must first mount the main partition temporarily.

```bash
mount /dev/mapper/cryptroot /mnt
```

Now, create the subvolumes. Notice we are separating the "heavy" folders like `@cache` and `@log`.

```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@pkg_cache
```

***Explanation***:

- **`@` (Root):** Contains the OS. We snapshot this to roll back the system.
- **`@home`:** Your personal data. Excluded from root snapshots so rolling back the OS doesn't delete your documents.
- **`@snapshots`:** Where Snapper saves the snapshots. Kept separate to prevent infinite recursion (snapshotting the snapshot folder).
- **`@log`:** System logs. Excluded so that if you roll back a broken system, you can still read the logs to see *why* it broke.
- **`@pkg_cache`:** Pacman cache. Excluded so you don't lose downloaded packages after a rollback (saves re-downloading).
- **Note on `@var`:** We do **NOT** separate `/var` entirely. The pacman database (`/var/lib/pacman`) MUST stay with the Root (`@`). If you separate it and roll back Root, your installed binaries won't match the database, breaking package management.

Unmount the temporary root:

```bash
umount /mnt
```

---

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-280 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="17" id="bkmrk-bash-6" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"></div><div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-280 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="17" id="bkmrk--24" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-280"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-280"></div></div></div>### 3. Mount Boot Partition and Subvolumes with Optimizations

We will now mount everything in its final location using performance flags.

**First, mount the Root (`@`):**

```bash
mount -o noatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@ /dev/mapper/cryptroot /mnt
```

Next, create the mount points (the "folders"):

```bash
mkdir -p /mnt/{home,.snapshots,var/log,var/cache/pacman/pkg,boot}
```

Now, mount the rest of the subvolumes:

```bash
mount -o noatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@home /dev/mapper/cryptroot /mnt/home
mount -o noatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@snapshots /dev/mapper/cryptroot /mnt/.snapshots
mount -o noatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@log /dev/mapper/cryptroot /mnt/var/log
mount -o noatime,compress=zstd,ssd,discard=async,space_cache=v2,subvol=@pkg_cache /dev/mapper/cryptroot /mnt/var/cache/pacman/pkg
```

***Explanation:***

<table data-path-to-node="11" id="bkmrk-flag-why-we-use-it-c" style="width: 100%;"><thead><tr><td style="width: 16.0836%;">**Flag**</td><td style="width: 83.9813%;">**Why we use it**</td></tr></thead><tbody><tr><td style="width: 16.0836%;"><span data-path-to-node="11,1,0,0">**`compress=zstd`**</span></td><td style="width: 83.9813%;"><span data-path-to-node="11,1,1,0">**The Space Saver.** Compresses data on the fly. It makes the disk "faster" because the CPU compresses the data quicker than the disk can write the uncompressed version.</span></td></tr><tr><td style="width: 16.0836%;"><span data-path-to-node="11,2,0,0">**`noatime`**</span></td><td style="width: 83.9813%;"><span data-path-to-node="11,2,1,0">**The Life Extender.** Normally, Linux writes to the disk every time you simply *read* a file (to record the "access time"). This turns that off, reducing unnecessary wear on your NVMe.</span></td></tr><tr><td style="width: 16.0836%;"><span data-path-to-node="11,3,0,0">**`ssd`**</span></td><td style="width: 83.9813%;"><span data-path-to-node="11,3,1,0">**Layout Optimization.** Tells Btrfs to use allocation strategies specifically designed for solid-state storage rather than spinning platters.</span></td></tr><tr><td style="width: 16.0836%;"><span data-path-to-node="11,4,0,0">**`discard=async`**</span></td><td style="width: 83.9813%;"><span data-path-to-node="11,4,1,0">**The Background Cleaner.** This is the modern way to handle "TRIM." It tells the SSD which blocks are no longer used in the background, keeping your write speeds high over time.</span></td></tr><tr><td style="width: 16.0836%;"><span data-path-to-node="11,5,0,0">**`space_cache=v2`**</span></td><td style="width: 83.9813%;"><span data-path-to-node="11,5,1,0">**Fast Booting.** This flag helps the system track free space much faster. While it is technically the default in modern Linux kernels (5.15+), explicitly including it is a "safety first" practice that ensures your filesystem always uses the most efficient, modern tracking method.</span></td></tr></tbody></table>

Finaly, mount the Boot Partition:

```bash
mount /dev/nvme0n1p1 /mnt/boot
```

---

#### Why this layout is "Lean"

By mounting `@log` and `@cache` as separate subvolumes, they are technically **outside** your root (`@`) subvolume. When you use a tool like `Snapper` to snapshot `@`, it will "skip" these folders.

- **Space Saved:** Your system snapshots won't grow every time you download a large update or generate massive logs.
- **Safety:** If you roll back your system to "yesterday," you won't lose the logs from "today," which helps you figure out what went wrong.

#### Pro-Tip: Verification Step

Before moving to the next phase, verify that your "plumbing" is correct. Run:

```bash
lsblk
```

***Success look like this*:** You should see your drive (`nvme0n1`) with two partitions. Under the second partition, you should see `cryptroot` with **all five** mount points listed:

[![image.png](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/scaled-1680-/Lj8image.png)](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/Lj8image.png)

If a mount point is missing, go back and re-run the `mount` command for that subvolume!

---

## Phase 4: Base System Installation


Now that the foundation is ready, this is the part where we pull the official Arch Linux packages from the mirrors and install them into your `/mnt` directory. This phase is fast because we do the heavy lifting **once** we are inside the new system.

### 1. Update the Mirrors (Optional but Recommended)

To ensure fast download speeds, pick the best servers:

```bash
reflector --country Canada --age 12 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

**Tip:** If you aren't in Canada, replace `Canada` with your own country (e.g., `United States`, `Germany`, `France`). You can also list multiple countries separated by commas, like `--country 'United States,Canada'`.

Here is the breakdown of the command:

- **`reflector`**: The base command that runs the mirror-sorting utility.
- **`--country Canada`**: Limits the search to servers physically located in Canada. This is crucial for speed because it reduces "latency" (the time it takes for a server to respond to a request).
- **`--age 12`**: Only includes mirrors that have successfully synchronized with the main Arch Linux server within the last 12 hours. This ensures you aren't using "stale" mirrors that are missing the latest software or security patches.
- **`--protocol https`**: Only includes mirrors that support encrypted HTTPS connections. This provides a layer of security by preventing "Man-in-the-Middle" attacks from tampering with your data during transit.
- **`--sort rate`**: Ranks the filtered list based on download speed (transfer rate). The fastest server will be placed at the top of the file.
- **`--save /etc/pacman.d/mirrorlist`**: Tells the program to take the resulting list and overwrite your existing mirrorlist file. This is the file `pacman` looks at whenever you install or update a package.

---

### 2. The `pacstrap` Command

We will now install the base system, the Linux kernel, and essential firmware. We are adding several critical tools here so you aren't left without internet or a text editor when you first reboot.

```bash
pacstrap -K /mnt base linux linux-firmware linux-firmware-marvell btrfs-progs neovim networkmanager sudo
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-282 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="21" id="bkmrk-bash-8" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="code-block-decoration header-formatted gds-title-s ng-tns-c576689755-282 ng-star-inserted">**Package Breakdown:**</div><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-282">  
</div></div><div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-282 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="21" id="bkmrk--31" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-282"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-282"></div></div></div>- **`base`**: Minimal filesystem and core tools (ls, cp, bash).
- **`base-devel`**: GCC, Make, Sudo. Required for compiling AUR packages later. (optional)
- **`linux`** : Standard Linux Kernel.
- **`linux-firmware`**: Drivers for Wi-Fi, GPUs, etc.
- **`linux-firmware-marvell`**: **Critical:** Specific firmware required for Marvell wireless chips (common in Surface devices and certain laptops).
- **`btrfs-progs`**: Userspace tools to manage Btrfs.
- **`networkmanager`**: Easy network management tool.
- **`neovim`**: Text editor (use `nano` if you prefer).
- **`sudo`**: Allow a regular user to run commands with administrative (root) privileges. If you choose to install `base-devel`, you do **not** need to list `sudo` separately, as it is already included inside this package.

---

### 3. Generate the File System Table (`fstab`)

This tells the system where all those subvolumes are.

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

***Action**:* fstab Verification

Before moving forward, it is critical to verify that your filesystem table was generated correctly. Run:

```bash
cat /mnt/etc/fstab
```

**What to look for:** On a Btrfs system like this one, your `fstab` will look a bit different than a traditional setup.

- **The UUID:** You should see the same long **UUID** string repeated for almost every entry. This is because all your subvolumes (`@`, `@home`, `@log`, etc.) live on the same physical partition.
- **Subvolume Flags:** Look at the options column (usually the 4th column). Each line must have a unique `subvol=@...` entry matching the subvolumes you created in Phase 3.
- **Mount Points:** Ensure the paths match your intended layout (e.g., the line with `subvol=@log` should point to `/var/log`).

[![image.png](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/scaled-1680-/g2Jimage.png)](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/g2Jimage.png)

**Note :** Your UUID (the long string of letters and numbers) will be different from the one in the image. Don't copy it! Just verify that your UUID is consistent across all your Btrfs entries.

---

### 4. Enter the System (Chroot)

Now that the "plumbing" is verified, we are going to "step inside" your new installation. From this point on, any command you run happens on your NVMe drive, not the USB stick.

```bash
arch-chroot /mnt
```

***Success*:** Your terminal prompt will change to `[root@archiso /]#`. You are now officially configuring your new OS!

---

## Phase 5: System Configuration

Now we install the core drivers and utilities while configuring the time, language, and performance settings that make the system truly yours."

### 1. The Essentials (The Bulk Command)

This command installs the "foundation" of your system, including the bootloader, hardware drivers, and core utilities required for a modern, functional desktop.

**Note:** Use `intel-ucode` for Intel CPUs or `amd-ucode` for AMD.

```bash
pacman -S grub efibootmgr zram-generator intel-ucode reflector linux-headers bluez bluez-utils xdg-utils xdg-user-dirs network-manager-applet pipewire pipewire-pulse pipewire-alsa pipewire-jack wireplumber
```

##### Package Breakdown

- **`grub` &amp; `efibootmgr`**: The core tools required to create and manage your boot menu so the motherboard can find Arch.
- **`zram-generator`**: The modern way to handle swap, compressing data in your RAM to keep the system fast under heavy load.
- **`intel-ucode`**: Critical stability and security patches for your CPU (essential for modern hardware).
- **`reflector`**: A script that fetches and sorts the fastest Arch mirrors to keep your future updates speedy.
- **`linux-headers`**: Required if you ever plan to install "out-of-tree" drivers like VirtualBox or certain Wi-Fi/NVIDIA drivers.
- **`bluez` &amp; `bluez-utils`**: The "brain" and tools for Bluetooth. You'll need these to connect your headphones or mouse.
- **`xdg-user-dirs`**: Automatically creates the standard folders in your home directory (Documents, Downloads, Music, etc.) once you log in.
- **`xdg-utils`**: A set of command-line tools that help different apps talk to each other (e.g., "open this link in the browser").
- **`network-manager-applet`**: Provides the Wi-Fi icon in your future desktop environment so you don't have to use the terminal for Wi-Fi anymore. (requires a GUI to actually show up)
- **`pipewire`**: The main "engine" that handles audio and video streams.
- **`pipewire-pulse`**: A "translator" that lets apps designed for PulseAudio (like Spotify or Discord) work with PipeWire.
- **`pipewire-alsa` &amp; `pipewire-jack`**: Ensures that even the oldest and the most professional audio apps can output sound.
- **`wireplumber`**: The "brain" that manages the logic—it decides which speakers to use when you plug in headphones.

#### 1.1. Useful packages you can add

##### Power Management (For Laptops)

If this is a laptop, your battery will drain much faster than it should without a power daemon.

- **Package:** `power-profiles-daemon`
- **Why:** This is the modern standard used by GNOME and KDE. It lets you switch between "Power Saver," "Balanced," and "Performance" modes easily.
- **Enable:** `systemctl enable power-profiles-daemon`

##### Printer Support (The "Just in Case")

Even if you don't own a printer, you'll eventually need to "Print to PDF" or connect to a wireless printer at a library or office.

- **Packages:** `cups` and `avahi`
- **Why:** `cups` is the printing engine; `avahi` is what allows your computer to "see" printers (and other devices) on your Wi-Fi network without typing in IP addresses.
- **Enable:** `systemctl enable cups` and `systemctl enable avahi-daemon`

##### Basic Graphics Drivers (The "Visual Brain")

We have the CPU patches, but your screen needs to know how to talk to your GPU (Intel, AMD, or NVIDIA) for smooth animations.

- **Packages:**
    
    
    - **<span class="citation-18">Intel:</span>** `<span class="citation-18">vulkan-intel</span>`<span class="citation-18"> and </span>`<span class="citation-18 citation-end-18">intel-media-driver</span>`
        
        
        - *Note: `vulkan-intel` handles the graphics, while `intel-media-driver` allows your hardware to decode video (like YouTube/Netflix) so your CPU doesn't have to work as hard.*
    - **AMD:** `vulkan-radeon`
    - **NVIDIA:** `nvidia` (or `nvidia-open` for newer cards)
- **Why:** This ensures your future Desktop Environment (like GNOME or KDE) uses hardware acceleration. Without this, your CPU has to do all the visual work, making the system feel "laggy."

---

### 2. Localization, Time and Keyboard

This section tells Arch your timezone, your language, and how your keyboard is laid out.

Set your timezone (Replace with your actual City/Region):

```bash
ln -sf /usr/share/zoneinfo/America/Edmonton /etc/localtime
```

Update current system time to the hardware clock on your motherboard, ensuring the correct time persists even after you shut down or reboot:

```bash
hwclock --systohc
```

Set Language (The "Echo" Shortcut):

```bash
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
```

***Note***: This adds the US English locale to the generation list without opening an editor.

Generate the locales by running:

```bash
locale-gen
```

Create Locale Config:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Set Console Keyboard Layout:

```bash
echo "KEYMAP=us" > /etc/vconsole.conf
```

---

### 3. Network Identity

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-285 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="27" id="bkmrk-bash-11" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"></div><div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-286 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="29" id="bkmrk-give-your-computer-a" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-286"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-286"><span class="ng-tns-c576689755-285">Give your computer a name:</span></div><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-286">  
</div></div></div>```bash
echo "arch-linux" > /etc/hostname
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-286 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="29" id="bkmrk--44" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-286"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-286"></div></div></div>Edit your hosts file:

```bash
nvim /etc/hosts
```

Add this text to the file:

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   arch-linux.localdomain   arch-linux
```

**Note:** Ensure "arch-linux" matches the name you chose in the `/etc/hostname` file above.

---

### 4. High-Performance zRAM

Since we just installed the generator, let's configure it.

Open the file:

```bash
nvim /etc/systemd/zram-generator.conf
```

Paste this in:

```ini
[zram0]
zram-size = min(ram / 2, 4096)
compression-algorithm = zstd
swap-priority = 100
fs-type = swap
```

---

## Phase 6: Mkinitcpio &amp; Bootloader

This is the critical step for booting an encrypted drive.

### 1. Configure Mkinitcpio

We need to tell Arch to include the tools for encryption and the Btrfs filesystem in the initial boot process.

**Open the configuration:**

```
nvim /etc/mkinitcpio.conf
```

**Edit the MODULES line:** Find the MODULES=() line. and add btrfs to it:

```
MODULES=(btrfs)
```

****Edit the HOOKS line:** Find the `HOOKS=(...)` line. It must look exactly like this:**

```
HOOKS=(base systemd autodetect microcode modconf kms keyboard keymap sd-vconsole sd-encrypt block filesystems fsck)
```

**Note:** We use `sd-encrypt` because it integrates perfectly with the modern `systemd` boot process we are building.

**Generate the images:**

```bash
mkinitcpio -P
```

---

### 2. The "Gatekeeper" (GRUB)

Now we configure the bootloader to find your specific encrypted partition.

**Find your Partition UUID:** Run this command to get the unique ID of your LUKS partition (`/dev/nvme0n1p2`):

```bash
blkid -s UUID -o value /dev/nvme0n1p2
```

**Tip:** Copy this long string carefully.

**Edit GRUB's configuration:**

```bash
nvim /etc/default/grub
```

**Set the Boot Parameters:** Find the line `GRUB_CMDLINE_LINUX_DEFAULT` and change it to (paste your UUID where indicated):

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet rd.luks.name=YOUR_UUID_HERE=cryptroot root=/dev/mapper/cryptroot rootflags=subvol=@"
```

**Install GRUB to the motherboard:**

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-289 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="35" id="bkmrk-generate-the-final-c" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="code-block-decoration header-formatted gds-title-s ng-tns-c576689755-289 ng-star-inserted">**<span class="ng-tns-c576689755-289">Generate the final config file:</span>**  
<div _ngcontent-ng-c576689755="" class="buttons ng-tns-c576689755-289 ng-star-inserted"><button aria-label="Copy code" class="mdc-icon-button mat-mdc-icon-button mat-mdc-button-base mat-mdc-tooltip-trigger copy-button ng-tns-c576689755-289 mat-unthemed ng-star-inserted"></button>  
</div></div></div>```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-289 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="35" id="bkmrk-bash-15" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="code-block-decoration header-formatted gds-title-s ng-tns-c576689755-289 ng-star-inserted"><div _ngcontent-ng-c576689755="" class="buttons ng-tns-c576689755-289 ng-star-inserted"></div></div>---

</div><div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-289 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="35" id="bkmrk--54" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-289"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-289"></div></div></div>### 3. User Setup &amp; Security

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-290 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="37" id="bkmrk--58" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-290"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-290"></div></div></div>Before we leave, we need to create the people who will live in this system.

**Set the Root Password** This is the "Master" password for the entire system:

```bash
passwd
```

**Create Your User** Replace `paulo` with your preferred username:

```bash
useradd -m -G wheel paulo
```

**Set the User Password:**

```bash
passwd paulo
```

**Grant Administrative Privileges (Sudo)** Since you installed `sudo` in Phase 4, you need to tell it that your user is allowed to use it:

```bash
EDITOR=nvim visudo
```

**Action:** Find the line `%wheel ALL=(ALL:ALL) ALL` and remove the `#` from the front. Save and exit (`:wq`).

### 4. Enable Services

Ensure your system is ready to connect to the world on the first boot.

```bash
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable fstrim.timer
```

**The "Enable" Breakdown:**

<table data-path-to-node="4" id="bkmrk-command-simple-expla" style="width: 100%;"><thead><tr><td style="width: 17.153%;">**Command**</td><td style="width: 28.7101%;">**Simple Explanation**</td><td style="width: 54.0827%;">**Why it’s Essential**</td></tr></thead><tbody><tr><td style="width: 17.153%;"><span data-path-to-node="4,1,0,0">**`systemctl enable NetworkManager`**</span></td><td style="width: 28.7101%;"><span data-path-to-node="4,1,1,0">Tells the system to start the internet manager as soon as the computer boots.</span></td><td style="width: 54.0827%;"><span data-path-to-node="4,1,2,0">Without this, you will have **no Wi-Fi or Ethernet** when you log in. You would be stuck in a terminal with no way to download updates.</span></td></tr><tr><td style="width: 17.153%;"><span data-path-to-node="4,2,0,0">**`systemctl enable bluetooth`**</span></td><td style="width: 28.7101%;"><span data-path-to-node="4,2,1,0">Activates the Bluetooth "brain" (the bluez stack) in the background.</span></td><td style="width: 54.0827%;"><span data-path-to-node="4,2,2,0">If you don't enable this, your Bluetooth hardware stays powered off. Your wireless mouse, keyboard, or headphones **won't connect**.</span></td></tr><tr><td style="width: 17.153%;"><span data-path-to-node="4,3,0,0">**`systemctl enable fstrim.timer`**</span></td><td style="width: 28.7101%;"><span data-path-to-node="4,3,1,0">Schedules a weekly "cleanup" (Trim) of your NVMe/SSD drive.</span></td><td style="width: 54.0827%;"><span data-path-to-node="4,3,2,0">It keeps your drive fast and healthy by telling the hardware which data blocks are no longer used. It’s like **automatic maintenance** for your storage.</span></td></tr></tbody></table>

---

## Phase 7: The Escape Sequence

Run these commands in order. This ensures all your data is physically written to the NVMe and the "vault" is locked safely before the restart.

#### 1. Step out of the new system

This takes you out of the `arch-chroot` and back to the USB environment.

```bash
exit
```

#### 2. Unmount everything

This "unplugs" all your Btrfs subvolumes and your boot partition safely.

```bash
umount -R /mnt
```

#### 3. Close the Encrypted Vault

This locks your LUKS partition so it can be re-opened by your bootloader.

```bash
cryptsetup close cryptroot
```

#### 4. The Final Reboot

Pull out your USB drive once the screen goes dark!

```bash
reboot
```

---

## Phase 7: Post-Install (The First Boot)

Welcome to your new Arch Linux system. Now that you’ve rebooted and logged in, let’s get the system connected and optimized.

#### 1. Connect to Wi-Fi

Since we are now in the installed system, we use `nmcli` (Network Manager) to handle our connections permanently.

**Turn on the Wi-Fi radio:**

```bash
nmcli radio wifi on
```

**List available networks:**

```bash
nmcli device wifi list
```

**Connect to your network:**

```bash
nmcli device wifi connect "YOUR_SSID" password "YOUR_PASSWORD"
```

---

#### 2. Enable Remote Management SSH (Optional) 

If you prefer to finish the setup from another computer (for easier copy-pasting), install and enable the SSH daemon:

**Install the Server**

You likely installed `base`, but you might not have the SSH daemon yet. Run this on your **Arch machine**:

```bash
sudo pacman -S openssh
```

**Start and Enable the Service**

This makes the "door" available now and every time you reboot:

```bash
sudo systemctl enable --now sshd
```

**Find Your IP Address**

You need to know where to point your other computer.

```bash
ip addr show
```

Look for the number next to `inet` under your Wi-Fi or Ethernet (usually looks like `192.168.1.X`).

---

#### 3. Optimize Your Mirrors (Reflector)

The mirrors used during installation might not be the fastest for your specific location. We’ll use `reflector` to find the best servers near you.

```bash
sudo reflector --country 'Canada,United States' --latest 10 --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

****What this does:**** It finds the 10 most recently synchronized HTTPS mirrors in your region, tests their actual download speed, and saves the winners to your system.

##### Automate Your Mirror Updates

Reflector comes with a built-in service that can update your mirrors automatically. Let’s configure it to run once a week so your system always stays fast.

Step 1: Edit the Reflector configuration:

```bash
sudo nvim /etc/xdg/reflector/reflector.conf
```

**Step 2: Paste this "High-Speed" config** Replace everything in that file with this. This tells Reflector to find the 10 fastest HTTPS mirrors in Canada or the US updated in the last 12 hours:

```ini
--save /etc/pacman.d/mirrorlist
--protocol https
--country Canada,United States
--latest 10
--sort rate
--age 12
```

**Step 3: Enable the Timer** Now, "flip the switch" to make it run automatically every week:

```bash
sudo systemctl enable --now reflector.timer
```

---

#### 4. Synchronize and Update

Now that you have fast mirrors, let’s do a final sync to make sure everything is perfectly up to date.

```bash
sudo pacman -Syu
```

---

#### 5. Verify Your Services

Before you move on run these commands to ensure your essential performance and maintenance services are running correctly:

**The "Master Schedule" Check:** Run this to see all your automated tasks (Reflector, Trim, etc.) in one list:

```bash
systemctl list-timers --all
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-1094 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="0" data-ved="0CAAQhtANahgKEwjRy5yYjPWRAxUAAAAAHQAAAAAQ1gc" decode-data-ved="1" id="bkmrk--49" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_b601b7334d3bbe10","c_cf7a34519b79b725",null,"rc_eae4914bb399a991",null,null,"en",null,1,null,null,1,0]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-1094"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-1094"></div></div></div>****What to look for:** Ensure `fstrim.timer` and `reflector.timer` are listed in the `ACTIVATES` column.**

**The Hardware &amp; Performance Check:** Since these aren't "timers," we check them individually:

- **zRAM:** `zramctl` (Confirms your RAM-based swap is active).
- **Audio:** `wpctl status` (Confirms PipeWire has "claimed" your speakers).
- **Network:** `nmcli device` (Confirms you are online and managed).

---

## Phase 8: System Hardening (Security)

Encryption (LUKS) protects your data when your computer is stolen or powered off. **System Hardening** protects you while you are actually using the computer and connected to the internet.

### 1. The Firewall (UFW)

We will use **UFW** (Uncomplicated Firewall). It is the standard for "set and forget" security. It blocks all unauthorized "knocks" on your system's digital doors.

```bash
sudo pacman -S ufw
sudo systemctl enable --now ufw
```

**Apply the "Public Wi-Fi" rules**

****CRITICAL:** If you are currently connected to this machine via **SSH**, you **must** allow the SSH port *before* enabling the firewall, or you will be immediately disconnected and locked out.**

**First, explicitly allow SSH:**

```bash
sudo ufw allow ssh
```

**Then set the default "Stealth" rules:**

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
```

**What this does:** It makes your laptop "invisible" on public networks. You can still browse and download anything, but no outside device can initiate a connection to your machine.

---

### 2. CPU Security (Microcode)

Modern CPUs (Intel and AMD) have hardware-level vulnerabilities. "Microcode" is a small piece of code that the Linux kernel loads at boot to patch these holes and keep your processor stable.

**Verify it is working:**

```bash
journalctl -b | grep microcode
```

- **Success Look:** You should see a line saying `microcode: updated early to...`.
- **If it's missing:** Ensure you have `intel-ucode` (or `amd-ucode`) installed and that it is listed in your GRUB configuration.

---

### 3. SSH Security (Optional)

If you enabled SSH in Phase 7 to finish this guide, you should now lock the door so hackers can't "brute force" their way in.

1. **Edit the config:** `sudo nvim /etc/ssh/sshd_config`
2. **Disable Root Login:** Find `PermitRootLogin` and set it to `no`.
3. **Restart the service:** `sudo systemctl restart sshd`

---

### Security Summary

This configuration represents the **essential minimum** required to secure a modern Arch Linux system. While you can always go deeper into hardening (MAC, AppArmor, or Sandboxing), the combination of **LUKS Encryption**, a **UFW Firewall**, and **Microcode patches** provides a rock-solid foundation that protects your data from both physical theft and network-based threats.

---

## Phase 9: The "Undo" Button (Snapper)

Since we are using **Btrfs**, we can take "snapshots" of the system. If an update or a configuration change breaks your OS, you can roll back the entire system to a previous state in seconds.

### 1. Install the Tools

We will install the core engine (`snapper`) and a highly recommended management tool.

```bash
sudo pacman -S snapper snap-pac grub-btrfs inotify-tools btrfs-assistant
```

- **`btrfs-assistant`**: An excellent GUI tool (once you install a desktop) that makes browsing and restoring snapshots point-and-click easy.
- <span data-path-to-node="10,0,2,0,0,0">`snap-pac`: Automates snapshots during pacman updates.</span><span data-path-to-node="10,0,2,0,0,1"><sup class="superscript" data-turn-source-index="4"></sup></span>
    
    <div _ngcontent-ng-c358061296="" class="source-inline-chip-container ng-star-inserted"></div><div _ngcontent-ng-c358061296="" class="source-inline-chip-container ng-star-inserted"></div>
- <span data-path-to-node="10,0,2,1,0,0">`grub-btrfs`: Adds "Arch Linux Snapshots" to your boot menu.</span><span data-path-to-node="10,0,2,1,0,1"></span>

---

### 2. Configure the "Time Machine"

Snapper needs to be told to watch your root directory, and we need to link it to the `@snapshots` subvolume we created in Phase 2.

**Delete Snapper's default folder:**

```bash
sudo umount /.snapshots
sudo rm -r /.snapshots
```

**Create the initial config:**

```bash
sudo snapper -c root create-config /
```

**The "Fix": Linking to our @snapshots subvolume:** Snapper creates its own folder, but we want it to use our dedicated subvolume for better organization.

**Delete the junk:**

```bash
sudo btrfs subvolume delete /.snapshots
```

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-292 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="39" id="bkmrk-bash-16" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"></div><div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-294 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="43" id="bkmrk-ini%2C-toml" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="code-block-decoration header-formatted gds-title-s ng-tns-c576689755-294 ng-star-inserted"></div></div><div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-294 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="43" id="bkmrk--72" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"><div _ngcontent-ng-c576689755="" class="formatted-code-block-internal-container ng-tns-c576689755-294"><div _ngcontent-ng-c576689755="" class="animated-opacity ng-tns-c576689755-294"></div></div></div>**Re-create and link our subvolume:**

```bash
sudo mkdir /.snapshots
sudo mount -a
```

(Since `/.snapshots` is already in your `/etc/fstab`, `mount -a` automatically remounts our `@snapshots` subvolume into the correct spot).

---

### 3. Set Permissions &amp; Automation

Ensure your user can access the snapshots and tell the system to handle the schedule automatically.

**Set Permissions:**

```
sudo chmod 750 /.snapshots
```

**Enable Automatic Snapshots:**

```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```

- **`snapper-timeline.timer`**: Takes a snapshot every hour.
- **`snapper-cleanup.timer`**: Automatically deletes old snapshots based on the rules in `/etc/snapper/configs/root` so your drive doesn't fill up.

---

### 4. Adding "Bootable Snapshots

To make your "Undo" button work from the GRUB menu.

#### **1. The "Pro" Automation (Daemon):** 

This is the best part. Instead of manually updating GRUB every time a snapshot is taken, we enable a "watchdog" that adds new snapshots to the boot menu automatically the moment they are created.

```bash
sudo systemctl enable --now grub-btrfsd
```

---

#### **2. The Daemon Fix (The Path):**

1. Run: `sudo EDITOR=NVIM systemctl edit --full grub-btrfsd`
2. Change the `ExecStart` line to: `ExecStart=/usr/bin/grub-btrfsd --syslog /.snapshots`
3. Save and restart: `sudo systemctl daemon-reload && sudo systemctl restart grub-btrfsd`

---

#### <span data-path-to-node="23,0">**3. The Overlay Hook (The Writable RAM):** </span>

<span data-path-to-node="23,0">Snapshots are read-only. </span><span data-path-to-node="23,2"><span class="citation-427">You must add a "writable layer" in RAM or the system will crash during boot</span></span><span data-path-to-node="23,3"><span class="citation-427 citation-end-427"><sup class="superscript" data-turn-source-index="9"></sup><sup class="superscript" data-turn-source-index="9"></sup></span></span><span data-path-to-node="23,4">.</span>

**Open mkinitcpio.con:**

```bash
sudo nvim /etc/mkinitcpio.conf
```

**<span data-path-to-node="24,2,0,1"><span class="citation-426">Add </span>`<span class="citation-426">grub-btrfs-overlayfs</span>`<span class="citation-426"> to the </span><span class="citation-426">very end</span><span class="citation-426"> of </span>`<span class="citation-426">HOOKS=(...)</span>`:</span>**

```
HOOKS=(base systemd autodetect microcode modconf kms keyboard keymap sd-vconsole block sd-encrypt filesystems fsck grub-btrfs-overlayfs)
```

**Build it:**

```bash
sudo mkinitcpio -P
```

---

#### **4. Update your GRUB config:** 

You need to tell GRUB to look for the snapshots and add them to the menu.

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

---

#### **5. The manual **Surgical Rollback Test**:**

We need to make a change we can actually "undo."

**Create the "Dirty" state:** Install something small so we can see it disappear:

```bash
sudo pacman -S fastfetch
```

(Verify it works by typing `fastfetch`).

**Check the snapshot number:**

```bash
sudo snapper list
```

[![image.png](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/scaled-1680-/wszimage.png)](https://bookstack.spiderhulk.net/uploads/images/gallery/2026-01/wszimage.png)


**Note**: you will see something like this. The number we want is 2, the pre fastfetch installation.

<span data-path-to-node="0,1"><span class="citation-1337">In the Btrfs filesystem, </span>**<span class="citation-1337">Subvolume ID 5</span>**<span class="citation-1337"> is the "Top-Level" or "Root" subvolume that is automatically created when you format the partition</span></span><span data-path-to-node="0,2"><span class="citation-1337 citation-end-1337"><sup class="superscript" data-turn-source-index="1"></sup><sup class="superscript" data-turn-source-index="1"></sup><sup class="superscript" data-turn-source-index="1"></sup><sup class="superscript" data-turn-source-index="1"></sup><sup class="superscript" data-turn-source-index="1"></sup><sup class="superscript" data-turn-source-index="1"></sup><sup class="superscript" data-turn-source-index="1"></sup></span></span><span data-path-to-node="0,3">. You can think of it as the **Master Container** that holds everything else you've created, including your `@`, `@home`, and `@snapshots` subvolumes.</span>

**How to Verify it on Your System**

While ID 5 is a hardcoded standard for the Btrfs root, you can see it for yourself by running this command:

```bash
sudo btrfs subvolume list /
```

<span data-path-to-node="4,0">In the output, look at the **ID** column for the subvolume that has no parent (often listed with a path like `<top level>` or just `/`). </span><span data-path-to-node="4,2"><span class="citation-1336">Every other subvolume you see (like </span>`<span class="citation-1336">@</span>`<span class="citation-1336"> or </span>`<span class="citation-1336">@home</span>`<span class="citation-1336">) will show a </span>`<span class="citation-1336">top level</span>`<span class="citation-1336"> or </span>`<span class="citation-1336">parent</span>`<span class="citation-1336"> ID of 5, proving they are "children" of that master subvolume</span></span><span data-path-to-node="4,3"><span class="citation-1336 citation-end-1336"><sup class="superscript" data-turn-source-index="2"></sup><sup class="superscript" data-turn-source-index="2"></sup><sup class="superscript" data-turn-source-index="2"></sup><sup class="superscript" data-turn-source-index="2"></sup></span></span><span data-path-to-node="4,4">.</span>

**Why we mount ID 5 for the Rollback**

<span data-path-to-node="7,1"><span class="citation-1335">Standard Linux mounting only shows you the "inside" of a specific subvolume (like your current </span>`<span class="citation-1335">@</span>`<span class="citation-1335"> system)</span></span><span data-path-to-node="7,2"><span class="citation-1335 citation-end-1335"><sup class="superscript" data-turn-source-index="3"></sup></span></span><span data-path-to-node="7,3">. </span><span data-path-to-node="7,5"><span class="citation-1334">To perform a "Surgical Rollback," you have to step </span>**<span class="citation-1334">outside</span>**<span class="citation-1334"> the current system so you can move the folders around</span></span><span data-path-to-node="7,6"><span class="citation-1334 citation-end-1334"><sup class="superscript" data-turn-source-index="4"></sup><sup class="superscript" data-turn-source-index="4"></sup><sup class="superscript" data-turn-source-index="4"></sup><sup class="superscript" data-turn-source-index="4"></sup></span></span><span data-path-to-node="7,7">.</span>

- <span data-path-to-node="8,0,1,0">**<span class="citation-1333">Accessing the Siblings</span>**<span class="citation-1333">: Mounting ID 5 allows you to see </span>`<span class="citation-1333">@</span>`<span class="citation-1333"> and </span>`<span class="citation-1333">@snapshots</span>`<span class="citation-1333"> side-by-side</span></span><span data-path-to-node="8,0,1,1"><span class="citation-1333 citation-end-1333"><sup class="superscript" data-turn-source-index="5"></sup><sup class="superscript" data-turn-source-index="5"></sup><sup class="superscript" data-turn-source-index="5"></sup><sup class="superscript" data-turn-source-index="5"></sup></span></span><span data-path-to-node="8,0,1,2">.</span>
- **Renaming the Living**: You cannot delete or move a subvolume if you are currently "inside" it. By mounting ID 5 to `/mnt`, you are looking at the drive from the "outside," which gives you the power to run the `mv` (rename) command on the `@` subvolume.

**Mount the top-level subvolume (ID 5):**

```bash
sudo mount -o subvolid=5 /dev/mapper/cryptroot /mnt
```

**Quarantine the broken/dirty system:**

```bash
sudo mv /mnt/@ /mnt/@_broken
```

**Create a read-write snapshot of your clean Snapshot 2 and name it @**:

```bash
sudo btrfs subvolume snapshot /mnt/@snapshots/2/snapshot /mnt/@
```

**Unmount:**

```bash
sudo umount /mnt
```

**Reboot:**

```bash
sudo reboot
```

- <span data-path-to-node="12,1,1,0">**<span class="citation-1322">Certainty</span>**<span class="citation-1322">: By renaming the subvolume to </span>`<span class="citation-1322">@</span>`<span class="citation-1322">, you are forcing the bootloader to load the clean files</span></span><span data-path-to-node="12,1,1,1"><span class="citation-1322 citation-end-1322"><sup class="superscript" data-turn-source-index="4"></sup><sup class="superscript" data-turn-source-index="4"></sup><sup class="superscript" data-turn-source-index="4"></sup><sup class="superscript" data-turn-source-index="4"></sup></span></span><span data-path-to-node="12,1,1,2">.</span>
- <span data-path-to-node="12,2,1,0">**<span class="citation-1321">Safety</span>**<span class="citation-1321">: Your "dirty" system is preserved in </span>`<span class="citation-1321">/@_broken</span>`<span class="citation-1321"> in case you forgot to save a file from it</span></span><span data-path-to-node="12,2,1,1"><span class="citation-1321 citation-end-1321"><sup class="superscript" data-turn-source-index="5"></sup><sup class="superscript" data-turn-source-index="5"></sup><sup class="superscript" data-turn-source-index="5"></sup><sup class="superscript" data-turn-source-index="5"></sup></span></span><span data-path-to-node="12,2,1,2">.</span>

**<span data-path-to-node="12,2,1,2">Final Step: Clean up the "Crime Scene"</span>**

<span data-path-to-node="12,2,1,2">Once you reboot and confirm `fastfetch` is gone, you are back in your clean state. To get your disk space back, you can delete the "broken" subvolume:</span>

**<span data-path-to-node="12,2,1,2">Get the broken subvolumes:</span>**

```bash
sudo btrfs subvolume list /
```

<span data-path-to-node="12,2,1,2">**Mount the top-level subvolume (ID 5):**</span>

<div _ngcontent-ng-c576689755="" class="code-block ng-tns-c576689755-301 ng-animate-disabled ng-trigger ng-trigger-codeBlockRevealAnimation" data-hveid="57" id="bkmrk-bash-20" jslog="223238;track:impression,attention;BardVeMetadataKey:[["r_78bece2bc76fe813","c_92ebb604e4ff0ad7",null,"c_92ebb604e4ff0ad7_arch_btrfs_master_guide",null,null,null,null,1,null,null,1,1]]"></div>```bash
sudo mount -o subvolid=5 /dev/mapper/cryptroot /mnt
```

**Delete the broken volume:**

If you have nested broken subvolumes you have to delete the children first (e.g. /mnt/@\_broken/var/lib/portables)

```bash
sudo btrfs subvolume delete /mnt/@_broken
```

**Unmount:**

```bash
sudo umount /mnt
```

**The "Kernel Gap" Warning**

<span data-path-to-node="15,1"><span class="citation-1450">Rolling back the root filesystem can sometimes leave your kernel out of sync if </span>`<span class="citation-1450">/boot</span>`<span class="citation-1450"> wasn't part of the snapshot.</span></span>

<span data-path-to-node="15,1"><span class="citation-1450">Once you reboot and log back in, immediately run this to ensure your kernel matches your new system:</span></span>

```bash
sudo pacman -S linux linux-headers
```

<span data-path-to-node="15,1"><span class="citation-1450"><span data-path-to-node="18,1"><span class="citation-1449">This forces the kernel image in your FAT32 </span>`<span class="citation-1449">/boot</span>`<span class="citation-1449"> partition to match the modules in your restored </span>`<span class="citation-1449">/usr/lib/modules</span>`</span><span data-path-to-node="18,2"><span class="citation-1449 citation-end-1449"><sup class="superscript" data-turn-source-index="9"></sup></span></span><span data-path-to-node="18,3">.</span></span></span>

<span data-path-to-node="15,1"><span class="citation-1450"><span data-path-to-node="18,3"><span data-path-to-node="8,1,0">**<span class="citation-1769">Troubleshooting: The "Ghost" Pacman Lock</span>** </span><span data-path-to-node="8,1,1"><span class="citation-1769 citation-end-1769"><sup class="superscript" data-turn-source-index="7"></sup></span></span><span data-path-to-node="8,1,2"> Every time you rollback, `pacman` might report that the database is locked. </span><span data-path-to-node="8,1,4"><span class="citation-1768">This is because the snapshot captured the system while a transaction was open. </span></span><span data-path-to-node="8,1,5"><span class="citation-1768 citation-end-1768"><sup class="superscript" data-turn-source-index="8"></sup></span></span><span data-path-to-node="8,1,6"> **The Fix:** </span></span></span></span>

```bash
sudo rm /var/lib/pacman/db.lck
```

Snapshots are "frozen in time," and if you took a snapshot while a package was installing, the "lock" is frozen into that snapshot too.

Note: Remember, once you clear that lock, you **must** finish the command: `sudo pacman -S linux linux-headers`

---

#### **6. Rollback from GRUB:**

If you break your system:

1. **Reboot**: Select a snapshot from the "Arch Linux snapshots" menu in GRUB.
2. **Select the Kernel**: Do not click the Date/Description (Header); select the indented line that mentions `vmlinuz-linux`.
3. <span data-path-to-node="28,2,0,0">**The Permanent Fix**: Once logged in, you are in a temporary session. </span><span data-path-to-node="28,2,0,2"><span class="citation-1690">To make it permanent repeat the steps from the part 5 above.</span></span><span data-path-to-node="28,2,0,3"><span class="citation-1690 citation-end-1690"><sup class="superscript" data-turn-source-index="18"></sup><sup class="superscript" data-turn-source-index="18"></sup><sup class="superscript" data-turn-source-index="18"></sup><sup class="superscript" data-turn-source-index="18"></sup></span></span>

---

### Final Note: From One User to Another

This guide is a collection of my own research, trial, and error. **I am not a professional Linux administrator or a Btrfs developer.** I am an enthusiast who wanted a "Time Machine" that actually works on Arch Linux, and after many broken installs, this is the configuration that finally held up.

**A few words of advice:**

- <span data-path-to-node="6,0,1,0">**<span class="citation-2347">The Arch Wiki is the Truth:</span>**<span class="citation-2347"> While this guide works for my specific setup (LUKS/Btrfs), always consult the </span>[<span class="citation-2347">Arch Wiki</span>](https://wiki.archlinux.org/)<span class="citation-2347"> for the most up-to-date information</span></span><span data-path-to-node="6,0,1,1"><span class="citation-2347 citation-end-2347"><sup class="superscript" data-turn-source-index="1"></sup></span></span><span data-path-to-node="6,0,1,2">.</span>
- **Test your Backups:** A rollback system you haven't tested is just a hope. Periodically try the "Manual Hero" method to stay sharp.
- **Your System, Your Responsibility:** Breaking things is the best way to learn how they work. Don't be afraid to experiment, but always make a manual snapshot before you do.
