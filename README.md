# Linux CachyOS T2 Kernel Build Guide

This repository contains the PKGBUILD for building the CachyOS Linux kernel with T2 Mac support. This kernel includes performance optimizations, LTO compilation, and support for Apple T2 chip hardware. Tested on MBP 16,1 with Omarchy

## Features

- **T2 Mac Support**: Support for Apple T2 chip hardware
- **BORE Scheduler**: Burst-Oriented Response Enhancer for improved responsiveness
- **LTO Optimization**: Link Time Optimization for better performance
- **Multiple CPU Schedulers**: BORE, BMQ, EEVDF, and RT options
- **Performance Optimizations**: O3 optimization, performance governor defaults
- **AutoFDO Support**: Profile-guided optimization capabilities
- **Propeller Support**: Advanced optimization with Propeller

## Prerequisites

### Required Dependencies

```bash
# Install base dependencies
sudo pacman -S base-devel git

# Install kernel build dependencies
sudo pacman -S bc cpio gettext libelf pahole perl python rust rust-bindgen rust-src tar xz zstd

# For LTO builds (required - this kernel uses LTO by default)
sudo pacman -S clang llvm lld

# For ZFS support (optional)
sudo pacman -S git
```

### System Requirements

- **Architecture**: x86_64
- **RAM**: Minimum 8GB (16GB recommended for LTO builds)
- **Storage**: At least 20GB free space
- **CPU**: Multi-core processor recommended

## Building the Kernel

### Step 1: Clone and Prepare

```bash
# Clone the repository
git clone <your-repo-url>
cd linux-cachyos-t2

# Update checksums (CRITICAL FIRST STEP)
updpkgsums PKGBUILD
```

**IMPORTANT**: Always run `updpkgsums PKGBUILD` first to ensure all source files have correct checksums for verification. **This command must be run every time you modify the PKGBUILD or config files.**

### Step 2: Configure Build Options

Edit the PKGBUILD file to customize your build. Key options include:

#### T2 Mac Support
```bash
# Enable T2 Mac support (default: yes)
_enable_t2=yes
```

#### CPU Scheduler Options
```bash
# Choose one scheduler:
_cpusched=cachyos    # BORE scheduler (default)
_cpusched=bore       # Pure BORE scheduler
_cpusched=bmq        # BMQ scheduler
_cpusched=eevdf      # EEVDF scheduler
_cpusched=rt         # Real-time with EEVDF
_cpusched=rt-bore    # Real-time with BORE
_cpusched=hardened   # Hardened BORE
```

#### Compiler Optimizations
```bash
# LTO (Link Time Optimization) - ENABLED BY DEFAULT
_use_llvm_lto=thin   # Default: thin. Options: none, thin, full, thin-dist

# CPU-specific optimizations
_processor_opt=generic  # Options: native, zen4, generic

# Enable O3 optimization
_cc_harder=yes
```

#### Performance Settings
```bash
# Tick rate (Hz)
_HZ_ticks=1000       # Options: 100, 250, 300, 500, 600, 750, 1000

# Preemption
_preempt=full        # Options: full, lazy, voluntary, none

# Transparent Hugepages
_hugepage=always     # Options: always, madvise

# Performance governor
_per_gov=yes
```

#### Optional Features
```bash
# Build ZFS module
_build_zfs=no

# Build NVIDIA modules
_build_nvidia=no
_build_nvidia_open=no

# Build debug package
_build_debug=no

# AutoFDO optimization
_autofdo=no
```

### Step 3: Build the Kernel

```bash
# Build with default settings (recommended for first build)
makepkg -s

# Build without confirmation prompts
makepkg -s --noconfirm

# Build without installing dependencies (if already installed)
makepkg -s --nodeps

# Clean build (removes previous build artifacts)
makepkg -s -c
```

### Step 4: Install the Kernel

```bash
# Install the built packages
sudo pacman -U linux-cachyos-t2-*.pkg.tar.zst
sudo pacman -U linux-cachyos-t2-headers-*.pkg.tar.zst

# Update bootloader (if using GRUB)
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Or update systemd-boot entries
sudo bootctl update
```

## Package Information

### Generated Packages

- `linux-cachyos-t2`: Main kernel package
- `linux-cachyos-t2-headers`: Kernel headers for module building
- `linux-cachyos-t2-dbg`: Debug symbols (if enabled)
- `linux-cachyos-t2-zfs`: ZFS module (if enabled)
- `linux-cachyos-t2-nvidia`: NVIDIA module (if enabled)

### Package Naming

The package name depends on LTO configuration:
- **LTO enabled (default)**: `linux-cachyos-lto-t2`
- GCC with suffix: `linux-cachyos-gcc-t2`
- LTO disabled: `linux-cachyos-t2`

## Troubleshooting

### Common Issues

#### Build Fails with Memory Error
```bash
# Reduce parallel jobs
MAKEFLAGS="-j2" makepkg -s

# Or limit memory usage
export MAKEFLAGS="-j$(nproc --ignore=1)"
```

#### T2 Patches Conflict
The PKGBUILD includes intelligent patch conflict resolution. If you encounter issues:
- Check that T2 patches are compatible with the kernel version
- Verify the T2 patch hash in the PKGBUILD

#### LTO Build Issues
```bash
# Disable LTO for troubleshooting (kernel uses LTO by default)
_use_llvm_lto=none
```

#### Module Loading Issues
```bash
# Rebuild initramfs
sudo mkinitcpio -p linux-cachyos-t2

# Check module dependencies
depmod -a
```

### Debug Information

Enable debug package for troubleshooting:
```bash
_build_debug=yes
```

This creates `linux-cachyos-t2-dbg` with unstripped vmlinux.

## Maintenance

### Updating the Kernel

```bash
# Update checksums (REQUIRED after any PKGBUILD or config changes)
updpkgsums PKGBUILD

# Clean previous build
makepkg -c

# Rebuild
makepkg -s

# Install new version
sudo pacman -U linux-cachyos-t2-*.pkg.tar.zst
```

### Cleaning Up

```bash
# Remove build artifacts
makepkg -c

# Remove installed packages
sudo pacman -R linux-cachyos-t2 linux-cachyos-t2-headers

# Clean package cache
sudo pacman -Sc
```

## Support

- **CachyOS Community**: [CachyOS Website](https://cachyos.org)
- **Kernel Documentation**: [Linux Kernel Documentation](https://www.kernel.org/doc/)
- **T2 Linux Project**: [T2 Linux GitHub](https://github.com/t2linux/linux-t2-patches)

## License

This kernel is based on the Linux kernel and follows the GPL-2.0 license. Additional patches and optimizations are provided under their respective licenses.

---

**Note**: Always backup your system before installing a custom kernel. Keep your original kernel as a fallback option in your bootloader.
