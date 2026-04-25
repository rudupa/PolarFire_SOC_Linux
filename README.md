# Stage1 Workspace Overview

## Table of Contents

- [Folder Layout](#folder-layout)
- [`buildroot/` (Core Buildroot)](#buildroot-core-buildroot)
	- [Top-level files (selected)](#top-level-files-selected)
	- [Important subfolders](#important-subfolders)
	- [Frequently visited nested areas](#frequently-visited-nested-areas)
- [`buildroot-external-microchip/` (Microchip External Tree)](#buildroot-external-microchip-microchip-external-tree)
	- [Top-level files](#top-level-files)
	- [Important subfolders](#important-subfolders-1)
	- [Typical relationship with `buildroot/`](#typical-relationship-with-buildroot)
- [Quick Mental Model](#quick-mental-model)

This workspace contains two related repositories:

- `buildroot/`: Upstream Buildroot tree (core build system and packages)
- `buildroot-external-microchip/`: Microchip external tree layered on top of Buildroot

The external tree is used through the `BR2_EXTERNAL` mechanism to add board support, package recipes, patches, and configuration without modifying the core Buildroot tree directly.

## Folder Layout

```
Stage1/
├── buildroot/
└── buildroot-external-microchip/
```

---

## `buildroot/` (Core Buildroot)

This is the main Buildroot source tree that provides:
- Kconfig-based configuration system
- package infrastructure
- filesystem image generation
- toolchain and bootloader integration

### Top-level files (selected)

- `Makefile`: Main entry point for Buildroot commands (`make menuconfig`, `make`, etc.)
- `Config.in`, `Config.in.legacy`: Root Kconfig menu definitions
- `README`, `CHANGES`, `DEVELOPERS`, `COPYING`: Documentation, release notes, maintainer info, license

### Important subfolders

- `arch/`: Architecture-specific configuration and Kconfig fragments
- `board/`: Board-specific support snippets used by defconfigs
- `boot/`: Bootloader and firmware package definitions (U-Boot, ATF, etc.)
- `configs/`: Predefined Buildroot defconfig files
- `docs/`: Buildroot documentation
- `dl/`: Download cache for source tarballs and archives
- `fs/`: Filesystem image backend logic
- `linux/`: Linux kernel package support files
- `output/`: Build outputs (host tools, target rootfs, images, intermediate artifacts)
- `package/`: Package recipes and build logic for user-space software
- `support/`: Helper scripts and host-side tooling support
- `system/`: Skeleton/system integration files for target rootfs
- `toolchain/`: Internal and external toolchain integration logic
- `utils/`: Utility scripts used by Buildroot developers/users

### Frequently visited nested areas

- `board/<vendor>/<board>/`: Vendor and board-specific assets
- `boot/<component>/`: Boot component package directories
- `package/<pkg>/`: One directory per package recipe

---

## `buildroot-external-microchip/` (Microchip External Tree)

This repository extends Buildroot for Microchip platforms. It keeps Microchip-specific board support and packages outside the main Buildroot tree.

### Top-level files

- `README.md`: Usage/build instructions for Microchip targets
- `external.desc`: Metadata describing the external tree
- `external.mk`: Makefile include used by Buildroot when `BR2_EXTERNAL` is set
- `Config.in`, `Config.in.sam`: External Kconfig menus for Microchip options
- `COPYING`: License information

### Important subfolders

- `board/`: Microchip board/family assets (boot scripts, overlays, image helpers)
- `configs/`: Microchip defconfigs (AT91, PolarFire SoC, PIC64GX, etc.)
- `docs/`: External-tree-specific documentation
- `package/`: Microchip package recipes and custom integrations
- `patches/`: Patch sets applied to components during build
- `system/`: Root filesystem overlays and system customization snippets

### Typical relationship with `buildroot/`

From inside `buildroot/`, you point Buildroot to this external tree:

```sh
BR2_EXTERNAL=../buildroot-external-microchip make <defconfig>
make
```

This lets Buildroot include Microchip-specific configs and packages while preserving a clean separation from upstream Buildroot.

---

## Quick Mental Model

- Use `buildroot/` for core infrastructure and generic packages.
- Use `buildroot-external-microchip/` for Microchip-specific board support and customizations.
- `configs/` in each tree chooses what gets built; `output/` (in `buildroot/`) contains the final images.
