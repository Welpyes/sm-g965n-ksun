# SM-G965N Kernel (Exynos9810 + KernelSU-Next)

Custom kernel for Samsung Galaxy S9+ Korea (SM-G965N) based on Samsung's vendor Linux 4.9.118 source with [KernelSU-Next](https://github.com/rifsxd/KernelSU-Next) 3.3.0 integration.

## Features

- **KernelSU-Next 3.3.0** (legacy branch, manual hook mode — kprobes disabled)
- QTAGUID network traffic accounting (per-UID data tracking for VPN/firewall apps)
- Droidspaces/LXC cgroup compatibility fixes
- Samsung RKP (Real-time Kernel Protection) compatibility fixes
- SELinux `u:r:su:s0` transition support
- ADB root via KernelSU

## Supported Devices

| Device | Codename | Chipset |
|--------|----------|---------|
| Samsung Galaxy S9+ (Korea) | star2ltekor | Exynos 9810 |

## Requirements

### Toolchain

Bundled GCC 4.9 Android NDK toolchain at `toolchain/gcc-cfp/gcc-ibv-jopp/aarch64-linux-android-4.9/`.

If building outside this repo, set `CROSS_COMPILE` to your `aarch64-linux-android-` toolchain prefix.

### Build Dependencies (Ubuntu/Debian)

```
bc bison build-essential cpio flex git kmod libelf-dev libssl-dev \
lz4 make openssl perl python3 tar unzip wget zip
```

## Local Build

```bash
export ARCH=arm64
export ANDROID_MAJOR_VERSION=q

make exynos9810-star2ltekor_defconfig
make -j$(nproc)
```

Or use the build script:

```bash
chmod +x build_kernel.sh
./build_kernel.sh
```

### Output

| File | Path |
|------|------|
| Kernel image | `arch/arm64/boot/Image` |
| Modules | `drivers/*/*.ko` |

### Clean

```bash
make clean
```

## GitHub Actions CI

A build workflow is included at `.github/workflows/build.yml`. To use it:

1. **Fork** this repository to your GitHub account
2. Go to **Actions** tab → enable workflows
3. Click **Run workflow**, provide:
   - **Branch**: your branch name (e.g. `ksun-3.3.0`)
   - **Apply KernelSU hooks**: `true` (if KSU patches not already applied to source)
   - **Silent build logs**: optional

Artifacts (kernel image, modules, build config/logs) will be uploaded after the build.

## KernelSU-Next Integration

This repo uses the **legacy branch** of KernelSU-Next (v3.3.0) designed for 4.9 kernels.

### How it works

- KSU source lives in `KernelSU-Next/kernel/` (symlinked via `drivers/kernelsu`)
- Built as `kernelsu.o` module linked into the kernel
- Hook mode: **manual** (kprobes unavailable on this kernel)
- Hook points injected by `apply_ksu_hooks.py` or `bsx.sh`

### Config options (in defconfig)

```
CONFIG_KSU=y
```

### KSU hook scripts

| Script | Description |
|--------|-------------|
| `apply_ksu_hooks.py` | Python script — injects KSU hooks into kernel source (preferred) |
| `bsx.sh` | Bash script — alternative hook injector by backslashxx |
| `patches/ksu-hooks.patch` | Pre-generated patch for KSU hooks |
| `patches/is_ksu_transition.patch` | SELinux `is_ksu_transition` declaration patch |

### Fix scripts

| Script | Description |
|--------|-------------|
| `fix-samsung-rkp-ksu.sh` | Fixes Samsung RKP `override_creds` const mismatch, adds missing RKP mount prototypes, removes GCC 10+ only warning flags, fixes `ns_get_path` return type for 4.9 |

## Patches

| Patch | Purpose |
|-------|---------|
| `01.fix_kernel_panic_in_xt_qtaguid.patch` | Fixes kernel panic in `xt_qtaguid` by adding `rcu_read_lock()` around `dev_get_stats()` and null-checking `net_dev` |
| `02.fix_restore_cgroup_file_prefix_handling.patch` | Restores cgroup file prefix handling for Droidspaces/LXC compatibility |
| `ksu-hooks.patch` | KernelSU hook injection points |
| `is_ksu_transition.patch` | SELinux `is_ksu_transition` declaration |

## VPN / Data Tracking Fix

`CONFIG_NETFILTER_XT_MATCH_OWNER` is **disabled** in the defconfig. This was required because it blocks `CONFIG_NETFILTER_XT_MATCH_QTAGUID` via Kconfig `depends on` dependency chain. With OWNER disabled, QTAGUID enables and per-UID data tracking (including VPN) works correctly.

## System Requirements

- **Kernel**: Linux 4.9.118 (Samsung vendor fork)
- **Architecture**: ARM64 (AArch64)
- **Toolchain**: GCC 4.9 `aarch64-linux-android-` (bundled)
- **RAM**: 4GB+ recommended for parallel builds
- **Disk**: ~15GB for full build tree

## Troubleshooting

### `YYLTYPE yylloc` error during DTC build

```bash
sed -i '/YYLTYPE yylloc;/d' scripts/dtc/dtc-lexer.lex.c_shipped
sed -i '/YYLTYPE yylloc;/d' scripts/dtc/dtc-lexer.lex.c
```

### GCC "unknown warning option" errors

The build scripts automatically strip `-Wno-gcc-compat` and `-Wno-int-conversion` flags (GCC 10+ only). If building manually with a newer GCC, run `fix-samsung-rkp-ksu.sh` first.

### Kernel panic on QTAGUID

Apply `patches/01.fix_kernel_panic_in_xt_qtaguid.patch` — the `dev_get_stats()` call requires `rcu_read_lock()` protection and a null check on `net_dev`.

## Credits

- [KernelSU-Next](https://github.com/rifsxd/KernelSU-Next) — KernelSU-Next by rifsxd
- [backslashxx](https://github.com/backslashxx) — `bsx.sh` hook injector
- JackA1ltman — `bsx.sh` author
- Samsung — Exynos 9810 vendor kernel source
