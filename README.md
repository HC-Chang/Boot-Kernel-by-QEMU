# Boot-Kernel-by-QEMU

## Purpose

	從無到有建立 linux 系統
	使用 QEMU 模擬，降低學習的硬體門檻

## Environment - Windows
- wsl2 distribution: Ubuntu20.04LTS
- U-Boot: 2022.04-rc5
- Linux Kernel: 4.14.275
- Busybox: 1_35_stable

## Virtual Hardware
- [Arm Versatile Express boards - vexpress-a9]

## Needed
1. u-boot
1. kernel
> 1. kernel - zImage
> 1. dtb - 設備樹
1. rootfs
> 1. busybox - basic Unix tool
> 1. make rootfs

## Flow
u-boot -> kernel -> rootfs

## pre-install
	#bash
	sudo apt-get update && sudo apt upgrade
	sudo apt-get -y install build-essential curl git
	sudo apt-get -y install libncurses5-dev libssl-dev 
	sudo apt-get -y install gcc-arm-linux-gnueabi g++-arm-linux-gnueabi
	sudo apt-get -y install qemu
	sudo apt-get -y install qemu-system-qemu
	sudo apt-get -y install bison flex

## 1. u-boot
	#bash
	sudo make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- vexpress_ca9x4_defconfig
	sudo make -j32 ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- all

## 2. kernel
> 1. build kernel

	#bash
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- vexpress_defconfig
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- all

> 2. find dtb

	// you can find the hardware dtb in the document
	\linux-4.14.275\arch\arm\boot\dts

## 4. rootfs
> 1. build busybox

	#bash
	cd busybox
	make ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- menuconfig
	// Busybox Settings —> Build Options —> [*] Build Busybox as a static binary (no shared libs)
	
	make -j32 ARCH=arm CROSS_COMPILE=arm-linux-gnueabi- install

> 2. make rootfs

	mkdir rootfs
	cd rootfs
	mkdir proc sys dev etc tmp

	// copy files from busybox to rootfs
	cp -a ../../busybox-1_35_stable/_install/* .
	cp -a ../../busybox-1_35_stable/examples/bootfloppy/etc/* ./etc
	
	// edit inittab
	cd /rootfs/etc
	tty2::askfirst:-/bin/sh -> ::tty2::askfirst:-/bin/sh

	// make rootfs.ext3
	dd if=/dev/zero of=rootfs.ext3 bs=1M count=32
	mkfs.ext3 rootfs.ext3

	// copy files from rootfs to tmpfs (mount rootfs.ext3)
	mkdir -p tmpfs
	sudo mount rootfs.ext3 tmpfs/
	cp -r ./rootfs/* tmpfs/
	sudo umount tmpfs

## demo

	#bash
	// test u-boot
	qemu-system-arm -M vexpress-a9 -dtb vexpress-v2p-ca9.dtb -kernel u-boot -drive if=sd,index=0,file=rootfs.ext3,format=raw -append "root=/dev/mmcblk0 console=tty0" -smp 4 -nographic

	// test kernel
	qemu-system-arm -M vexpress-a9 -dtb vexpress-v2p-ca9.dtb -kernel zImage -drive if=sd,index=0,file=rootfs.ext3,format=raw -append "ro
	ot=/dev/mmcblk0 console=tty0" -smp 4 -nographic

	// exit qemu
	ctrl + a
	x

---

## NOTE:
	
**test kernel error** -> audio: Failed to create voice 'lm4549.out' 

**solve** -> 
1. -enable-kvm
1. export DISPLAY to Xming (X: -nographic)

---

## Ref:
- [Arm Versatile Express boards - vexpress-a9]
- [用Qemu模拟vexpress-a9 （一） --- 搭建Linux kernel调试环境]


[Arm Versatile Express boards - vexpress-a9]: https://qemu-project.gitlab.io/qemu/system/arm/vexpress.html
[用Qemu模拟vexpress-a9 （一） --- 搭建Linux kernel调试环境]: https://www.cnblogs.com/pengdonglin137/p/5023342.html