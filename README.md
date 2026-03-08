# Xpra Client Flatpak (Work in Progress - INCOMPLETE)

Flatpak packaging for the Xpra X11 forwarding client (client-only).

## Status: Incomplete - Complex Build Requirements

This flatpak is currently incomplete due to significant build complexity. Multiple approaches have been attempted:

### Approach 1: flatpak-pip-generator (Attempted)
- **Tool**: Used flatpak-pip-generator to create python3-xpra-deps.yaml
- **Problem**: Generated manifest uses source distributions that require:
  - `maturin` (Rust Python build tool) for bcrypt/cryptography
  - `meson-python` for pycairo
  - Complex build chains for native extensions
- **Result**: Build tool dependency hell - hit missing maturin

### Approach 2: Network-enabled pip with wheels (Currently)
- **Method**: Use `--share=network` and pip to install prebuilt wheels
- **Progress**: Successfully installed Python packages (pycairo, PyGObject, etc.)
- **Problem**: Pip-installed packages don't provide:
  - pkg-config files (py3cairo.pc, pygobject-3.0.pc) - **Manually created**
  - C development headers (pygobject.h) - **Not available in SDK**
- **Blocker**: Xpra's setup.py compiles C extensions that need PyGObject headers:
  ```
  xpra/x11/gtk/bindings.c:1237:10: fatal error: pygobject-3.0/pygobject.h: No such file or directory
  ```

### What's Needed to Complete

To finish this flatpak, one would need to:

1. **Build pycairo from source** using meson (not pip):
   ```yaml
   - name: pycairo
     buildsystem: meson
     sources:
       - type: archive
         url: https://github.com/pygobject/pycairo/releases/download/v1.29.0/pycairo-1.29.0.tar.gz
         sha256: ...
   ```

2. **Build PyGObject from source** using meson (not pip):
   ```yaml
   - name: pygobject
     buildsystem: meson
     config-opts:
       - -Dpython=python3
     sources:
       - type: archive
         url: https://download.gnome.org/sources/pygobject/3.56/pygobject-3.56.0.tar.xz
         sha256: ...
   ```

3. **Install remaining Python deps** with pip (paramiko, cryptography, lz4, pillow, etc.)

4. **Build Xpra** with all dependencies now providing proper headers

This requires deep knowledge of:
- Meson build system and configuration
- PyGObject/pycairo build process
- Python C extension compilation
- Flatpak SDK library paths and linking

## Complexity Assessment

Xpra is significantly more complex than typical Python applications:
- **Heavy C Extensions**: Uses Cython and C extensions for performance-critical code
- **Deep System Integration**: Requires X11, Wayland, GStreamer, v4l2, libvpx, etc.
- **Build Dependencies**: Needs full PyGObject development stack with headers
- **No Official Flatpak**: Not available on Flathub (indicating difficulty level)

## Files Included

- `org.xpra.Client.yaml` - Main manifest (current approach: network pip + manual pkg-config)
- `org.xpra.Client.desktop` - Desktop entry
- `org.xpra.Client.metainfo.xml` - AppStream metadata  
- `python3-xpra-deps.yaml` - Generated dependencies from flatpak-pip-generator (abandoned due to maturin)
- Build logs documenting attempts:
  - `build-networkpip.log` - Discovered need for pkg-config files
  - `build-pygobject.log` - Created pkg-config files, discovered need for headers
  - `build-headers.log` - Added include paths, confirmed SDK lacks PyGObject headers

## Recommendations

1. **For experienced flatpak developers**: 
   - Build pycairo and PyGObject from source using meson
   - Reference GNOME flatpaks that use these libraries
   - May need to use flatpak shared-modules

2. **For others**: Consider alternatives:
   - Use Xpra's native distribution packages (deb/rpm/etc)
   - Use Xpra's official installer from xpra.org
   - Wait for official Xpra flatpak support from upstream
   - Contribute to a Flathub submission (collaborative effort recommended)

## Resources

- Xpra upstream: https://github.com/Xpra-org/xpra
- Flatpak-builder-tools: https://github.com/flatpak/flatpak-builder-tools
- Flatpak shared-modules: https://github.com/flathub/shared-modules
- PyGObject documentation: https://pygobject.readthedocs.io/
- This repository: https://github.com/MrMEEE/org.xpra.Client

## Building (Incomplete)

Current manifest will not complete successfully:
```bash
flatpak-builder --user --force-clean builddir org.xpra.Client.yaml
# Expect: failure at Xpra C extension compile step
```

