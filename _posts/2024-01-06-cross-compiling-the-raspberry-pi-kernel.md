---
layout: post
title: "Cross-Compiling the Raspberry Pi Kernel"
date: 2024-01-06 10:00:00 +0800
categories: [Linux, Kernel]
tags: [linux, kernel, cross-compiling, raspberry-pi, wsl, modules, dts]
image: /assets/img/2024-01-06-cross-compiling-the-raspberry-pi-kernel/0_XvF29ZrCnJkL-e2r.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E4%BA%A4%E5%8F%89%E7%B7%A8%E8%AD%AF%E6%A8%B9%E8%8E%93%E6%B4%BEkernel-0de7461fa2a8) and **translated** by Gemini pro 2.5.

---

Building modules is heavily related to compiling the kernel, so I took a detour to figure out the kernel compilation process first. My target is a Raspberry Pi 4, the OS is RPi OS 64bit Lite, and my build environment is WSL in Windows.

First, here is the reference I used, the official Raspberry Pi kernel compilation guide:
> [https://www.raspberrypi.com/documentation/computers/linux_kernel.html](https://www.raspberrypi.com/documentation/computers/linux_kernel.html)

You'll need to change the command parameters based on your target. The following parameters are all based on my target, so read with caution.

### 1. Install Miscellaneous Build Tools

```bash
sudo apt install git bc bison flex libssl-dev make libc6-dev libncurses5-dev
```

### 2. Install the Cross-Compilation Toolchain

```bash
sudo apt install crossbuild-essential-arm64
```

### 3. Download the Kernel Source

```bash
git clone --depth=1 https://github.com/raspberrypi/linux
```

### 4. Set Up the Kernel Configuration

```bash
cd linux

# default config
KERNEL=kernel8
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- bcm2711_defconfig
```

You can also use `menuconfig` to edit the settings:

```bash
# use menuconfig to config
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- menuconfig
```

Or you can edit the config file directly:

```bash
# edit file directly
vim .config
```

It's recommended to modify the **General Setup -> Local Version** item. This makes it easy to confirm if the kernel has been updated after installation.

### 5. Start Compiling

```bash
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) Image modules dtbs
```

If you compile directly on the Pi, you don't have to deal with cross-platform issues, but the compile time is about 2-3 hours. Using WSL, it took me about 30 minutes.

That's the build process. If it finishes without any errors, it's successful. Next is deploying the cross-compiled kernel to the Pi.

### 1. Transfer the Image File

```bash
scp arch/arm64/boot/Image <user>@raspberrypi.local:~/tmp
```

### 2. Transfer the Device Tree Files

```bash
scp arch/arm64/boot/dts <user>@raspberrypi.local:~/tmp
```

### 3. Install Modules Locally (in WSL), Compress, and Transfer

```bash
# install to local wsl
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j$(nproc) INSTALL_MOD_PATH=../modules modules_install

# compress the folder
cd ..
tar jcvf modules.tar.bz2 modules/

# send to the rpi
scp modules.tar.bz2 <user>@raspberrypi.local:~/tmp
```

After all files are on the Pi, start deploying them. The following operations are on the Raspberry Pi, so SSH in first.

### 4. Connect to the Raspberry Pi

```bash
ssh <user>@raspbberrypi.local
```

### 5. Deploy the Image

```bash
sudo cp tmp/Imgae /boot/firmware/<img_name.img>
```

You can either overwrite the default `img` file or give it a new name. If you use a new name, you must edit `/boot/firmware/config.txt` and add a new line `kernel=<img_name.img>` to specify the boot kernel file.

### 6. Deploy the Device Tree

```bash
sudo cp tmp/dts/broadcom/*.dtb /boot/firmware/
sudo cp tmp/dts/overlays/*.dtb* /boot/firmware/overlays/
sudo cp tmp/dts/overlays/README /boot/firmware/overlays/
```

### 7. Deploy the Modules

```bash
# decompress
cd tmp
tar jxf modules.tar.bz2

# deploy files
sudo cp -r modules/lib/modules/* /lib/modules/
sudo cp -r modules/lib/firmware/* /lib/firmware/
```

### 8. Reboot and Verify

```bash
sudo reboot

# wait rpi boot then connect
ssh <user>@raspberrypi.local

# check the kernel
uname -r
```

After successfully following these steps, cross-compiling a *module* is just following the same pattern.

---

### Setting Up the Pi for Native Module Compilation

To enable the Pi to compile modules *natively* against this custom kernel, we still need to prepare the required files. If you followed the steps above, you'll notice that `/lib/modules/<kernel>/build` on the Pi is empty (or a broken symlink). This is because it was a symlink to the build directory on the remote build system (WSL).

#### 1. Clean the Build, Package, and Transfer

```bash
# clear or it will be too large
make mrproper

# compress
cd ..
tar zcvf linux.tar.gz linux/

# send to the rpi
scp linux.tar.gz <user>@raspberrypi.local:~/folder/you/want/to/put
```

#### 2. On the Pi, Unzip and Prepare Only Module-Related Components

```bash
# decompress
tar zxvf linux.tar.gz
cd linux

# make modules
make -j$(nproc) prepare
make -j$(nproc) modules_prepare
```

#### 3. Move to the Correct Location and Create the Symlink

```bash
cd ..
mv linux/ <header_name>/
sudo mv <header_name> /usr/src/

sudo rm /lib/modules/<kernel>/build
sudo ln -s /usr/src/<header_name> /lib/modules/<kernel>/build
```

And with that, you can now compile modules natively on the Raspberry Pi.