# Quark A11+KSU experimental integration

Base: A11+ f9c074c2cab. KernelSU: backslashxx/KernelSU commit
21a0a46155d964927afab6fde40b9a09aa995357 (driver version 32716).
Sources and GPL-2.0-only license are preserved in drivers/kernelsu.
QUARK_SOURCE.json records the original SHA-256 of every copied file.
The upstream setup.sh is omitted; builds never fetch a mutable upstream branch.

## Configuration

- Built into the ARM32 kernel with CONFIG_KSU=y; no Linux module is needed.
- ARM syscall-table interception and ultra-legacy automatic LSM hooks enabled.
- Full UID checks and debug diagnostics enabled. KSU does not grant universal
  shell su. KernelSU's optional ADB-root feature, SU logging and hosts redirect
  are disabled for this first integration.
- A11+ SELinux enforcement remains unchanged. No global permissive switch.
- ARM64-only preinit is automatically excluded by the upstream driver.
- No OverlayFS backport or assumption of compatibility with root modules.

## Build and test

Use the A11+KSU local_manifests branch in a separate LineageOS 18.1 workspace.
Device and vendor remain their current A11+ versions; only the kernel differs.
Do not modify a workspace while another build is running.

```sh
source build/envsetup.sh
lunch lineage_quark-userdebug
repo sync kernel/motorola/apq8084
mka bootimage -j8
cp out/target/product/quark/boot.img out/target/product/quark/boot-A11-KSU-$(git -C kernel/motorola/apq8084 rev-parse --short HEAD).img
```

Use the existing ROM's matching device/ramdisk/signing configuration when
building only boot.img. Keep the last working boot.img available for rollback.
The GitHub workflow builds zImage-dtb and uploads compiler/config/log evidence;
zImage-dtb is NOT a flashable boot.img and must not be flashed as one.

First boot without root modules. Confirm `adb shell getenforce` still returns
Enforcing and inspect dmesg for KernelSU initialization, symbol/LSM discovery
errors and stack warnings. Retest microphone, camera, P2, Wi-Fi, Bluetooth,
phone calls and suspend before installing modules.

The fork's release v3.3.1+36 (driver interface version 32716) contains ARMv7
ksud and a Manager certificate already accepted by manager/apk_sign.c:
867 bytes, SHA-256 4359c171f32543394cbc23ef908c4bb94cad7c8087002ba164c8230948c21549.
The downloaded APK's v2 signature and signed content digest were verified.
APK SHA-256: 798254823ffcc60c136a8f6ae903bfee6ff4ccd164fffc8ba1a3b749df0f607b.
Download: https://github.com/backslashxx/KernelSU/releases/tag/v3.3.1%2B36

These are the fork's existing accepted certificates; no signature checks
were disabled. The Makefile additionally defines the official KernelSU key.
For a production custom Manager, use your own protected signing key and
review all accepted keys. Random APKs and ARM64-only builds are unsuitable.

Check for an existing root provider before enabling KernelSU in daily use:
inspect `command -v su`, active su/Magisk processes and modules. Do not run
two module frameworks concurrently. Existing sudaemon policy alone does not
prove a daemon is active and was not removed by this change.

Validate root approval, denial and revocation per app and persistence after
reboot. Module support is a separate test: CONFIG_OVERLAYFS_FS remains off.
For rollback, flash the saved matching boot.img; this does not delete Manager
or /data/adb state. Keep those states separated when comparing root providers.
