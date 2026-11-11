dts-overlays
============

The dts-overlays project builds device tree overlays that the kernel loads at runtime.

Kernel
======

The kernel needs these config options:

```
CONFIG_OF_CONFIGFS
CONFIG_OVERLAY_FS
CONFIG_OVERLAY_FS_INDEX
CONFIG_OVERLAY_FS_XINO_AUTO
CONFIG_OVERLAY_FS_METACOPY
```

Microchip BSP release 2026.03 or later includes CONFIG_OF_CONFIGFS.

Older releases may work after applying the patch
'0001-OF-DT-Overlay-configfs-interface.patch' to the kernel.

Build
=====

Required tools:

- gcc: runs the C preprocessor to resolve the header includes.
- dtc: the device tree compiler.
- cmake: starts the build (version 3.5 or later).

To compile all overlays:
```
cmake -B build
cmake --build build
```

To compile a single overlay, use the file name without the extension as the target:
```
cmake --build build --target my_overlay
```

The .dtbo files are written to the build directory. Copy them to the target filesystem.

Extend
======

To add a new overlay, add a file with the extension .dtso to the top directory.

Load-dtbo
=========

```
mount -t configfs none /sys/kernel/config
mkdir -p /sys/kernel/config/device-tree/overlays/tsys01
cat my_overlay.dtbo > /sys/kernel/config/device-tree/overlays/tsys01/dtbo
```
