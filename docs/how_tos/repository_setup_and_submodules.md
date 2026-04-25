# Repository Setup and Submodules

## Purpose

This guide explains how the Stage1 workspace is structured and how to keep submodule-based repositories healthy.

## Current Repository Model

The root repository tracks three submodules:

- [buildroot](../../buildroot)
- [buildroot-external-microchip](../../buildroot-external-microchip)
- [av_services](../../av_services)

This model is preferred for:
- clean history per upstream project
- predictable commit pinning
- easier updates and rollbacks

## Fork Strategy

Use your own forks as `origin` in each submodule and keep original project as `upstream`.

Example:

```bash
# buildroot
cd /home/ritesh/Stage1/buildroot
git remote set-url origin https://github.com/rudupa/buildroot.git
git remote add upstream https://git.busybox.net/buildroot

# buildroot-external-microchip
cd /home/ritesh/Stage1/buildroot-external-microchip
git remote set-url origin https://github.com/rudupa/buildroot-external-microchip.git
git remote add upstream https://github.com/linux4microchip/buildroot-external-microchip.git
```

## Root Submodule Registration

If needed from scratch:

```bash
cd /home/ritesh/Stage1
git submodule add https://github.com/rudupa/buildroot.git buildroot
git submodule add https://github.com/rudupa/buildroot-external-microchip.git buildroot-external-microchip
git submodule add https://github.com/rudupa/av_services.git av_services
git add .gitmodules buildroot buildroot-external-microchip av_services
git commit -m "Add platform submodules"
```

## Daily Workflow

```bash
# root view
cd /home/ritesh/Stage1
git status
git submodule status

# check each submodule
cd /home/ritesh/Stage1/buildroot && git status
cd /home/ritesh/Stage1/buildroot-external-microchip && git status
cd /home/ritesh/Stage1/av_services && git status
```

## Push Sequence Rule

When you changed submodule content:

1. Commit and push submodule repo first.
2. Update root pointer commit second.
3. Push root repo last.

This avoids broken submodule references in root.
