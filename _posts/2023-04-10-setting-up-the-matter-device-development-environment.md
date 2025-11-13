---
layout: post
title: "Setting Up the Matter Device Development Environment"
date: 2023-04-10 10:00:00 +0800
categories: [DIY, Matter]
tags: [matter, esp-idf, esp32, iot]
image: /assets/img/2023-04-10-setting-up-the-matter-device-development-environment/1_3U6eMsZGA56U5oQSFY8F_A.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/matter%E8%A3%9D%E7%BD%AE%E9%96%8B%E7%99%BC%E7%92%B0%E5%A2%83%E5%BB%BA%E7%AB%8B-9cded4167879) and **translated** by Gemini pro 2.5.

---

A while ago, my phone updated to iOS 16.4, and my custom-made HomeKit curtain controller suddenly went offline. Although I later found that simply re-pairing it fixed the issue, I couldn't help but worry. What if a future Apple update completely drops support for the HAP protocol? (Unlikely, but still...)

This concern sparked my journey to start learning and documenting Matter.

Matter is the open-source IoT protocol jointly developed by all the major smart device leaders (Apple, Google, Amazon, Zigbee). If you want to build a cross-platform and future-proof smart device, Matter is undoubtedly the protocol of choice.

Here's a record of the "landmines" I stepped on while setting up my Matter development environment, so I can look back on this if I ever need to set it up again.

### The (Bad) Advice from ChatGPT

First, I asked ChatGPT for a simple tutorial on using an ESP32 with Matter. It gave me a delightfully easy-looking list of four or five steps:

1.  Install and configure ESP-IDF (for compiling ESP32 projects)
2.  Install and configure Matter
3.  Compile one of the built-in Matter examples
4.  Flash to the ESP32 and test

After *actually* trying this, I found it was **completely unworkable**. In the end, I had to go back and meticulously Google the details myself, only to discover that Matter development is currently only supported on Mac and Linux.

### The Right Environment: WSL

I started by installing Ubuntu 20.04 in VMWare and followed the steps again. This time I got closer, but still got stuck at the compilation phase.

Then I discovered that Windows 11 has **WSL (Windows Subsystem for Linux)**. Installing a Linux environment with it is incredibly fast. Although it's CLI-only, it's a good way to force myself to learn Linux commands. Plus, when I really need to edit a file or search, WSL can directly open Windows File Explorer, Notepad, or even VS Code. It's incredibly convenient.

### Defusing the Landmines

After getting the Linux environment sorted, the next step was defusing the landmines I found on the road to installing Matter.

#### Landmine #1: ESP-IDF Version Conflict
First, the ESP-IDF version supported by Matter seems to be **only up to v4.4.3**. Without thinking, I installed the latest `v5.0.1` of ESP-IDF, and it just wouldn't compile, no matter what I did.

#### Landmine #2: Missing `gn`
A required library, `gn`, was not installed. This also causes the compilation to get stuck later on.

#### Landmine #3: WSL Serial Port Permissions
In the WSL environment, the Serial Port needs `USBIPD` to be installed to map the Windows COM Port into WSL. On top of that, you have to **change the port's user permissions**; otherwise, ESP-IDF will fail to find the port during the flashing process.

### Success!

After flashing the Matter example, I checked the documentation and found that to connect a Matter device to HomeKit, you need an Apple TV or HomePod as a hub. Luckily, I have a HomePod Mini at home. After a few more tries, I successfully added the Matter device to my Home app!

Here are the commands that solved the key issues:

```bash
# Supplement: Install gn
sudo apt-get install gn

# Change port permissions (replace ttyUSB0 with your port)
sudo chmod a+rw /dev/ttyUSB0
```

Once these few points were handled, I was able to compile and flash the Matter examples normally.

I've noticed there's very little documentation on this in Chinese online. I'm not sure if the technology is just not mature yet or if it's not getting much attention in the community. When I have time, I'll start digging into the Matter code itself and see if I can port my old custom curtain controller to the Matter protocol.
