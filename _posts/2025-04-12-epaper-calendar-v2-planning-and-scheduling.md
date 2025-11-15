---
layout: post
title: "E-Paper Calendar V2: Planning and Scheduling"
date: 2025-04-12 10:00:00 +0800
categories: [DIY, E-Paper calendar ver2]
tags: [planning, esp32, v2, checklist, devlog, risk-management]
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99%E6%9C%88%E6%9B%86-v2-%E8%A8%88%E5%8A%83%E6%8E%92%E7%A8%8B-75a7484c68c1) and **translated** by Gemini pro 2.5.

---

This post is still a summary of discussion points I organized with GPT. This time, however, I fed it a few of my previous articles, hoping the writing style wouldn't be *too* different.

This is the implementation plan I've organized for the second version of the e-paper calendar.

Unlike V1, where I just coded and fixed things as I went, this time I want to break the entire development process into manageable stages. I'll follow the sequence, verifying each part, tackling the highest-risk items first before moving on to integration and adjustments.

This post serves as my own development checklist and a way to look back on the process later.

### Project Goals (Brief)

The goals for this V2 are:
* Use an ESP32 + e-paper display to build a long-battery-life calendar device.
* No daily charging; should run for 1-2 months on a single battery.
* Automatically fetch future events from an ICS calendar link.
* Update the screen 1-2 times a day; otherwise, stay in Deep Sleep.
* Allow a non-technical user to configure Wi-Fi and calendar links without writing code.
* Maintain the V1 characteristic of "sitting quietly on the desk, not disturbing your life."

### Development Sequence Planning

To avoid getting halfway through the project only to find that the most critical feature doesn't work, I'm starting with the highest-risk parts first. For example: Can the ESP32 even parse an ICS file? Can the RTC reliably wake the device every day?

After ensuring the critical functions work, I'll move on to UI optimization, the setup flow, and layout.

### Development Checklist (In Implementation Order)

#### Phase 1: Basic Functionality Tests
* [ ] Light up the e-paper display (using the Waveshare example).
* [ ] Have the ESP32 fetch an ICS calendar link and parse the `SUMMARY` / `DTSTART` fields.
* [ ] Implement Wi-Fi setup (Device boots into AP mode, phone connects to configure).
* [ ] Implement Deep Sleep + RTC wakeup for daily scheduled refreshes.
* [ ] Test font rendering and support for Mandarin/Chinese characters.
* [ ] Use the V1 background image and complete an overlay of the V1 screen layout (text on image).

#### Phase 2: Expansion and Integration
* [ ] Read battery voltage via ADC voltage divider; prepare a "low battery" reminder.
* [ ] Test if the device can still refresh stably when the voltage is low.
* [ ] Switch to using `config.json` to control text size/position and other layout info.
* [ ] Fully integrate the background image + ICS data + font configuration to build the final screen.
* [ ] Store settings (Wi-Fi / ICS / Layout) in SPIFFS.
* [ ] Add a reset mechanism (e.g., long-press a button to clear settings).

#### Phase 3: Advanced Features (TBD, but possible)
* [ ] OTA firmware updates (uploading from the web page).
* [ ] Support for uploading custom background images and fonts.
* [ ] Move ICS parsing to a server → ESP32 just fetches a simplified JSON.

### Progress Update Method

After I complete each stage, I'll write a separate post documenting the implementation details and any pitfalls I encountered. If you're interested in this type of product, you can also use this checklist as a reference for your own development process.

Feel free to discuss or ask questions. I'll continue to update this series.