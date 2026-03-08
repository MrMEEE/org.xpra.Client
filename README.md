# Xpra Client Flatpak

This is a work-in-progress flatpak for the Xpra client.

## Current Status

The flatpak manifest is created but the build is not yet complete. The main challenges are:

1. **Python Dependencies**: Xpra has extensive Python dependencies that need to be either:
   - Pre-downloaded with correct URLs and SHA256 checksums
   - Built from source with proper build tools (meson-python, etc.)
   
2. **System Libraries**: The build requires:
   - libXres (X11 Resource extension) - added ✓
   - pycairo (Python Cairo bindings) with meson build system
   - PyGObject (Python GObject bindings) with complex build requirements
   
3. **Build Complexity**: Xpra's setup.py checks for many system libraries during the build, and pip's dependency resolution tries to build everything from source.

## Next Steps

To complete this flatpak:

1. **Use flatpak-pip-generator**: This tool can automatically generate Flatpak manifests for Python dependencies:
   ```bash
   pip3 install flatpak-pip-generator
   flatpak-pip-generator --runtime=org.freedesktop.Platform//25.08 xpra
   ```

2. **Or manually specify all dependencies**: Add pre-built wheels for all Python packages:
   - Cython
   - pycairo
   - PyGObject  
   - paramiko
   - cryptography
   - pyopenssl
   - All other dependencies from xpra's pyproject.toml [gui] and [cli] sections

3. **Consider using flatpak-builder-tools**: https://github.com/flatpak/flatpak-builder-tools
   Contains scripts to generate dependency manifests for Python, Node.js, Cargo, etc.

## Alternative Approach

Another option would be to contribute this to Flathub and work with the community to properly package all dependencies.

## Files

- `org.xpra.Client.yaml` - Flatpak manifest (work in progress)
- `org.xpra.Client.desktop` - Desktop entry
- `org.xpra.Client.metainfo.xml` - AppStream metadata

## Testing

Once dependencies are resolved, test with:
```bash
flatpak-builder --user --install builddir org.xpra.Client.yaml
flatpak run org.xpra.Client
```
