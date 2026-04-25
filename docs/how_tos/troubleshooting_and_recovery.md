# Troubleshooting and Recovery

## Purpose

This guide captures issues encountered and proven fixes for this workspace.

## 1. GitHub Push Authentication Failure

Symptom:
- `Invalid username or token. Password authentication is not supported`

Fix:
- Use token-based HTTPS or SSH auth.
- Push submodule repos first, root repo second.

## 2. Submodule Dirty State from Line Endings

Symptom:
- Massive modified files in submodules after Windows-side operations.

Fix:

```bash
cd /home/ritesh/Stage1
wsl bash -lc '
  cd /home/ritesh/Stage1 &&
  git config core.autocrlf false &&
  git -C buildroot config core.autocrlf false &&
  git -C buildroot-external-microchip config core.autocrlf false
'
```

Run git operations for submodules from WSL paths when possible.

## 3. Kconfig Parser Warnings on New Package

Symptom:
- `ignoring unsupported character` in package `Config.in`

Cause:
- CRLF line endings.

Fix:

```bash
cd /home/ritesh/Stage1
wsl bash -lc 'sed -i "s/\r$//" buildroot-external-microchip/package/av-services/Config.in buildroot-external-microchip/package/av-services/av-services.mk buildroot-external-microchip/package/av-services/files/*.conf'
```

## 4. Buildroot PATH Check Failure in WSL

Symptom:
- `Your PATH contains spaces...`

Fix:

```bash
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

Run Buildroot builds with sanitized PATH.

## 5. fakeroot preload library not found

Symptom:
- `fakeroot: preload library libfakeroot.so not found`

Cause:
- Host fakeroot script baked with different absolute prefix.

Fix used:

```bash
ln -s /home/ritesh/Stage1/buildroot /home/ritesh/buildroot
```

Then rebuild with sanitized PATH.

## 6. Suggested Hardening

1. Build from native WSL path and shell only.
2. Keep all package files LF-only.
3. Add CI step to run:
   - `make <defconfig>`
   - `make av-services`
4. Add first-boot smoke script to verify:
   - `av-core.target active`
   - `av-core-orchestrator running`
   - expected binaries and config files present
