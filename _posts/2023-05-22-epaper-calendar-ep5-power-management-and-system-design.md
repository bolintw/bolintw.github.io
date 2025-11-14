---
layout: post
title: "E-Paper Calendar Ep.5 — Power Management and System Design"
date: 2023-05-22 10:00:00 +0800
categories: [DIY, E-Paper calendar ver1]
tags: [power-management, battery, pisugar, raspberry-pi, rtc, system-design, diy, iot, epaper]
image: /assets/img/2023-05-22-epaper-calendar-ep5-power-management-and-system-design/1_Ysc7J9IVOh6i91Iol_KHfQ.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99google%E6%9C%88%E6%9B%86ep-5-%E9%9B%BB%E6%BA%90%E7%AE%A1%E7%90%86%E5%8F%8A%E7%B3%BB%E7%B5%B1%E8%A8%AD%E8%A8%88-8335546174d4) and **translated** by Gemini pro 2.5.

---

The project is finally nearing its end. I spent some time on the final system integration and adjustments, mainly because I wanted to be able to power it with a battery and fit everything into a picture frame. That would be the perfect setup.

If I wanted to handle the battery myself, I could, but I'd need to find a separate charging module. On top of that, a battery-powered project requires thinking about power consumption. The ideal scenario is to update the display once or twice a day and keep the Raspberry Pi in a powered-off state the rest of the time.

But to achieve a "shutdown and wake-up" cycle, I would need *another* external RTC (Real-Time Clock) module. Trying to wire up all these separate modules (battery, charger, RTC) and fit them neatly into a frame... just thinking about it gives me a headache.

![PiSugar Module](/assets/img/2023-05-22-epaper-calendar-ep5-power-management-and-system-design/0_kafx14PKktK3YzXv.webp)

For DIY projects, the most convenient path is to find a pre-integrated module. If I had to reinvent the wheel and assemble everything myself, it wouldn't just slow down the project; it would make the whole process tedious.

So, I found this module: **PiSugar**. It’s a ready-made solution that has a battery, a charging module, *and* an RTC, plus a simple user interface for setting wake-up times. It perfectly meets all my needs.

> [https://www.pisugar.com/](https://www.pisugar.com/)

With that solved, I designed the final system flow:

1.  **On system boot**, set the *next* wake-up time. (This is done by directly modifying the PiSugar config file. It's important to remember to `restart` the PiSugar server afterward for the settings to take effect).
2.  **Run the update script.** First, check the network connection. Then, update the date and weather, fetch and sort all the iCal events, and get the current battery level to display on the e-paper screen.
3.  **After everything is done**, set the system to **shut down in one minute**.
    (This one-minute buffer is crucial. I initially used `sudo shutdown now` and ran into a bug. I couldn't even SSH in to fix it before the system powered off on me.)

The system is basically just that simple. I've currently set it to update twice a day, at 6 AM and 6 PM. Now I just need to monitor the battery life. If it can last for two weeks, or even a month, on a single charge, I'll consider it very usable. If not, I'll have to consider underclocking the system.
