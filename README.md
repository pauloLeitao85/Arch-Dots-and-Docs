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
