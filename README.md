# AxionOS for Samsung r8q

Custom Android ROM based on AxionOS (Android 16) for Samsung r8q (Snapdragon 865).

---

## 🔧 Features

- **GMS Build** – Google Mobile Services included
- **KernelSU integrated** – Root access for advanced users
- **UDFPS support** – Under-display fingerprint sensor
- **Blur enabled** – Visual polish and depth effects
- **60/75/90/120Hz refresh rate support** – Smooth scrolling
- **Performance tuning** – schedutil governor for balanced performance

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

## 🚀 First Time Setup

### 1. Initialize the Manifest

```bash
repo init -u https://github.com/AxionOS/manifest.git -b 2.6
```

### 2. Sync the Source

```bash
repo sync -j8
```

Replace `-j8` with your CPU thread count for faster sync. Monitor your RAM usage if building on lower-end systems.

---

## 🔐 Generate Signing Keys (IMPORTANT)

**⚠️ Do this ONLY ONCE before your first build.**

Run the key generation script:

```bash
gk -s
```

Keys will be stored in:

```
vendor/lineage-priv/keys/
```

**CRITICAL:** Backup these keys immediately—losing them will break OTA updates and force users to clean flash.

### Backup Your Keys

Create a secure backup:

```bash
tar -czvf axion-r8q-keys-backup.tar.gz vendor/lineage-priv/keys/
```

Store this backup in a **secure, separate location** (external drive, cloud storage with encryption, etc.).

---

## 📱 Choose Device Build Variant

### 1. Source Build Environment

```bash
source build/envsetup.sh
```

### 2. Select ROM Variant

```bash
axion r8q gms user
```

**Available variants:**

| Variant | Description |
|---------|-------------|
| `gms` | Full Google apps (Gmail, Play Store, Maps, etc.) |
| `gms core` | Minimal Google services |
| `vanilla` | No Google services (pure AOSP) |

Use `gms` for most users. Switch to `userdebug` instead of `user` for testing:

```bash
axion r8q gms userdebug
```

---

## 🏗️ Build ROM (First Time)

### Build Command

```bash
ax -br -j8
```

- `-b`: Build ROM
- `-r`: Build recovery (optional)
- `-j8`: Use 8 parallel jobs (adjust to your CPU threads)

**Build time:** 45–120 minutes depending on hardware and cache status.

### Monitor Build

Watch the output. The build will:
1. Compile framework
2. Compile system apps
3. Package final ZIP

**Note:** First build takes longer due to ccache population. Subsequent builds are faster.

---

## ✅ After First Successful Build

The final ROM zip is located at:

```
out/target/product/r8q/axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip
```

### Flash to Device

1. Boot into recovery (TWRP/OrangeFox)
2. Wipe:
   - **Data** (if coming from stock)
   - **Cache & Dalvik**
3. Flash ROM zip
4. Reboot

**First boot:** May take 5–10 minutes. Be patient.

---

## 🔄 Future Builds with the SAME Signing Keys

### For Subsequent Builds (Using Existing Keys)

If your keys are still in `vendor/lineage-priv/keys/`, rebuilding is straightforward:

#### 1. Update Source

```bash
axionSync
```

Or manually:

```bash
repo sync -j8
```

#### 2. Rebuild ROM

```bash
source build/envsetup.sh
axion r8q gms user
ax -br -j8
```

The build system **automatically uses your existing keys**.

---

### If Building on a New Machine (Restore Keys First)

If you've moved to a different machine or directory:

#### 1. Extract Keys

```bash
mkdir -p vendor/lineage-priv
tar -xzvf axion-r8q-keys-backup.tar.gz -C vendor/lineage-priv/
```

#### 2. Verify Key Files

```bash
ls -la vendor/lineage-priv/keys/
```

You should see:

```
releasekey.pk8
releasekey.x509.pem
platform.pk8
platform.x509.pem
shared.pk8
shared.x509.pem
media.pk8
media.x509.pem
testkey.pk8
testkey.x509.pem
```

#### 3. Proceed with Build

```bash
source build/envsetup.sh
axion r8q gms user
ax -br -j8
```

---

## ⚠️ Critical: Signing Key Management

### Why Key Consistency Matters

