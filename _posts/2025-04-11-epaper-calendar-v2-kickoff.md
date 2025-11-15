---
layout: post
title: "E-Paper Calendar v2: Kick-off"
date: 2025-04-11 10:00:00 +0800
categories: [DIY, E-Paper calendar ver2]
tags: [planning, hardware, esp32, pcb, power-management, v2]
image: /assets/img/2025-04-11-epaper-calendar-v2-kickoff/0_PqG3cSi_m54PdHJ7.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99%E6%9C%88%E6%9B%86v2-%E5%95%9F%E5%8B%95-8c0cb86af590) and **translated** by Gemini pro 2.5.

---

Because I was too lazy, the following content is a Medium article I had GPT write directly after I discussed the initial design with it. I didn't really proofread it, just adjusted the layout a bit. Feel free to just skim it.

### V1 Implementation and Limitations

For the first version, I used a Raspberry Pi paired with a Waveshare e-paper module. I got the functionality working: it would automatically boot up once a day to update the screen, then shut down to save power. Calendar data was imported from an ICS link, and weather was scraped from Taiwan's weather bureau website.

But it had several obvious drawbacks:
* **High Cost** (Raspberry Pi + PiSugar battery module)
* **Battery Life was good,** a single charge could last two or three months
* **Settings were hard-coded,** making it impossible for a non-developer to use
* **The weather API broke,** and the weather could no longer be updated

These limitations pushed me to start planning the second version.

### V2 Design Goals

The focus of V2 is: **stability, ease of configuration, extensibility, and suitability for more people.** The specific goals include:
* **Switch to ESP32** to replace the Raspberry Pi, lowering cost and power consumption.
* Power with a **single 18650 Li-ion battery,** aiming for 1-3 months of battery life.
* Support **AP Mode** for setting up Wi-Fi, uploading ICS links, background images, etc.
* Use an **SD card and JSON config file** to achieve UI customization and layout configuration.
* Establish a **voltage monitoring and low-battery protection** mechanism.

### System Architecture

The overall system will be composed of the following modules:

* **ESP32-WROOM Module:** The main controller, responsible for network connection, data updates, and E-Ink control.
* **Waveshare 7.3" E-Ink Display:** SPI communication, supports displaying backgrounds and multiple fonts.
* **DS3231 RTC Module:** For accurate timekeeping and waking the device for updates.
* **SD Card Module:** Stores background images, fonts, ICS files, and config files.
* **TP4056 + 18650 Li-ion + 3.3V LDO:** A single-cell, low-power system.
* **GPIO + ADC Voltage Divider:** To monitor battery voltage and remind the user to charge.

### User Scenario: From Unboxing to Usage

I envision the user flow as follows:

1.  Get the device → Plug it in to power on.
2.  The device enters **Wi-Fi setup (AP) mode**.
3.  Connect a phone, which brings up a setup webpage → Enter Wi-Fi credentials, ICS link, and choose a background image.
4.  Save, and the device reboots → It now updates the screen at a set time every day.

All setting information will be stored in `config.json`, and the user can modify settings again later via the web backend. The core UX goal is: **"Usable without writing any code."**

### Is the ESP32 Enough? My Observations

Besides its low power consumption and low cost, the ESP32 has other advantages:

* **Flash space and SPIFFS** are sufficient to hold settings and a simple webpage.
* **Stable** network connectivity, JSON parsing, and SD card operations.
* Supports **OTA updates** and file uploads.
* **Rich peripheral resources,** an active community, and easy-to-source modules.

So far, all my project requirements can be met by the ESP32, with enough headroom left over for future features like sensors, language switching, MQTT, etc.

### Conclusion: The Next Steps Toward a Product

This ESP32 e-paper calendar is evolving from a "personal prototype" into a practical "device that can be offered to more people." My next goals are to:
1.  **Design a custom PCB** to replace the jumper wires.
2.  Add an **OTA firmware update** mechanism.
3.  Optimize the **setup webpage UI**.
4.  Consider implementing a **MAX17048 Fuel Gauge** for battery level display.

If you're also interested in this kind of device, or are working on a similar product, feel free to get in touch. Let's build more practical and stable IoT display systems together!