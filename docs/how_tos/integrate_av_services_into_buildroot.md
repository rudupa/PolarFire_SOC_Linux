# Integrate AV Services into Buildroot

## Purpose

This guide captures the AV services integration performed in the external Buildroot tree.

## What Was Added

### 1. External package definition

Created package folder:
- [buildroot-external-microchip/package/av-services](../../buildroot-external-microchip/package/av-services)

Key files:
- [Config.in](../../buildroot-external-microchip/package/av-services/Config.in)
- [av-services.mk](../../buildroot-external-microchip/package/av-services/av-services.mk)

### 2. Default runtime configs

Installed into target `/etc/av-core` from:
- [gateway.conf](../../buildroot-external-microchip/package/av-services/files/gateway.conf)
- [health.conf](../../buildroot-external-microchip/package/av-services/files/health.conf)
- [logger.conf](../../buildroot-external-microchip/package/av-services/files/logger.conf)
- [ota.conf](../../buildroot-external-microchip/package/av-services/files/ota.conf)
- [orchestrator.conf](../../buildroot-external-microchip/package/av-services/files/orchestrator.conf)

### 3. Kconfig wiring

Added package source entry in:
- [buildroot-external-microchip/Config.in](../../buildroot-external-microchip/Config.in)

### 4. Defconfig enablement

Enabled for Discovery Kit in:
- [buildroot-external-microchip/configs/mpfs_discovery_kit_defconfig](../../buildroot-external-microchip/configs/mpfs_discovery_kit_defconfig)

## Buildroot Package Behavior

The package uses local source from submodule:

```make
AV_SERVICES_SITE = $(TOPDIR)/../av_services
AV_SERVICES_SITE_METHOD = local
```

It builds CMake targets in [av_services](../../av_services), installs binaries and systemd units, then:
- installs config files to `/etc/av-core`
- creates `/etc/systemd/system/multi-user.target.wants/av-core.target`
- creates runtime dirs under `/var/lib/av-core` and `/var/log/av-core`

## Preconditions

1. `av_services` submodule must exist at root.
2. Init system must be systemd.
3. Toolchain must support C++ and threads.
4. Line endings in package files must be LF.

## Verify Config Selection

```bash
cd /home/ritesh/Stage1/buildroot
BR2_EXTERNAL=../buildroot-external-microchip make mpfs_discovery_kit_defconfig
grep -E '^BR2_PACKAGE_AV_SERVICES=|^BR2_INIT_SYSTEMD=|^BR2_PACKAGE_LIBCURL=' .config
```

Expected:
- `BR2_PACKAGE_AV_SERVICES=y`
- `BR2_INIT_SYSTEMD=y`
- `BR2_PACKAGE_LIBCURL=y`