- **Same keys** → OTA updates work, users keep their data
- **Different keys** → Device rejects update, users must clean flash

### Best Practices

1. **Keep keys private** – Never commit to public repos or share
2. **Backup in multiple locations** – External drive + cloud (encrypted)
3. **Use consistent machine** – Same dev machine = faster rebuilds
4. **Document key generation date** – For your records

### What NOT to Do

- ❌ Store keys in public GitHub repos
- ❌ Email keys to others
- ❌ Rebuild with `gk -s` a second time (creates new keys)
- ❌ Share `vendor/lineage-priv/` folder

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

Example:

```
axion-2.6-QUASIS-UNOFFICIAL-GMS-r8q.zip
```

### For OTA Updates

After signing (see advanced section below), OTA zips are generated at:

```
signed-ota.zip
```

Users can flash this for incremental updates without wiping data.

---

## 📲 Flashing Instructions

### Clean Flash (First Time or Major Update)

1. **Boot recovery:**
   ```
   adb reboot recovery
   ```
   Or: Power off → Hold Vol Up + Power

2. **Wipe partitions:**
   - Wipe Data
   - Wipe Cache
   - Wipe Dalvik/ART Cache

3. **Flash ROM:**
   - Select zip → Confirm
   - Wait for installation

4. **Reboot:**
   - Reboot System
   - First boot takes 5–10 minutes

### OTA Update (Preserves Data)

1. **Place OTA zip** in internal storage or recovery root
2. **Boot recovery**
3. **Apply update from ZIP**
4. **Reboot**

---

## 🔥 Advanced: Signing Builds for OTA Distribution

### Prerequisites

- Build tools generated: `mka target-files-package otatools`
- Keys backed up and accessible in `vendor/lineage-priv/keys/`

### Sign Build

```bash
./build/tools/releasetools/sign_target_files_apks \
  -o -d vendor/lineage-priv/keys \
  out/target/product/r8q/obj/PACKAGING/target_files_intermediates/*-target_files*.zip \
  signed-target_files.zip
```

### Generate OTA ZIP

```bash
./build/tools/releasetools/ota_from_target_files \
  -b vendor/lineage-priv/keys/releasekey \
  signed-target_files.zip \
  signed-ota.zip
```

Final OTA is ready at: `signed-ota.zip`

---

## ❌ Do NOT Include in GitHub

```
out/                           (build output)
.repo/                         (repo metadata)
vendor/lineage-priv/keys/      (signing keys - CONFIDENTIAL)
*.img, *.bin, *.so, *.apk      (proprietary blobs)
ccache/                        (build cache)
```

---

## 📄 Environment Variables (Optional)

Speed up builds with ccache:

```bash
export USE_CCACHE=1
export CCACHE_EXEC=/usr/bin/ccache
ccache -M 50G
```

Add to `~/.bashrc` for permanent setup.

---

## 🐛 Troubleshooting

### Build Fails: "Command not found"

Ensure repo is initialized:

```bash
source build/envsetup.sh
```

### Keys Missing After Rebuild

Restore from backup:

```bash
tar -xzvf axion-r8q-keys-backup.tar.gz -C vendor/lineage-priv/
```

### Out of Storage

Check available space:

```bash
df -h
```

Clean old builds:

```bash
rm -rf out/
```

Then rebuild.

### Slow Build

Enable ccache (see Environment Variables section above).

---

## 🔗 Useful Links

- [AxionOS Official](https://github.com/AxionOS)
- [LineageOS r8q Device Tree](https://github.com/LineageOS/android_device_samsung_r8q)
- [Android Build System](https://source.android.com/docs/setup/build/building)
- [TWRP Recovery](https://twrp.me)

---

## 👨‍💻 Maintainer

**Suryansh Deo**  
GitHub: [@suryanshdeo](https://github.com/suryanshdeo)

---

## 📝 License

This ROM is built upon AxionOS and LineageOS, which are based on AOSP. Respect all applicable licenses and contributor guidelines.

---

## 🔥 Final Notes

- **Always test builds before flashing** to real devices
- **Keep signing keys private and backed up**
- **Use consistent build environment** for reproducible builds
- **Monitor first boot** – first-time startup can take 5–10 minutes
- **Report issues** on GitHub with build logs if problems occur

---

**Happy building! 🚀**
