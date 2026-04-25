# QEMU Validation Notes

## Purpose

This note explains what is and is not expected when trying to run the generated Discovery Kit image in QEMU.

## Current Image Context

The generated image targets Microchip PolarFire SoC Discovery Kit (RISC-V).

Relevant files:
- [buildroot/output/images/sdcard.img](../../buildroot/output/images/sdcard.img)
- [buildroot/output/images/dts/microchip/mpfs-disco-kit.dtb](../../buildroot/output/images/dts/microchip/mpfs-disco-kit.dtb)

## Practical Constraints

1. Board-specific images often do not boot on generic `virt` machine without adaptation.
2. QEMU support depends on machine model availability.
3. Local environment must have `qemu-system-riscv64` installed.

## Recommended Validation Strategy

1. Primary validation on real Discovery Kit hardware.
2. Secondary QEMU validation with a dedicated `qemu virt` Buildroot config.
3. Keep hardware smoke tests as source of truth for boot and service startup.

## If You Still Want QEMU Trial

Generic experiment command (may fail for board-specific image):

```bash
qemu-system-riscv64 \
  -M virt \
  -m 2G \
  -smp 4 \
  -nographic \
  -kernel /home/ritesh/Stage1/buildroot/output/images/Image \
  -append "console=ttyS0 root=/dev/vda rw" \
  -drive file=/home/ritesh/Stage1/buildroot/output/images/rootfs.ext2,format=raw,if=virtio
```

Use this only as a development aid, not final platform qualification.
