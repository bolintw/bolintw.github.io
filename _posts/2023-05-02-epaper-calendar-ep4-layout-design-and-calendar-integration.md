---
layout: post
title: "E-Paper Calendar Ep.4 — Layout Design and Calendar Integration"
date: 2023-05-02 10:00:00 +0800
categories: [DIY, E-Paper calendar ver1]
tags: [chatgpt, ics, python, google-calendar, raspberry-pi, diy, iot, epaper]
image: /assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/0_f0HCnLfcgkcPmxX8.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99google%E6%9C%88%E6%9B%86ep-4-%E8%A8%AD%E8%A8%88%E7%95%AB%E9%9D%A2%E5%8F%8A%E8%A1%8C%E4%BA%8B%E6%9B%86%E6%95%B4%E5%90%88-a583a2f3c5bf) and **translated** by Gemini pro 2.5.

---

Well, I've finally made it to *that* part: designing the layout. Creating an aesthetically pleasing calendar might just be the biggest bottleneck of this entire project.

I figured, why not see if ChatGPT, the pinnacle of new-generation human technology, could spark some new ideas?

![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_4fwKSwejCzf4WG_OeK9cAQ.webp)
![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_dazaY8HlKSVmE1sX002IBw.webp)

Maybe the prompt was the problem. Everyone says the tool isn't the issue; it's how you use it.

![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_Iq4q7_LsU20FYHZmce8YYw.webp)
![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_kmwNfOjUMh_fe8heKCb6ng.webp)

I have no idea what GPT's definition of "beautiful" is, and I'm not going to dig into it. But could it at least get the format right?

![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_VCshEBPAw6ljjBoW0fjMUA.webp)
![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_J-mNutELWyHLWKiXfx9VDg.webp)

OK, it's a version that is *technically* usable. But I just can't bring myself to use it.

This is when the professional network you've built up comes into play. I decided to pull in a favor from a colleague. Our designer should have no problem with this. Otherwise, this project was about to be declared dead...

![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_unr_kRyCUuqZFGMOOq2ThQ.webp)

All I can say is, everyone has their specialty. At least the project can continue.

---

### Integrating the Calendars

Next up is the calendar integration. In the previous post, I used the Google API to get Google Calendar data. Later, because some of my calendars are in iCloud, I also used the `pyiCloud` library, logged into my Apple account, and successfully pulled that data too.

But then a colleague wanted a set for themselves (I've got orders before it's even finished!). I can't possibly expect every user to apply for Google Cloud credentials, get a token, and go through authentication just to use it. After some simple research, I found there is indeed a much more convenient way.

> iCalendar (Internet Calendar, often abbreviated as iCal) is a universal calendar data exchange format used to represent and share calendar events, schedules, and to-do lists. iCalendar is widely used in PIMs, scheduling systems, and calendar applications to facilitate data exchange between different platforms and services. The iCalendar data structure is based on a text file, typically with an .ics extension. (The above is from GPT).

Simply put, both Google and Apple calendars use this format. This means I can get a corresponding `.ics` URL, and then use a library to parse it to get the calendar data. **Using this format doesn't require any login or account authentication**; you just need to manage your private `.ics` address.

![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_EQWTo8yhzp0sbTk4KoDnkg.webp)

You can find this `.ics` URL in the Google Calendar settings page. After that, I followed the official documentation for the [Python `ics` library](https://pypi.org/project/ics/) and easily integrated all my scattered calendars. There were some formatting issues with the iCloud parsing, but I handled those with some extra processing and got it done.

![](/assets/img/2023-05-02-epaper-calendar-ep4-layout-design-and-calendar-integration/1_oP4Ec0_P2tPVO8-a6gDozA.webp)

Next up is the circuit integration and the problem of how to fit it all into a picture frame. It looks like this project will be wrapping up soon.
