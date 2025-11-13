---
layout: post
title: "E-Paper Calendar Ep.3 — Fetching Calendar Data and Displaying It"
date: 2023-04-17 10:00:00 +0800
categories: [DIY, E-Paper calendar ver1]
tags: [google-calendar, python, raspberry-pi, diy, iot, epaper]
image: /assets/img/2023-04-17-epaper-calendar-ep3-fetching-calendar-data-and-displaying/1_XJ2dqPE1cr7NJoaUnEEkMA.webp
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99google%E6%9C%88%E6%9B%86ep-3-%E5%8F%96%E5%BE%97%E6%9C%88%E6%9B%86%E8%B3%87%E6%96%99%E5%8F%8A%E9%9B%BB%E5%AD%90%E7%B4%99%E9%A1%AF%E7%A4%BA-f1a5a67a360f) and **translated** by Gemini pro 2.5.

---

Actually, I finished testing this part a long time ago, but I got busy and forgot to document it. I'm adding these notes here for my own future reference.

### Fetching Google Calendar Data

First up was fetching data from Google Calendar. I originally expected this to be a huge hassle, but by following the steps in Google's official API documentation, I got it working right away.

> [Python Quickstart Google Calendar Google Developers](https://developers.google.com/workspace/calendar/api/quickstart/python?hl=zh-tw&source=post_page-----f1a5a67a360f---------------------------------------)

Google provides support for many languages. I chose Python because the Waveshare e-paper demo for Raspberry Pi also uses Python, and as I'll explain, this turned out to be very convenient.

Google's documentation already clearly explains the credential and environment setup. I'll just pull out and highlight the key code snippet for fetching calendar events.

```python
# Call the Calendar API
now = datetime.datetime.utcnow().isoformat() + 'Z'  # 'Z' indicates UTC time
print('Getting the upcoming 10 events')
events_result = service.events().list(calendarId='primary', timeMin=now,
                                      maxResults=10, singleEvents=True,
                                      orderBy='startTime').execute()
events = events_result.get('items', [])

if not events:
    print('No upcoming events found.')
    return

# Prints the start and name of the next 10 events
for event in events:
    start = event['start'].get('dateTime', event['start'].get('date'))
    print(start, event['summary'])
```

### Displaying Data on the E-Paper

Next up was displaying the data on the e-paper screen. The Waveshare demo code is quite complete. It uses the Python Pillow library (PIL) to convert text into an image, making it incredibly easy to draw any picture and text you want onto the display.

```python
#initialize e-paper process
epd = epd7in3g.EPD()
epd.init()
epd.Clear()

#setting some font size for drawing words
font24 = ImageFont.truetype(os.path.join(picdir, 'Font.ttc'), 24)
font18 = ImageFont.truetype(os.path.join(picdir, 'Font.ttc'), 18)
font40 = ImageFont.truetype(os.path.join(picdir, 'Font.ttc'), 40)

#make a new image
Himage = Image.new('RGB', (epd.width, epd.height), epd.WHITE)  # 255: clear the frame
draw = ImageDraw.Draw(Himage)

#draw some words on the image
draw.text((5, 0), 'hello world', font = font18, fill = epd.RED)
draw.text((5, 20), '7.3inch e-Paper', font = font24, fill = epd.YELLOW)
draw.text((5, 45), u'微雪电子', font = font40, fill = epd.BLACK)
draw.text((5, 85), u'微雪电子', font = font40, fill = epd.YELLOW)
draw.text((5, 125), u'微雪电子', font = font40, fill = epd.RED)

#send the image to e-paper display
epd.display(epd.getbuffer(Himage))
```

The code above is the most important part of the demo. The next step is simply to combine the two: change the part that's being drawn to use the data fetched from Google Calendar.

```python
#initialize e-paper process
epd = epd7in3g.EPD()
epd.init()
epd.Clear()

#setting some font size for drawing words
font24 = ImageFont.truetype(os.path.join(picdir, 'Font.ttc'), 24)
font18 = ImageFont.truetype(os.path.join(picdir, 'Font.ttc'), 18)
font40 = ImageFont.truetype(os.path.join(picdir, 'Font.ttc'), 40)

# Call the Calendar API
service = build('calendar', 'v3', credentials=creds)

now = datetime.datetime.utcnow().isoformat() + 'Z'  # 'Z' indicates UTC time
events_result = service.events().list(calendarId='primary', timeMin=now,
                                      maxResults=10, singleEvents=True,
                                      orderBy='startTime').execute()
events = events_result.get('items', [])

if not events:
    print('No upcoming events found.')
    return

# Prints the start and name of the next 10 events
for event in events:
    start = event['start'].get('dateTime', event['start'].get('date'))
    print(start, event['summary'])
        
# Drawing on the image
logging.info("1.Drawing on the image...")
Himage = Image.new('RGB', (epd.width, epd.height), epd.WHITE)  # 255: clear the frame
draw = ImageDraw.Draw(Himage)
line = 0
for event in events:
    data = event['start'].get('dateTime', event['start'].get('date'))+' '+event['summary']
    draw.text((5, line), data, font = font24, fill = epd.BLACK)
    line+=18 #next line

#push the image to e-paper display
epd.display(epd.getbuffer(Himage))
```

![Final result showing calendar events on the e-paper](/assets/img/2023-04-17-epaper-calendar-ep3-fetching-calendar-data-and-displaying/1_DeDCYkZX_10LU0osXASY-Q.webp)

And just like that, the main functionality of the project is complete. It was much smoother than I expected.

### Lingering Issues

I've noted a few small problems so far:

1. Events synced from iCloud to Google Calendar are not being fetched by the Google API.

2. Due to the screen resolution, Manarin fonts become blurry if the font size is too small.

The next step is the overall system design, including the update frequency and the screen layout... (This will probably be the most difficult and time-consuming part).
