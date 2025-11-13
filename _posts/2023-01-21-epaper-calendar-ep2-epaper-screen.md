---
title: "E-Paper Calendar Ep.2 — The E-Paper Display"
date: 2023-01-21 10:00:00 +0800
categories: [DIY, E-Paper calendar ver1]
tags: [diy, iot, epaper]
image: /assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_4YaUCUPm8-48xFqim9DCtw.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99google%E6%9C%88%E6%9B%86ep-2-%E9%9B%BB%E5%AD%90%E7%B4%99%E8%9E%A2%E5%B9%95-4ac74c2337cb) and **translated** by Gemini pro 2.5.

---

### Driving the E-Paper
The e-paper display finally arrived! I've never played with this kind of screen before, but thankfully, Waveshare's documentation is quite complete, and they provide demo code for several platforms.

![E-Paper Screen Specifications](/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_VOu-6ziAESkj1M8XDU4XOw.webp)

![Official Waveshare Documentation](/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_jK0rMygXMtnZOsQK5pl16g.webp)

When I learned this screen uses SPI communication, I originally wanted to use an STM32 to run the demo. But after not being able to find my STM32 test board, I thought I'd use an ESP32 S3 (since I'll need Wi-Fi functionality later anyway). However, the official ESP32 demo code they provide doesn't cover this specific screen model.

Not wanting to waste too much time just getting the demo to work, I decided to just go with a Raspberry Pi 4.

### Raspberry Pi 4 Environment Setup

There are a ton of tutorials for this online, so I won't go into detail. The process is just: install Pi OS, set up SSH, and control the RPi4 via SSH after it boots.

Next up was wiring. The SPI adapter module that Waveshare included can be installed directly onto the Raspberry Pi's header pins. You just press it right in. Then, connect the screen's flat cable. **You have to pay attention to the direction**—I installed it backward at first, and the screen had no reaction at all. I almost thought I had received a defective product.

![Waveshare module installed on the Raspberry Pi](/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_4YaUCUPm8-48xFqim9DCtw.webp)

### First Time Lighting Up the Screen

After that, I just followed the official documentation: enable the SPI peripheral, install the Python environment, download the demo code, and run it.

![First screen refresh](/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_jUd_xMzg0_OEPo-O7a0MMg.webp)

[![Demo video](/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/thumbnail.png)](https://youtu.be/qqOuStTM99s?si=n59ECyucnQ10M45T)

As you can see from the video, a full screen refresh takes over ten seconds, which is pretty close to the 16 seconds stated in the official specs. The display quality is actually quite beautiful, and the 800x480 resolution should be sufficient.

I took a brief look at the demo code; it's simple and easy to understand. Modifying it later shouldn't be difficult.

**The most fortunate part was seeing Mandarin characters displayed.** This was the bottleneck I was most worried about, and it seems to be half-solved already.

The next step will be to start controlling the screen to display my own specified images and text. After that, I'll start working on a rough layout, and then move on to the next research phase: how to fetch data from Google Calendar.
