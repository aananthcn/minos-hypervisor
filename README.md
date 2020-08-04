Reference: https://github.com/minosproject/minos/blob/master/Documents/009-Test_Minos_on_Raspberry_3B.md

Download code and build images
==============================

export CMP_PATH=/opt/tools/aarch64-none-linux/bin/aarch64-none-linux-gnu-

    Build Minos
    -----------

    # git clone https://github.com/minosproject/minos-hypervisor.git
    # cd minos-hypervisor/
    # ARCH=aarch64 CROSS_COMPILE=${CMP_PATH} make rpi_3_defconfig
    # ARCH=aarch64 CROSS_COMPILE=${CMP_PATH} make && make dtbs && make mvm



    Build u-boot
    ------------

    # git clone https://github.com/u-boot/u-boot.git && cd u-boot
    # ARCH=arm64 CROSS_COMPILE=${CMP_PATH} make rpi_3_defconfig
    # ARCH=arm64 CROSS_COMPILE=${CMP_PATH} make -j8



    Build Kernel and Kernel modules
    -------------------------------

    # git clone https://github.com/minosproject/linux-raspberry.git && cd linux-raspberry
    # git checkout -b rpi3 origin/minos-rpi3
    # make ARCH=arm64 CROSS_COMPILE=${CMP_PATH} bcmrpi3_defconfig
    # make ARCH=arm64 CROSS_COMPILE=${CMP_PATH} Image -j8
    # make ARCH=arm64 CROSS_COMPILE=${CMP_PATH} modules dtbs -j8
    # sudo make ARCH=arm64 CROSS_COMPILE=${CMP_PATH} modules_install INSTALL_MOD_PATH=<mount-point>/rootfs
 
    # cp arch/arm64/boot/Image <mount-point>/boot/




    Get other images
    ----------------
    # git clone https://github.com/minosproject/minos-misc.git




Update SD card image
====================

    copy images
    -----------
    # cp u-boot/u-boot.bin /media/aananth/boot/
    # cp minos-hypervisor/minos.bin /media/aananth/boot/
    # cp minos-hypervisor/dtbs/bcm2837-rpi-3-b-plus.dtb /media/aananth/boot/minos.dtb
    # cp minos-misc/rpi-3b/vm0.dtb /media/aananth/boot/

    change the /media/minle/boo/config.txt to below content

    arm_control=0x200
    #dtoverlay=pi3-miniuart-bt
    enable_uart=1
    kernel=u-boot.bin

Boot System

Connect your raspberry4 with a usb serial, and enter into u-boot command line mode,

fatload mmc 0:1 0x28008000 minos.bin;
fatload mmc 0:1 0x29e00000 minos.dtb;
fatload mmc 0:1 0x80000 Image; 
fatload mmc 0:1 0x03e00000 vm0.dtb; 
booti 0x28008000 - 0x29e00000

minosload=fatload mmc 0:1 0x28008000 minos.bin; fatload mmc 0:1 0x29e00000 rpi3b.dtb; fatload mmc 0:1 0x80000 Image; fatload mmc 0:1 0x03e00000 vm0.dtb
setenv bootargs earlyprintk console=tty0 console=ttyAMA0 root=/dev/mmcblk0p2 rootfstype=ext4 rootwait noinitrd
setenv bootargs earlyprintk console=tty0 console=ttyAMA0 root=/dev/mmcblk0p2 rootfstype=ext4 fsck.repair=yes rootwait noinitrd video=HDMI-A-1:1600x900M@60
setenv bootargs "console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 fsck.repair=yes rootwait video=HDMI-A-1:1600x900M@60" 

You can write below command to u-boot's common/Kconfig, then u-boot will auto load these image and jump to Minos (need make clean and recompile u-boot)

diff --git a/common/Kconfig b/common/Kconfig
index 46e4193fc8..67c8771c1b 100644
--- a/common/Kconfig
+++ b/common/Kconfig
@@ -396,7 +396,7 @@ config USE_BOOTCOMMAND
 config BOOTCOMMAND
        string "bootcmd value"
        depends on USE_BOOTCOMMAND
-       default "run distro_bootcmd" if DISTRO_DEFAULTS
+       default "fatload mmc 0:1 0x28008000 minos.bin; fatload mmc 0:1 0x29e00000 minos.dtb; fatload mmc 0:1 0x80000 Image; fatload mmc 0:1 0x03e00000 vm0.dtb; booti 0x28008000 - 0x29e00000" if DISTRO_DEFAULTS
        help
          This is the string of commands that will be used as bootcmd and if
          AUTOBOOT is set, automatically run.

Create Guest VM using MVM

copy below images to your rpi3 system

    mvm - minos/tools/mvm/mvm
    aarch32-boot.img - minos-misc
    aarch64-boot.img - minos-misc

use below command to create a 64bit Guest VM

sudo ./mvm run_as_daemon memory=64M vm_name=linux vm_os=linux vcpus=2 os-64bit bootimage=aarch64-boot.img cmdline="console=hvc0 loglevel=8 consolelog=9" gic=gicv2 device@virtio-console,backend=@pty

and below command can create a 32bit Guest VM with the virtioblk rootfs

sudo ./mvm run_as_daemon memory=64M vm_name=linux vm_os=linux vcpus=2 bootimage=boot32.img no-ramdisk cmdline="console=hvc0 loglevel=8 consolelog=9 root=/dev/vda2 rw" gic=g
