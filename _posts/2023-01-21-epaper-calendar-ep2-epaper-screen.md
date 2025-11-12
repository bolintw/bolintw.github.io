---
title: "E-Paper Calendar Ep.2 — The E-Paper Display"
date: 2023-01-21 10:00:00 +0800
categories: [DIY, E-Paper calendar ver1]
tags: [diy, iot, epaper]
image: /assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_4YaUCUPm8-48xFqim9DCtw.webp
---

### Driving the E-Paper
The e-paper display finally arrived! I've never played with this kind of screen before, but thankfully, Waveshare's documentation is quite complete, and they provide demo code for several platforms.

<p class="text-center">
  <img src="/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_VOu-6ziAESkj1M8XDU4XOw.webp" alt="E-Paper Screen Specifications" width="600">
  <em>E-Paper Screen Specifications</em>
</p>

<p class="text-center">
  <img src="/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_jK0rMygXMtnZOsQK5pl16g.webp" alt="Official Waveshare Documentation" width="600">
  <em>Official Waveshare Documentation</em>
</p>

When I learned this screen uses SPI communication, I originally wanted to use an STM32 to run the demo. But after not being able to find my STM32 test board, I thought I'd use an ESP32 S3 (since I'll need Wi-Fi functionality later anyway). However, the official ESP32 demo code they provide doesn't cover this specific screen model.

Not wanting to waste too much time just getting the demo to work, I decided to just go with a Raspberry Pi 4.

### Raspberry Pi 4 Environment Setup

There are a ton of tutorials for this online, so I won't go into detail. The process is just: install Pi OS, set up SSH, and control the RPi4 via SSH after it boots.

Next up was wiring. The SPI adapter module that Waveshare included can be installed directly onto the Raspberry Pi's header pins. You just press it right in. Then, connect the screen's flat cable. **You have to pay attention to the direction**—I installed it backward at first, and the screen had no reaction at all. I almost thought I had received a defective product.

<p class="text-center">
  <img src="/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_4YaUCUPm8-48xFqim9DCtw.webp" alt="Waveshare module installed on the Raspberry Pi" width="600">
  <em>Waveshare module installed on the Raspberry Pi</em>
</p>

### First Time Lighting Up the Screen

After that, I just followed the official documentation: enable the SPI peripheral, install the Python environment, download the demo code, and run it.

<p class="text-center">
  <img src="/assets/img/2023-01-21-epaper-calendar-ep2-epaper-screen/1_jUd_xMzg0_OEPo-O7a0MMg.webp" alt="First screen refresh" width="600">
</p>

<div class="ratio ratio-16x9 mx-auto" style="max-width: 700px;">
  <iframe 
    src="https://www.youtube.com/embed/qqOuStTM99s?si=JvNJdTyUzn52CRWI" 
    title="YouTube video player" 
    frameborder="0" 
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
    allowfullscreen>
  </iframe>
</div>

As you can see from the video, a full screen refresh takes over ten seconds, which is pretty close to the 16 seconds stated in the official specs. The display quality is actually quite beautiful, and the 800x480 resolution should be sufficient.

I took a brief look at the demo code; it's simple and easy to understand. Modifying it later shouldn't be difficult.

**The most fortunate part was seeing Mandarin characters displayed.** This was the bottleneck I was most worried about, and it seems to be half-solved already.

The next step will be to start controlling the screen to display my own specified images and text. After that, I'll start working on a rough layout, and then move on to the next research phase: how to fetch data from Google Calendar.
