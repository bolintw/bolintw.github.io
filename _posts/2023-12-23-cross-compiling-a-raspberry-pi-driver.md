---
layout: post
title: "Cross-Compiling a Raspberry Pi Driver"
date: 2023-12-23 11:00:00 +0800
categories: [Linux, Driver]
tags: [linux, kernel, driver, cross-compiling, raspberry-pi, makefile, arm64]
image: /assets/img/2023-12-23-cross-compiling-a-raspberry-pi-driver/1_KBlZCLxp-oueUuM4OYT2kw.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E8%B7%A8%E5%B9%B3%E5%8F%B0%E7%B7%A8%E8%AD%AF%E6%A8%B9%E8%8E%93%E6%B4%BE%E7%9A%84driver-492eec6db994) and **translated** by Gemini pro 2.5.

---

After a bunch of trial and error, combined with asking GPT in Mandarin and then in English (which gave me slightly different answers), I finally succeeded in cross-compiling a usable RPi3 Driver.

Although there were still some warnings during compilation, I'm just happy that I can successfully install the driver. I'll deal with the warnings later.

### Preparing the Build Environment

1.  **You can only compile a Linux driver in a Linux environment.** You can either set up a separate Ubuntu system or use the WSL service on Windows. I've tested both, and they work.

2.  **Install the cross-compilation toolchain.**

```bash
sudo apt-get update
sudo apt-get install crossbuild-essential-arm64
```

3.  **Download the corresponding Kernel source.**

```bash
git clone --depth=1 https://github.com/raspberrypi/linux
cd linux
```

4.  **Prepare the required build modules/packages.**

```bash
# Default config file
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig

# How to adjust if needed
make menuconfig
```

The `bcm2711_defconfig` file might have a slightly different name. You can check the folder mentioned in the error message for the latest filename. These two commands will create a `.config` file.

Since I don't really know how to adjust the Kernel config, I just pressed Enter for all the default answers whenever the process asked for user confirmation.

Alternatively, you can copy the `.config` file directly from your Raspberry Pi. The command to download files from the Pi via SSH is as follows:

```bash
scp ssh_name@ssh_address:remote/date/path local/target/path
```

Once you have the `.config` file, you can use the following commands to prepare the rest of the required modules:

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- prepare
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules_prepare
```

5.  **Modify the Makefile's target path.**

```bash
KERNEL_PATH=~/Download/linux
MODULE_PATH=$(PWD)

obj-m := hello.o

all:
        make -C $(KERNEL_PATH) M=$(MODULE_PATH) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- modules

clean:
        make -C $(KERNEL_PATH) M=$(MODULE_PATH) ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- clean
```

---

There were still some warnings during compilation:

![Warning](/assets/img/2023-12-23-cross-compiling-a-raspberry-pi-driver/1_6T7wqGGyvnM2Ipi9cCGkhA.webp)

The contents of the cross-compiled driver (`modinfo`):

![modinfo](/assets/img/2023-12-23-cross-compiling-a-raspberry-pi-driver/1_ZubpwZMIiwBvOhmk_qXk1g.webp)

Comparing it with the driver compiled directly on the Pi:

![Comparation](/assets/img/2023-12-23-cross-compiling-a-raspberry-pi-driver/1_SOdDVk3RHToiqLoysCzLbQ.webp)

Although I ran into some compilation warnings, installing (`insmod`) and removing (`rmmod`) the driver both work normally.