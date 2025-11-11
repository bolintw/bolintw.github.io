---
title: "E-paper calendar ep1: kicking off the project"
date: 2023-01-13 10:00:00 +0800
categories: [DIY, E-Paper calendar ver1]
tags: [diy, iot, epaper]
image: /assets/img/2023-01-13-epaper-calendar-ep1-kicking-off-the-project/1_WZ11xMxszC_aFozGy8DY_w.webp
---

I've decided to start a new project: building a physical Google Calendar display using an e-paper screen.

### Why a Physical Google Calendar?

You might be wondering why anyone would need this. For me, it comes down to a few key points:

1.  **E-Paper is Easy on the Eyes:** E-paper screens don't emit their own light; they rely on ambient light, just like real paper. This makes them incredibly comfortable to look at.
2.  **Ultra-Low Power:** E-paper only consumes power when the image is *changing*. A static display uses zero energy.
3.  **Passive Reminders:** A calendar is meant to be a constant reminder of your schedule. It should be placed somewhere you can see it often. If I have to pull out my phone just to check my schedule, the calendar has already failed its primary purpose.

### The Initial (Failed) Shortcut

When I first started gathering info, I found a crowdfunding project on ZecZec for a very similar product. The price was even lower than my own estimated bill of materials (BOM), so I decisively gave up on the DIY plan and just placed an order.

However, even before I received the product, I saw early backers reporting a terrible user experience. I already knew the screen refresh would be slow—e-paper isn't designed for high-frequency updates. But then I saw someone say you had to **pull out your phone to force the screen to update**.

Doesn't that defeat the entire purpose of the product?

I got a refund and went back to the drawing board. It was time to build it myself.

After some research, I found a couple of projects that served as great inspiration:
* [SpeedyG0nz's MagInkCal (Raspberry Pi)](https://github.com/speedyg0nz/MagInkCal)
* [An E-Ink Family Calendar Using ESP32 on Instructables](https://www.instructables.com/E-Ink-Family-Calendar-Using-ESP32/)

### Defining the Project Goals

First, I need to set clear goals so I can review them later.

* The device **must** be able to connect to Google Calendar on its own to fetch the latest event data. A once-daily update should be sufficient. The main goal is to see future plans; last-minute additions aren't the focus here.
* It can be either a desktop or wall-mounted device. I'll figure this out at the end, as it relates to whether I'll use battery power or a wired adapter.

The most critical component is the e-paper screen. I saw the first reference project used a Waveshare dual-color screen with good results. I've used their 7" touch screen before for a custom car navigation project, and the quality was decent. Plus, they offer a wide variety of options and resources.

A calendar needs to display a lot of information, so the screen size should be 7 inches or larger to be comfortable to read. Considering this is my first time using an e-paper display and the project *could* fail, I decided not to buy a massive 12" screen just yet.

I settled on a **7.5" four-color (Black, White, Yellow, Red) e-paper display**. The price was more acceptable, and at this size, I might have to consider a "daily" calendar format, as a "monthly" view might be too cluttered to read.

### Identifying Potential Bottlenecks

Next, I listed out the challenges I can anticipate:

1.  **Driving the E-Paper Display:** This shouldn't be a huge problem, but since I've never used one, I don't know how deep the water is.
2.  **Fetching Google Calendar Data:** There should be plenty of resources online for this.
3.  **Efficient Display Rendering:** I'm thinking I'll need to divide the screen into several "regions" to manage and update them efficiently.
4.  **Displaying Chinese Characters:** This is the biggest bottleneck I can think of right now, and it has the potential to kill the entire project.

...Unfortunately, I only thought of this *after* I had already bought the screen. Oh well, I'm already in too deep.

I'll wait for the screen to arrive and start experimenting. I'll post the next update then.
