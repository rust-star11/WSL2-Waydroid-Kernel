# WSL2 Waydroid Kernel

A custom Linux kernel for Windows Subsystem for Linux 2 (WSL2), built to support running Waydroid and Android userspace environments inside WSL.

This project follows the approach described in [onomatopellan's Waydroid in WSL2 guide](https://gist.github.com/onomatopellan/c5220c0efddaff69aaff77cca80b7b8e), using Microsoft's WSL2 Linux kernel as the foundation.

## Project Goals

- Enable the Linux kernel features required by Waydroid.
- Preserve compatibility with the WSL2 environment.
- Provide a reproducible kernel build and configuration record.
- Document installation, verification, and troubleshooting.
- Keep kernel changes focused on Android container support.

## Kernel Information

| Property | Value |
|---|---|
| Base project | [Microsoft WSL2 Linux Kernel](https://github.com/microsoft/WSL2-Linux-Kernel) |
| Intended platform | Windows 10/11 with WSL2 |
| Intended workload | Waydroid / Android userspace |
| Architecture | x86_64 (verify for each release) |
| Kernel release | Record the exact release with `uname -r` |
| Kernel image | `bzImage` |
| External module image | `modules.vhdx` or `modules.vhdx`-compatible image, if generated |

> The exact source revision, kernel version, configuration changes, and build environment should be recorded for every published release.

## Required Kernel Configuration

The original guide identifies Android Binder IPC as the essential kernel configuration for its approach.

```text
CONFIG_ANDROID_BINDER_IPC=y
CONFIG_ANDROID_BINDER_DEVICES="binder,hwbinder,vndbinder"
```

These settings enable Binder IPC and define the device names used by Android components.

Other configuration options may be needed depending on the kernel version, Waydroid image, and runtime environment. Do not assume that every Android-related kernel feature is enabled unless it has been verified in the actual build.

## Build Method

The intended build process follows Microsoft's WSL2 kernel build workflow:

1. Obtain the appropriate WSL2 Linux kernel source revision.
2. Start with the WSL configuration at `Microsoft/config-wsl`.
3. Enable the required Android Binder IPC options.
4. Build the kernel and install its modules into a staging directory.
5. Generate the kernel image and, if required, a modules VHDX image.
6. Record the configuration, source revision, build commands, and output checksums.

Upstream build reference:

[Microsoft WSL2 Linux Kernel](https://github.com/microsoft/WSL2-Linux-Kernel)

Detailed reference guide:

[Waydroid in WSL2 with sound](https://gist.github.com/onomatopellan/c5220c0efddaff69aaff77cca80b7b8e)

## Installation

Place the custom kernel image and any required module image in a stable directory on the Windows filesystem.

Configure WSL through WSL Settings or the appropriate Windows-side WSL configuration. For example:

```ini
[wsl2]
kernel=D:\\WSL-Kernels\\Waydroid\\bzImage
kernelModules=D:\\WSL-Kernels\\Waydroid\\modules.vhdx
```

Adjust the paths to match the actual files. The `kernelModules` entry is only needed when using a separate custom module image.

Restart WSL to load the configuration:

```powershell
wsl --shutdown
```

Then start the target Linux distribution and verify the running kernel:

```bash
uname -r
```

**Important:** The custom kernel is managed at the WSL virtual-machine level, not independently by each Linux distribution. A kernel configured through the standard WSL configuration can affect multiple WSL2 distributions. Keep a backup of the default configuration and kernel before making changes.

## Verification

Verify the kernel configuration and the presence of the required Binder devices.

Check the running kernel:

```bash
uname -a
```

Check the kernel configuration, if available:

```bash
if [ -r /proc/config.gz ]; then
    zcat /proc/config.gz | grep -E \
      'CONFIG_ANDROID_BINDER_IPC=|CONFIG_ANDROID_BINDER_DEVICES='
elif [ -r "/boot/config-$(uname -r)" ]; then
    grep -E \
      'CONFIG_ANDROID_BINDER_IPC=|CONFIG_ANDROID_BINDER_DEVICES=' \
      "/boot/config-$(uname -r)"
else
    echo "Kernel configuration is not exposed at the expected paths."
fi
```

Check Binder devices:

```bash
ls -l /dev/binder /dev/hwbinder /dev/vndbinder 2>/dev/null
```

Device nodes may be created by Waydroid or its setup scripts, so their absence alone does not conclusively establish that the kernel lacks Binder support.

## Waydroid Testing

After installing Waydroid and configuring its runtime environment, inspect its status:

```bash
waydroid status
```

Start the session and launch the UI using the procedure appropriate for the installed Waydroid and WSLg/Weston setup.

A successful test should record:

- Kernel release and architecture.
- Waydroid session and container status.
- Whether the Android UI opens and remains usable.
- Whether networking and audio work.
- Whether the result survives a WSL shutdown and restart.

## Known Limitations

- WSL2 is not a conventional Linux installation; hardware access and kernel integration differ from a native Linux system.
- Android container compatibility depends on kernel configuration, userspace components, and the Waydroid image.
- GPU acceleration, audio, networking, and display integration may require additional configuration.
- A kernel update can change compatibility with an existing Waydroid installation.
- The use of a custom kernel or module image through WSL's standard configuration is not isolated to one Linux distribution.

## Release Checklist

Before publishing a release, document:

- [ ] Upstream source commit or tag.
- [ ] Kernel version and release string.
- [ ] All intentional configuration changes.
- [ ] Build environment and reproducible commands.
- [ ] Kernel image and module image checksums.
- [ ] Tested WSL version and Windows version.
- [ ] Tested Linux distribution and Waydroid version.
- [ ] Verified Binder support and Android UI startup.
- [ ] Known issues and recovery instructions.

## Credits

- [Microsoft WSL2 Linux Kernel](https://github.com/microsoft/WSL2-Linux-Kernel)
- [onomatopellan — Waydroid in WSL2 with sound](https://gist.github.com/onomatopellan/c5220c0efddaff69aaff77cca80b7b8e)
- [Waydroid](https://waydro.id/)

This project documents a custom WSL2 kernel configuration and its intended use with Waydroid. It is not an official Microsoft or Waydroid kernel release.

