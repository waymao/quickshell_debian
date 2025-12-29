# Debian Package Build Notes for Quickshell

## Package Information
- **Package Name**: quickshell
- **Version**: 0.2.1-1
- **Architecture**: any (will build for your architecture)
- **Section**: x11

## Build Configuration

### Features Enabled
All upstream features are enabled except:
- **Crash Reporter**: Disabled (requires libgoogle-breakpad-dev which is not available in Debian repos)

### Build Dependencies Fixed
The following package names were corrected for Debian Trixie:
- `qt6-shadertools-dev` (instead of libqt6shadertools6-dev)
- `libpolkit-agent-1-dev` (added, required by polkit feature)

## Building the Package

### Prerequisites
```bash
sudo apt-get install devscripts equivs
```

### Build Steps

1. **Install build dependencies** (if not already done):
```bash
sudo apt-get install cmake ninja-build pkg-config \
    qt6-base-dev qt6-declarative-dev qt6-declarative-private-dev \
    qt6-wayland-dev qt6-wayland-private-dev qt6-tools-dev \
    qt6-shadertools-dev spirv-tools libcli11-dev libjemalloc-dev \
    libwayland-dev wayland-protocols libdrm-dev libgbm-dev \
    libxcb1-dev libpipewire-0.3-dev libpam0g-dev \
    libpolkit-agent-1-dev libpolkit-gobject-1-dev libglib2.0-dev
```

2. **Build the package**:
```bash
dpkg-buildpackage -us -uc -b
```

The `-us -uc` flags skip signing (you can sign later if needed).
The `-b` flag builds binary-only (no source package).

3. **Install the package** (after successful build):
```bash
cd ..
sudo dpkg -i quickshell_0.2.1-1_amd64.deb
```

### Output Files
After a successful build, you'll find in the parent directory:
- `quickshell_0.2.1-1_amd64.deb` - Main package
- `quickshell-dbgsym_0.2.1-1_amd64.deb` - Debug symbols package
- `quickshell_0.2.1-1_amd64.buildinfo` - Build information
- `quickshell_0.2.1-1_amd64.changes` - Changes file

## Customization

### Change Build Options
Edit `debian/rules` to modify CMake configuration options. For example, to disable a feature:

```make
override_dh_auto_configure:
    dh_auto_configure -- \
        -GNinja \
        -DCMAKE_BUILD_TYPE=RelWithDebInfo \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DDISTRIBUTOR="Debian" \
        -DDISTRIBUTOR_DEBUGINFO_AVAILABLE=YES \
        -DCRASH_REPORTER=OFF \
        -DSERVICE_PIPEWIRE=OFF \  # Add this to disable PipeWire
        -DINSTALL_QML_PREFIX=lib/$(DEB_HOST_MULTIARCH)/qt6/qml
```

See `BUILD.md` in the source tree for all available options.

### Update Maintainer Information
Before building for distribution, update:
- `debian/control` - Maintainer field
- `debian/changelog` - Your name and email
- `debian/copyright` - Your copyright information

## Troubleshooting

### Missing Dependencies
If you get dependency errors, check package names for your Debian/Ubuntu version:
```bash
apt-cache search <package-name>
```

### Build Failures
Check the build log for errors:
```bash
less /tmp/build.log
```

Common issues:
- Missing private Qt headers packages
- Incorrect package names for your distro version
- Qt version incompatibility (requires Qt 6.6+)

## Notes
- The `debian/compat` file was removed (modern Debian uses debhelper-compat in control file)
- Debug symbols are automatically generated in a separate package
- The build uses Ninja for faster compilation
