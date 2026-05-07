# AxionOS for Samsung r8q

Custom Android ROM based on AxionOS (Android 16) for Samsung r8q (Snapdragon 865).

**Version:** 2.6 QUASIS-UNOFFICIAL-GMS  
**Device:** Samsung Galaxy S20 Ultra (r8q) / SM8250  
**Build Date:** April 2026  
**Status:** Production Ready

---

## 🔧 Device Specifications

| Property | Value |
|----------|-------|
| **Device Codename** | r8q (SM8250) |
| **SoC** | Qualcomm Snapdragon 865 |
| **Camera (Rear)** | 12MP + 12MP + 8MP |
| **Camera (Front)** | 32MP |
| **UDFPS** | Yes (Under-Display) |
| **Refresh Rates** | 60Hz, 120Hz |
| **Maintainer** | LoveMyAss |
| **ROM Version** | AxionOS |

---

## ✨ Key Features

- **GMS Build** – Google Mobile Services included (Gmail, Play Store, Maps, etc.)
- **KernelSU integrated** – Root access for advanced users
- **UDFPS support** – Under-display fingerprint sensor fully functional
- **Blur enabled** – Visual polish and depth effects throughout UI
- **60/120Hz refresh rate support** – Smooth scrolling and animations
- **Performance tuning** – schedutil CPU governor for balanced performance and battery life
- **High Brightness Mode (HBM)** – Enhanced visibility in sunlight (framework support)
- **Charge Warning** – Charge protection enabled

---

## 📦 Build Requirements

- **Ubuntu 22.04 LTS** or newer
- **32GB+ RAM** (recommended; 16GB minimum, but slower)
- **500GB+ storage** (SSD strongly recommended for speed)
- **Python 3**, Git, repo tool

