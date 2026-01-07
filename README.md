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

		 **Note:** You may see errors for other files; as long as it says `archlinux-		202x...iso: OK`, you are safe.

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
### 4. Booting the ISO

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


- *Explanation:* `timedatectl` synchronizes your clock. If your clock is wrong, SSL certificates (used by HTTPS websites/mirrors) will appear invalid, and package downloads will fail.

---
### 6. Setting up the SSH Bridge (Optional)

Installation is easier when you can copy-paste commands from your main computer.

1. **Set a temporary root password (t**his password is only for the **live session):**
    
	```bash
	passwd
	```
    
	**Note:** This is only for the **live session**. It disappears 	when you reboot and won't affect your final system.

2. **Start the SSH service:**
    
	```bash
	systemctl start sshd
	```
3. **Find your IP address:**
    
	```bash
	ip addr show wlan0
	```
    
	*Look for the number after `inet` (e.g., `192.168.1.15`).*

4. **Connect from your OTHER computer:** Open your terminal on your daily-driver machine and type:
    
    ```bash
    ssh root@192.168.1.15
    ```

---
