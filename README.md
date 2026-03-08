# Xpra Client Flatpak

Flatpak packaging for the Xpra X11 forwarding client (client-only, version 6.4.3).

## Status: ✅ COMPLETED

This flatpak is fully functional and successfully builds pycairo and PyGObject from source to support Xpra's C extensions.

## Installation

```bash
git clone git@github.com:MrMEEE/org.xpra.Client.git
cd org.xpra.Client
flatpak-builder --user --install --ccache builddir org.xpra.Client.yaml
```

## Usage

```bash
flatpak run org.xpra.Client --version
# xpra v6.4.3

flatpak run org.xpra.Client attach ssh://user@host/display
```

## Build Solution

After multiple attempts, the successful approach was:

### 1. Build pycairo from source (meson)
- Not pip - needed for C headers
- Provides `/app/include/pycairo/`

### 2. Build PyGObject from source (meson)
- Not pip - needed for C headers  
- Provides `/app/include/pygobject-3.0/pygobject.h` and `pygobject-types.h`

### 3. Install Python dependencies with pip
- Network-enabled pip for prebuilt wheels
- Packages: paramiko, cryptography, pyopenssl, lz4, pillow, pyopengl, netifaces, zeroconf

### 4. Compile Xpra with proper C flags
- `CFLAGS="-I/app/include/pygobject-3.0 -I/app/include/pycairo"`
- Allows Cython extensions to find PyGObject headers

### 5. Install multiple icon sizes
- 48x48, 128x128, 192x192 pixels
- Required for appstream validation to pass

## Challenges Overcome

1. **libXres missing**: Added X11 Resource extension library
2. **Pip-installed PyGObject lacks headers**: Built from source with meson
3. **pycairo build requirements**: Built from source with meson (not pip)
4. **C extension compilation**: Added explicit CFLAGS for include paths
5. **AppStream icon validation**: Installed icon in multiple standard sizes (128x128 was key)

## Build Attempts History

- ❌ Attempt 1: flatpak-pip-generator → blocked by maturin requirement
- ❌ Attempt 2: Network pip + manual pkg-config files → missing C headers
- ✅ Attempt 3: Meson-built pycairo/PyGObject + network pip → SUCCESS

See build logs for detailed history:
- `build-meson.log` - First meson attempt
- `build-cflags.log` - Added C flags
- `build-success.log` - Final successful build

## Technical Details

- **Runtime**: org.freedesktop.Platform 25.08
- **SDK**: org.freedesktop.Sdk 25.08
- **Python**: 3.13
- **Build system**: Mixed (meson for PyGObject/pycairo, setup.py for Xpra, pip for deps)
- **Network required**: Yes (during build for pip dependencies)

## Repository

https://github.com/MrMEEE/org.xpra.Client

## Resources

- Xpra upstream: https://github.com/Xpra-org/xpra
- Xpra documentation: https://xpra.org/
- PyGObject: https://gnome.pages.gitlab.gnome.org/pygobject/
- pycairo: https://pycairo.readthedocs.io/

