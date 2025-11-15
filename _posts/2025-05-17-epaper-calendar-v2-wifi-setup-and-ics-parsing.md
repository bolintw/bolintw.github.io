---
layout: post
title: "E-Paper Calendar V2: Wi-Fi Setup and ICS Parsing"
date: 2025-05-17 10:00:00 +0800
categories: [DIY, E-Paper calendar ver2]
tags: [esp32, v2, wifi, softap, ics, parsing, https, debugging]
---

This article is **migrated** from [Medium](https://medium.com/@k2345777/%E9%9B%BB%E5%AD%90%E7%B4%99%E6%9C%88%E6%9B%86v2-%E8%A8%AD%E5%AE%9Awifi%E5%8F%8A%E8%A7%A3%E6%9E%90ics-0283118f0437) and **translated** by Gemini pro 2.5.

---

*(The following article was produced by Gemini 2.5 Pro. I've found that the error rate in code from GPT is so high that I've started to migrate to Gemini. However, I'm not quite used to the phrasing Gemini uses for blogs, so it requires a bit more editing.)*

Hello, fellow Makers! It's been a while since I last shared an update on my e-paper Google Calendar project. I've fallen into quite a few pits in the meantime, but I've also gained some experience. For this V2 version, I've focused on implementing and optimizing two core functions: allowing users to **conveniently set up Wi-Fi** and **more reliably parsing ICS calendar files** from Google Calendar.

If you recall, the goal of this project is to build an ultra-low-power e-paper calendar that can run for months on a single charge. It automatically fetches and displays to-do items from a specified ICS URL every day. Therefore, a critical user experience problem is how to let a user easily set up the network when they first get the product, or when they change their home Wi-Fi.

### The Wi-Fi Setup Challenge: User Experience First

For an IoT device with no keyboard or screen, how do you get it to connect to your home Wi-Fi? There are a few common methods:

1.  **BLE (Bluetooth Low Energy) Provisioning:** Use a mobile app to send Wi-Fi credentials to the device via Bluetooth. **Pro:** Smooth experience. **Con:** Requires developing a dedicated mobile app for both iOS and Android, which is a high maintenance cost.
2.  **SmartConfig (e.g., ESP-TOUCH):** A mobile app encodes and broadcasts the Wi-Fi info; the device captures it in listening mode. **Pro:** Also relatively simple. **Con:** Success rate can be affected by the network environment or phone OS.
3.  **SoftAP (Software Access Point) Provisioning:** The ESP32 turns itself into a small Wi-Fi hotspot. The user connects their phone or computer to this hotspot, then opens a specific webpage in their browser to enter their home Wi-Fi SSID and password.

Considering I didn't want the extra complexity and maintenance cost of developing an app, and I wanted users on any platform to be able to set it up easily, I ultimately chose the **SoftAP provisioning method**.

### My Choice and Implementation: SoftAP

The SoftAP flow looks roughly like this:

1.  **Check Condition:** When the ESP32 boots, if it has no Wi-Fi credentials saved in NVS (Non-Volatile Storage), or if it fails to connect with saved credentials, it automatically enters SoftAP mode. A user could also trigger this manually (e.g., by long-pressing a button).
2.  **ESP32 Becomes a Hotspot:** The ESP32 creates an open Wi-Fi hotspot, for example, named `ESP-Calendar-Setup`.
3.  **User Connects:** The user searches for Wi-Fi on their phone or computer and connects to the `ESP-Calendar-Setup` hotspot.
4.  **Open Setup Page:** Once connected, a mini HTTP server running on the ESP32 serves a configuration page. The user enters the ESP32's IP address (usually `192.168.4.1`) into their browser to see the setup screen.
5.  **Enter and Submit:** On the webpage, the user enters their home Wi-Fi SSID and password, then submits.
6.  **Save and Connect:** The ESP32 receives the info, saves it to NVS, and then attempts to connect to Wi-Fi with the new credentials. On success, it shuts down the SoftAP mode and proceeds to the normal calendar workflow.

```c
// Pseudo-code: Core logic for handling a POST request in SoftAP mode (illustrative)
esp_err_t connect_post_handler(httpd_req_t *req) {
    char buf[256];
    // ... (Receive POST data into buf) ...
    char ssid[33] = {0};
    char password[65] = {0};
    // ... (Parse ssid and password from buf, watch out for URL decode) ...
    if (strlen(ssid) > 0) {
        // Save ssid and password to NVS
        save_wifi_credentials_to_nvs(ssid, password);
        
        // Notify the main process to start connecting with new credentials
        trigger_wifi_connect_attempt();
        
        httpd_resp_send(req, "Wi-Fi settings received!", HTTPD_RESP_USE_STRLEN);
    } else {
        httpd_resp_send_err(req, HTTPD_400_BAD_REQUEST, "SSID cannot be empty");
    }
    return ESP_OK;
}
```

This method, while requiring the user to manually switch their Wi-Fi, has the benefit of working on almost any device with a browser. No extra app installation is needed, striking a good balance between development cost and user convenience.

### The Pits of ICS Parsing

With Wi-Fi setup handled, the next step was fetching the ICS file from Google Calendar and parsing the to-do items. I naively thought this would be simple—after all, ICS is a standard format, right?
As it turns out, it was one pitfall after another.

### Problems Encountered and Solutions

#### 1. HTTPS Connection and Certificates
Google Calendar's ICS URL is HTTPS. For an ESP32 to make an HTTPS connection, it needs to handle TLS/SSL encryption and server certificate validation. My first attempt to fetch it with `esp_http_client` immediately resulted in errors like `No server verification option set` or `SSL handshake failed`.

**Solution:**
You need to enable the CA certificate bundle to verify the server in the `esp_http_client_config_t`. This requires setting `.crt_bundle_attach = esp_crt_bundle_attach;` in the code.
At the same time, you **must** enable `Component config` -> `mbedTLS` -> `Certificate Bundle` -> `Enable ESP x509 Certificate Bundle` and `Add common CA certificates` in `menuconfig`.

#### 2. Time Synchronization
TLS certificates have expiration dates. If the ESP32's local time is wrong (e.g., it just booted and thinks it's 1970), it will cause the certificate validation to fail. Therefore, **before** making any HTTPS connection, you must first sync the network time via SNTP and set the local timezone (e.g., `setenv("TZ", "CST-8", 1); tzset();`).

#### 3. Parsing `DTSTART` (Date-Time) Strings
The event start time (`DTSTART`) in ICS files comes in multiple formats, such as `DTSTART:20231001T120000Z` (UTC time), `DTSTART;VALUE=DATE:20231001` (all-day event), or `DTSTART:20231001T120000` (local or floating time).

I initially tried to use the standard C function `strptime` to parse these, but I found that in the ESP-IDF (Newlib C library) environment, it is **very unstable** for formats that include a `T` (datetime separator) or `Z` (UTC identifier) directly in the format string. It often failed to parse.

**Solution:**
After many attempts and discussions with the community, I found that relying on `strptime` to parse these mixed-format strings is unreliable on the ESP32. In the end, I gave up the struggle and decided to **manually parse the `DTSTART` string.**

I wrote my own `manual_parse_dtstart` function that uses `sscanf` or direct character comparison to extract the year, month, day, hour, minute, and second from the `DTSTART` value. It also needs to check for a `Z` at the end of the string to determine if it's UTC time.

```c
// Illustrative: Core logic for manually parsing DTSTART
static int manual_parse_dtstart(const char* dt_str_in, struct tm *t_out, bool *is_utc_out) {
    // ... (First, clean up whitespace around dt_str_in) ...
    char* cleaned_dt_str = ...;
    
    int year, month, day, hour = 0, min = 0, sec = 0;
    int consumed_chars = 0;
    
    if (sscanf(cleaned_dt_str, "%4d%2d%2dT%2d%2d%2d%n",
               &year, &month, &day, &hour, &min, &sec, &consumed_chars) == 6) {
        // Successfully parsed YYYYMMDDTHHMMSS
        if (is_utc_out && cleaned_dt_str[consumed_chars] == 'Z') {
            *is_utc_out = true;
        }
    } else if (sscanf(cleaned_dt_str, "%4d%2d%2d%n", &year, &month, &day, &consumed_chars) == 3) {
        // Successfully parsed YYYYMMDD (all-day event)
        // ...
    } else {
        return 0; // Parse failed
    }
    
    // ... (Fill the t_out struct) ...
    return 1; // Parse success
}
```

#### 4. Annoying Timezone Conversions
After parsing the string into a `struct tm`, it needs to be converted to a `time_t` (UTC timestamp) for easy comparison and sorting. The problem is, if the `DTSTART` is UTC (with a `Z`), the `mktime` function, by default, treats the `struct tm` as **local time** for the conversion.

**Solution:**
First, after syncing time via SNTP, set the ESP32's local timezone, e.g., `setenv("TZ", "CST-8", 1); tzset();`.

When a UTC `DTSTART` is parsed, **before** calling `mktime`, temporarily set the `TZ` environment variable to `"UTC0"` (e.g., `setenv("TZ", "UTC0", 1); tzset();`). This forces `mktime` to treat the `struct tm` components as UTC, converting it to the correct UTC `time_t`. After the conversion, remember to restore the `TZ` setting.

If the `DTSTART` does not have a `Z`, just call `mktime` using the normal local timezone setting.

### Current Progress and Next Steps

After all this trouble, the basic functionality of the V2 version is now working:
* Users can configure Wi-Fi credentials via SoftAP mode.
* The device can successfully download calendar data from a specified ICS URL via HTTPS.
* Through manual parsing of `DTSTART`, it can handle common date-time formats (including UTC, local time, and all-day events).
* It correctly performs timezone conversions and can filter/sort future events from the ICS file, preparing them for display.

Although the Wi-Fi setup and basic ICS parsing are taking shape, the real challenge still lies in perfectly integrating these functions with the e-paper display to create a truly usable core product.

The next focus will be:
1.  **Integrating all modules:** Ensure the flow of Wi-Fi setup, time sync, ICS fetching, event parsing, and displaying to the screen works smoothly from start to finish.
2.  **Building the Core MVP (Minimum Viable Product):** The goal is to first complete a version with the most essential features, bringing this e-paper calendar to life so it can reliably display the latest schedule.

### Conclusion

DIY electronics projects are just like this—full of challenges, but also full of fun. Solving each problem is a chance to learn and grow. The E-Paper Calendar V2 journey is still ongoing. I will continue to document and share my progress and the "pits" I fall into. Thanks for reading, and feel free to leave a comment to share your thoughts!