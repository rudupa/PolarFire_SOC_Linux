# Build, Flash, and Boot Verification

## Purpose

This guide explains how to build images with AV services, flash media, and verify service startup.

## Build Commands (WSL)

Use sanitized Linux PATH to avoid Windows PATH injection issues:

```bash
cd /home/ritesh/Stage1/buildroot
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
BR2_EXTERNAL=../buildroot-external-microchip make mpfs_discovery_kit_defconfig
BR2_EXTERNAL=../buildroot-external-microchip make -j$(nproc)
```

## Expected Image Artifacts

After successful build, check:

- [buildroot/output/images/sdcard.img](../../buildroot/output/images/sdcard.img)
- [buildroot/output/images/mpfs_discovery_kit.itb](../../buildroot/output/images/mpfs_discovery_kit.itb)
- [buildroot/output/images/rootfs.ext2](../../buildroot/output/images/rootfs.ext2)

## Flash Example

```bash
cd /home/ritesh/Stage1/buildroot/output/images
sudo dd if=sdcard.img of=/dev/sdX bs=1M status=progress conv=fsync
```

Replace `/dev/sdX` with the correct device.

## Boot-Time Verification on Target

After boot:

```bash
systemctl status av-core.target
systemctl status av-core-orchestrator.service
journalctl -u av-core.target -b
```

## File-Level Verification in Target Rootfs

Before flashing, you can check in output tree:

```bash
cd /home/ritesh/Stage1/buildroot
ls output/target/usr/local/bin/av-core-*
ls output/target/etc/systemd/system/av-core*
readlink output/target/etc/systemd/system/multi-user.target.wants/av-core.target
ls output/target/etc/av-core
```

Expected:
- five av-core binaries
- av-core service and target units
- symlink points to `/etc/systemd/system/av-core.target`
- config files exist under `/etc/av-core`

## Suggestions

1. Pin and tag known-good image builds.
2. Capture `journalctl -b` logs for first boot validation.
3. Add a smoke test script for `av-core-*` processes and target state.