**References:**  
- [LineageOS Build Guide for r8q](https://wiki.lineageos.org/devices/r8q/build/variant1/)
- [AOSP Build Setup](https://source.android.com/docs/setup/build/initializing)

---

## 🏗️ Build Configuration Overview

### Axion Custom Properties (device/samsung/r8q/lineage_r8q.mk)

```makefile
# Camera Configuration
AXION_CAMERA_REAR_INFO := 12,12,8     # Rear cameras: 12MP + 12MP + 8MP
AXION_CAMERA_FRONT_INFO := 32         # Front camera: 32MP

# Device Information
AXION_MAINTAINER := LoveMyAss
AXION_PROCESSOR := Snapdragon_865

# Feature Flags
TARGET_ENABLE_BLUR := true            # Enable blur effects
BYPASS_CHARGE_SUPPORTED := true       # Charge protection support
TARGET_HAS_UDFPS := true              # Under-display fingerprint sensor
TARGET_INCLUDE_AXFX := true           # AxFX effects engine
TARGET_SUPPORTED_REFRESH_RATES := 60,120  # Supported refresh rates
```

### Performance Configuration (device/samsung/r8q/device.mk)

```makefile
# CPU Governor Configuration
PERF_GOV_SUPPORTED := false            # Use default governor
PERF_DEFAULT_GOV := schedutil          # schedutil for power efficiency
PERF_ANIM_OVERRIDE := false            # Standard animation timing

# Display Features
HBM_SUPPORTED := false                 # HBM framework support (hardware agnostic)
HBM_NODE := /sys/class/backlight/panel0-backlight/hbm_mode
```

---

## 🚀 First Time Setup

### Step 1: Initialize the Repository Manifest

```bash
repo init -u https://github.com/AxionAOSP/android.git -b lineage-23.2 --git-lfs
```

**Flags:**
- `-u`: Manifest repository URL
- `-b lineage-23.2`: Branch to use (LineageOS 23.2 / Android 16)
- `--git-lfs`: Enable Git LFS for large binary files

### Step 2: Sync the Source (First Sync)

```bash
repo sync -c -j16
```

**Flags:**
- `-c`: Current branch only (faster sync)
- `-j16`: 16 parallel jobs (adjust to your CPU thread count)

**Estimated time:** 30–60 minutes on a good connection  
**Storage used:** ~400–500GB

Replace `-j16` with your CPU thread count for faster sync. Monitor your RAM usage if building on lower-end systems.

### Step 3: Clone Device Trees and Vendor Blobs

If you prefer manual control, remove the hardware directory and clone all device trees and vendor blobs individually:

```bash
rm -rf hardware/samsung/
git clone https://github.com/suryanshdeo/android_device_samsung_r8q -b bka-q2 device/samsung/r8q/
git clone https://github.com/suryanshdeo/android_device_samsung_sm8250-common -b bka-q2 device/samsung/sm8250-common/
git clone https://github.com/suryanshdeo/proprietary_vendor_samsung_r8q -b bka-q2 vendor/samsung/r8q/
git clone https://github.com/suryanshdeo/proprietary_vendor_samsung_sm8250-common -b bka-q2 vendor/samsung/sm8250-common/
git clone https://github.com/suryanshdeo/android_kernel_samsung_sm8250 -b bka-q2 kernel/samsung/sm8250/
git clone https://github.com/suryanshdeo/android_hardware_samsung -b bka-q2 hardware/samsung/
git clone https://github.com/suryanshdeo/android_hardware_samsung_nfc -b bka-q2 hardware/samsung/nfc/
git clone https://github.com/suryanshdeo/android_hardware_samsung_slsi_nfc -b bka-q2 hardware/samsung_slsi/nfc/
```

---

## 🔐 Generate Signing Keys (CRITICAL - FIRST TIME ONLY)

**⚠️ Do this ONLY ONCE before your first build. Regenerating keys will break OTA updates.**

### Step 1: Generate Keys

Source the build environment and generate signing keys:

```bash
source build/envsetup.sh
gk -s
```

Keys will be stored in:

```
vendor/lineage-priv/keys/
```

Key files generated:
- `releasekey.pk8` / `releasekey.x509.pem` – ROM signature
- `platform.pk8` / `platform.x509.pem` – System apps
- `shared.pk8` / `shared.x509.pem` – Shared libraries
- `media.pk8` / `media.x509.pem` – Media framework
- `testkey.pk8` / `testkey.x509.pem` – Test applications

### Step 2: Backup Signing Keys (ESSENTIAL)

Create a secure backup immediately:

```bash
tar -czvf keys-backup.tar.gz vendor/lineage-priv/keys/
```

Store this backup in **multiple secure locations**:
- External USB drive (encrypted)
- Cloud storage with encryption (Google Drive, Nextcloud, etc.)
- Separate machine/server

**CRITICAL:** Losing these keys means:
- All future OTA updates will fail on users' devices
- Users will be forced to clean flash (data loss)
- You cannot release minor/patch updates

### Step 3: Restore Keys on New Machine

If building on a different machine or after a clean install:

```bash
# Extract keys to the correct location
tar -xzvf keys-backup.tar.gz -C vendor/lineage-priv/

# Verify keys are present
ls -la vendor/lineage-priv/keys/
```

---

## 📱 Prepare for Build

### Step 1: Source Build Environment

```bash
cd ~/axion
source build/envsetup.sh
export SKIP_ABI_CHECKS=true
```

**Environment variables:**
- `SKIP_ABI_CHECKS=true` – Skip ABI checks (useful for development builds)
- `USE_CCACHE=1` – Enable compiler cache (optional, speeds up rebuilds)
- `CCACHE_EXEC=/usr/bin/ccache` – Use ccache if installed

### Step 2: Select Device and Build Variant

```bash
axion r8q gms user
```

**Parameters:**
- `r8q` – Target device
- `gms` – Build variant with Google Mobile Services
- `user` – User build (production; use `userdebug` for testing)

**Available variants:**

| Variant | Description | Use Case |
|---------|-------------|----------|
| `gms` | Full Google apps (Play Store, Gmail, Maps, etc.) | Most users |
| `gms core` | Minimal Google services | Privacy-conscious users |
| `vanilla` | No Google services (pure AOSP) | De-googled builds |

For testing builds, use `userdebug` instead of `user`:

```bash
axion r8q gms userdebug
```

---

## 🔨 Build the ROM

### Full Build Command

```bash
export SKIP_ABI_CHECKS=true
ax -br -j8
```

**Flags:**
- `-b` – Build ROM
- `-r` – Build recovery
- `-j8` – Use 8 parallel jobs (adjust to your CPU thread count)

**Expected output location:**

```
out/target/product/r8q/axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip
```

### Build Timeline

| Step | Duration | Notes |
|------|----------|-------|
| Initial setup | 2–3 min | First time only |
| Framework compilation | 10–15 min | Slower on first build (ccache populate) |
| System apps | 15–25 min | Varies with parallelism |
| Device-specific modules | 5–10 min | Kernel, HALs, vendor libs |
| Package ROM | 5–8 min | Zip creation and signing |
| **Total (first build)** | **45–90 min** | Depends on CPU cores and SSD speed |
| **Total (rebuild)** | **30–60 min** | With ccache enabled |

### Monitor Build Progress

```bash
# Watch build logs in real-time
tail -f out/verbose.log

# Check current parallel jobs
ps aux | grep -E "cc1plus|clang|ld"

# Monitor disk usage
watch -n 1 'df -h'
```

### Build Failure Troubleshooting

**If the build fails:**

1. Check for error messages in the output
2. Ensure all dependencies are installed
3. Verify signing keys are present: `ls -la vendor/lineage-priv/keys/`
4. Clean and retry:

```bash
rm -rf out/
source build/envsetup.sh
axion r8q gms user
ax -br -j8
```

---

## ✅ Post-Build Verification

### Verify ROM Package

After a successful build, verify the ROM package:

```bash
ls -lah out/target/product/r8q/axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip
```

Expected output:
```
-rw-r--r-- 1 user user 950M Apr 29 21:22 axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip
```

### Verify ROM Signature

The ROM is automatically signed with your keys during the build. No additional signing step is required.

---

## 📤 Output & Distribution

### ROM Location

```
out/target/product/r8q/axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip
```

### Naming Convention

```
axion-<VERSION>-<CODENAME>-<VARIANT>-<DEVICE>.zip
```

### Copy to Distribution Location

```bash
cp out/target/product/r8q/axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip ~/Downloads/
```

---

## 🔄 Future Builds (Using Existing Keys)

After your first successful build, subsequent builds are simpler:

### Update Source

```bash
repo sync -c -j16
```

Or use the convenience command:

```bash
repo sync
```

### Rebuild ROM

```bash
source build/envsetup.sh
export SKIP_ABI_CHECKS=true
axion r8q gms user
ax -br -j8
```

The build system automatically uses your existing signing keys from `vendor/lineage-priv/keys/`. **Do NOT run `gk -s` again.**

---

## 📲 Flashing Instructions

### Clean Flash (First Time or Major Update)

**Prerequisites:**
- Device unlocked bootloader (Samsung: `adb reboot bootloader` then `fastboot oem unlock`)
- TWRP or OrangeFox recovery installed
- 2GB free internal storage minimum

**Steps:**

1. **Boot into recovery:**
   ```bash
   adb reboot recovery
   ```
   Or manually: Power off → Hold Vol Up + Power

2. **Wipe partitions:**
   - Swipe to confirm wipes
   - **Wipe Data** (factory reset)
   - **Wipe Cache**
   - **Wipe Dalvik/ART Cache**

3. **Flash ROM:**
   - Select ZIP
   - Choose: `out/target/product/r8q/axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip`
   - Confirm
   - Wait for completion (2–5 minutes)

4. **Reboot system:**
   - **Reboot → System**
   - **First boot takes 5–10 minutes** – device is optimizing apps, do NOT interrupt

5. **Initial setup:**
   - Complete first-time setup
   - Sign in to Google account
   - Allow system updates if prompted

### OTA Update (Incremental, Data Preserved)

For subsequent builds, users can perform OTA updates:

1. Place OTA ZIP in recovery accessible location or internal storage root
2. Boot recovery
3. Select **Apply Update from ZIP**
4. Select the OTA package
5. Confirm and wait for installation
6. Reboot

---

## 🐛 Troubleshooting


### Build Fails: "Command not found"

Ensure build environment is sourced:

```bash
source build/envsetup.sh
```

Or the device function is not available:

```bash
axion r8q gms user    # Check command availability
```

### Build Fails: "Permission denied"

Ensure scripts are executable:

```bash
chmod +x build/envsetup.sh
find device/ kernel/ vendor/ -name "*.sh" -exec chmod +x {} \;
```

### Build Fails: "Out of memory"

Reduce parallel jobs:

```bash
ax -br -j4   # Use 4 jobs instead of 8-16
```

Or free up RAM:

```bash
sudo sync && sudo echo 3 > /proc/sys/vm/drop_caches
```

### Build Fails: "No space left on device"

Check available storage:

```bash
df -h
```

If low on space, clean build artifacts:

```bash
rm -rf out/
ccache -C    # Clear compiler cache if using ccache
```

You need **minimum 500GB free** for full builds.

### Keys Missing After Rebuild

Your keys should be preserved in `vendor/lineage-priv/keys/`. If missing, restore from backup:

```bash
mkdir -p vendor/lineage-priv/
tar -xzvf keys-backup.tar.gz -C vendor/lineage-priv/
ls -la vendor/lineage-priv/keys/
```

Then rebuild:

```bash
source build/envsetup.sh
axion r8q gms user
ax -br -j8
```

---

## 🔗 Useful Resources & Links


- **[AxionOS Official Repository](https://github.com/AxionOS)** – Main ROM sources
- **[LineageOS r8q Device Tree](https://github.com/LineageOS/android_device_samsung_r8q)** – Device-specific configs
- **[LineageOS Build Guide](https://wiki.lineageos.org/devices/r8q/build/)** – General build instructions
- **[Android Build System](https://source.android.com/docs/setup/build/building)** – AOSP build documentation
- **[Snapdragon 865 Specs](https://www.qualcomm.com/products/technology/snapdragon/phones)** – SoC information
- **[TWRP Recovery](https://twrp.me)** – Custom recovery flashing instructions
- **[OrangeFox Recovery](https://orangefox.io/)** – Alternative recovery with OTA support

## 📝 License


This ROM is built upon:
- **AxionOS** – Custom Android distribution
- **LineageOS** – Community-maintained Android distribution
- **AOSP** – Android Open Source Project

Respect all applicable licenses:
- AOSP: Apache License 2.0
- LineageOS: Individual component licenses
- AxionOS: Check official repository

All proprietary vendor files retain their original licenses from Samsung.


