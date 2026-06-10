# sm-g965n-ksun

Linux kernel 4.9 source for Samsung Galaxy S9/S9+ (Exynos 9810) with KernelSU-Next integration.

## Project Overview

- **Device:** Samsung Galaxy S9 (starqlte/starlte), S9+ (star2qlte/star2lte)
- **SoC:** Exynos 9810
- **Kernel Version:** 4.9.118
- **Features:**
    - KernelSU-Next support
    - Samsung RKP (Real-time Kernel Protection) compatibility fixes
    - Optimized for Android 9 (P) and 10 (Q)

## Environment Setup

Building requires an `aarch64` cross-compiler (GCC 4.9 or Clang).

```bash
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-android-
export ANDROID_MAJOR_VERSION=q
```

## KernelSU-Next Integration

KernelSU-Next is integrated via manual hooks and RKP fixes.

### 1. Apply KSU Hooks
Apply manual hooks to core kernel files (fs/exec.c, fs/open.c, etc.):
```bash
python3 apply_ksu_hooks.py
```

### 2. Fix Samsung RKP Compatibility
Fix issues with Samsung's RKP when using KernelSU:
```bash
./fix-samsung-rkp-ksu.sh
```

## Building and Running

### 1. Configure
Use the default configuration for the Galaxy S9/S9+ (Exynos 9810):
```bash
make exynos9810-star2ltekor_defconfig
```

### 2. Build
Build the kernel image and modules:
```bash
make -j$(nproc)
```

### 3. Output
- **Kernel Image:** `arch/arm64/boot/Image` (or `Image.gz-dtb`)
- **Modules:** `drivers/*/*.ko`

## Development Conventions

- **Hooks:** Always use `apply_ksu_hooks.py` for KSU integration instead of manual patching to avoid errors.
- **RKP:** Ensure `fix-samsung-rkp-ksu.sh` is run if `CONFIG_RKP` or `CONFIG_UH` is enabled in the defconfig.
- **Defconfigs:** Modified defconfigs should be saved back to `arch/arm64/configs/`.
