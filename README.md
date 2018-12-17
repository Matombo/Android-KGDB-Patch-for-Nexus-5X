# Android KGDB Patch for Nexus 5X

Kernel patch to get KGDB working on the Nexus 5X.

For background, please see associated blog post at http://www.contextis.com/resources/blog/kgdb-android-debugging-kernel-boss

- `git clone https://android.googlesource.com/kernel/msm`
- `git checkout android-msm-bullhead-3.10-nougat`
- Apply this patch.
- `export ARCH=arm64 && export CROSS_COMPILE=<path_to_ndk_r17c>/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-`
- `make bullhead_kgdb_defconfig && make`
- Create your boot image, passing console arguments e.g. to update a stock image I used: `abootimg -u boot.img -k zImage-dtb -c 'cmdline=console=ttyHSL0,115200,n8 kgdboc=ttyHSL0,115200'`
- Flash boot.img to the device.
- Open a shell (adb shell), su to root, then type: `echo -n g > /proc/sysrq-trigger` to make the kernel break
- On your host machine fire up GDB (you'll need a working version of GDB cross-compiled for ARM, ndk_r10e has a prebuilt binary included): `<path_to_ndk_r10e>/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-gdb <path_to_kernel_repo>/vmlinux`
- Inside GDB connect to device: `(gdb) target remote /dev/ttyUSB0`

You should hit the KGDB breakpoint and be able to continue, examine memory, etc.

More verbose explanation of the patch:

    Add poll commands to Nexus 5X serial driver to make KGDB work.
    
    Also fix compiling issue when deactivating CONFIG_MSM_WATCHDOG_V2. Which
    is necessary because the watchdog would trigger a reboot while KGDB
    breaks execution.
    
    To compile with KGDB use bullhead_kgdb_defconfig or enable CONFIG_KGDB
    and CONFIG_KGDB_SERIAL_CONSOLE manually while deactivating
    CONFIG_MSM_WATCHDOG_V2.
    
    You can also enable CONFIG_KGDB_KDB to debug directly from the serial
    console. If it is enable you can still also connect using an external
    GDB but you need to close the serial console first in this case.
