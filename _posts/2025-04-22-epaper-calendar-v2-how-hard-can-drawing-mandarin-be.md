---
layout: post
title: "E-Paper Calendar V2: How Hard Can Drawing Mandarin Be?"
date: 2025-04-22 10:00:00 +0800
categories: [DIY, E-Paper calendar ver2]
tags: [esp32, v2, fonts, debugging, freetype, ttf, tinyttf, cjk]
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99%E6%9C%88%E6%9B%86-v2-%E7%95%AB%E5%80%8B%E4%B8%AD%E6%96%87%E6%9C%89%E5%A4%9A%E9%9B%A3-5a6061b85041) and **translated** by Gemini pro 2.5.

---

This post was also produced by GPT.

Back in the V1 era of the e-paper calendar, I used a Raspberry Pi with Python to develop the display interface. Drawing Mandarin characters (Hanzi) was practically a non-issue. Libraries for text processing and font loading were everywhere—Pillow, OpenCV, Pango... you name it. I could just grab a Google Font or a Noto font, and it would render Mandarin on the screen instantly.

But for V2, I decided to ditch the Raspberry Pi and switch to an ESP32-S3 to pursue lower power consumption and cost.

At first, I thought it would be simple: "I just need one font, right? Just load the TTF and draw some Mandarin text." How hard could it be?

**The result: it blew up on compile.**

I converted a font file (Noto Sans TC Regular) into a `.c` array using `xxd -i` and put it in my project. I planned to use the Tiny TTF library to display a line of Mandarin text. The compiler immediately threw an error:

> .dram0.bss overflowed by 259240 bytes

The ESP32's internal RAM simply cannot hold this much font data.

### What *Is* a Computer Font?

To solve this, I first had to go back and research the nature of fonts.

The `.ttf` (TrueType Font) or `.otf` (OpenType Font) files we commonly use aren't simple image files. They are complex package formats containing vector descriptions, bitmap hints, character-to-glyph mapping, and other data.

When you zoom in or out on a font on your computer, what's actually happening is a system called a **text rendering engine** (like Freetype) is interpreting the vector outlines, calculating the corresponding pixels, and even automatically adjusting kerning based on the screen DPI.

These processes require enormous computational resources and memory. A Raspberry Pi can handle it easily, but an ESP32 is clearly not in the same league.

### Two Solutions: Space for Time, or Flexibility for Budget?

Back to our goal: drawing Traditional Chinese characters on an ESP32.

Considering the e-paper resolution is 800x600 and the screen only updates once or twice a day, response speed isn't the bottleneck. The real question is: **"How should the font file be handled?"**

After some discussion and experimentation, I sorted out two viable solutions:

#### Solution 1: Pre-render to Bitmap Font
Convert all *possible* Chinese characters, in a fixed font and size, into bitmap images or a binary array. For example, if each character takes up 32x32 pixels, you can just "paste" these images directly onto the screen.

* **Pros:**
    * Easy memory control.
    * Low computational load.
* **Cons:**
    * If the font or size needs to change, the entire dataset must be re-generated.
    * Not suitable for supporting dynamic or user-defined text.

#### Solution 2: Use a Text Engine + TTF Subset
This method uses a "subsetted" TTF (containing only the specified characters) and dynamically *rasterizes* it into a bitmap on the device. **Tiny TTF** is a relatively lightweight implementation of this.

* **Pros:**
    * High flexibility for maintenance.
    * Can change fonts and styles.
    * Can automatically expand the font subset as needed.
* **Cons:**
    * More complex initial development.
    * Requires the extra overhead of a rendering engine.

### I Chose Solution 2

Even though the initial compile failures and log-checking were frustrating, I ultimately decided to go the dynamic rendering route.

The reason is simple: **extensibility and maintainability are key** for this product to have a long-term future.

If I had to re-generate the entire font map every time I added a new feature, I would never be able to adapt to user needs quickly. In contrast, using dynamic rendering solves the need for font style switching and custom character support all at once. It's more painful to develop initially, but it's the right direction.

Drawing Mandarin is a trivial task for a computer. But doing it on a chip that costs less than a cup of bubble tea? That's not so simple.

In the next post, I'll continue to document how the ESP32 renders Traditional Chinese characters with minimal resources and sends the result back to the host via UART as a screen preview. Stay tuned.