# בניין יפה נוף 25 — Digital Signage Display

A full-screen building information display designed for a lobby TV. Built as a single static HTML file — no server, no build tools, no dependencies.

**Live display:** https://eugenekrapivin.github.io/yafeh-nof-25/

| Landscape | Portrait |
|---|---|
| ![Landscape](https://raw.githubusercontent.com/EugeneKrapivin/yafeh-nof-25/main/.github/screenshot.png) | ![Portrait](https://raw.githubusercontent.com/EugeneKrapivin/yafeh-nof-25/main/.github/screenshot-portrait.png) |

---

## What it shows

| Section | Source | Refresh |
|---|---|---|
| Clock & Hebrew/Gregorian date | Browser | Every second |
| Weather: current + 8-hour forecast + 4-day forecast | [open-meteo.com](https://open-meteo.com) | Every 5 min |
| Building announcements | Google Sheets | Every 5 min |
| Upcoming events | Google Sheets | Every 5 min |
| Birthday banner + calendar chips | Google Sheets | Every 5 min |
| Monthly calendar with holidays & Shabbat times | [Hebcal API](https://www.hebcal.com) | Every 5 min |
| Scrolling news ticker | Ynet RSS via rss2json | Every 5 min |
| Red alert (Pikud HaOref) | [oref.org.il](https://www.oref.org.il) | Every 3 sec |
| Background music | Local MP3 | Loops continuously |

---

## Red Alert (Pikud HaOref)

The display polls `https://www.oref.org.il/warningMessages/alert/Alerts.json` every 3 seconds for real-time missile alerts. When "חדרה מזרח" appears in the response, a red pulsing banner is shown:

> 🚨 צבע אדום! יש להכנס למרחב מוגן במיידי ולחכות להוראות פיקוד העורף!

The banner clears automatically when the area leaves the API response.

**Audio siren:** Disabled by default. Set `SIREN_ENABLED = true` in the CONFIG block to enable a Web Audio API siren that overrides background music during alerts.

> **Note:** The oref.org.il API requires an Israeli IP and does not support CORS. The kiosk Chrome must be launched with `--disable-web-security` (see TV kiosk setup below).

---

## Google Sheets setup

The display reads from a single Google Sheet with **three tabs**.  
Sheet ID in use: `1wBrfbGsQGP-CTnb-OcNKdwyuRgnAxVLCwPtM6H_5DiQ`

> The sheet must be **shared as "Anyone with the link can view"**.  
> Data is fetched via [opensheet.elk.sh](https://opensheet.elk.sh) — no API key needed.

### Tab 1 — `messages`

Building-wide announcements shown in the right panel (landscape) or top-left (portrait).

| Column | Name | Description |
|---|---|---|
| A | `date_added` | Date the message was added — `DD/MM/YYYY` |
| B | `last_date_show` | Last day to show this message — `DD/MM/YYYY` (leave blank to show forever) |
| C | `active` | `TRUE` to show, `FALSE` to hide |
| D | `content` | The message text |

---

### Tab 2 — `events`

Upcoming building events shown in the right panel and on the calendar.

| Column | Name | Description |
|---|---|---|
| A | `date_added` | Date the event was added — `DD/MM/YYYY` |
| B | `event_date` | Date of the event — `DD/MM/YYYY` |
| C | `active` | `TRUE` to show, `FALSE` to hide |
| D | `content` | Event description |
| E | `location` | Optional location (e.g. `חדר ועד`) |

---

### Tab 3 — `birthdays`

Residents whose birthday is **today** get a banner at the top. All birthdays for the current month appear as chips on the calendar.

| Column | Name | Description |
|---|---|---|
| A | `name` | Resident's name |
| B | `apt` | Apartment number |
| C | `birthday` | Birthday in `DD/MM` format (year is ignored) |

---

## Configuration

All settings are at the top of `index.html` in the `CONFIG` block:

```javascript
const SHEET_ID      = "your-google-sheet-id";
const LATITUDE      = 32.432033;   // building GPS latitude
const LONGITUDE     = 34.946715;   // building GPS longitude
const RSS_URL       = "https://www.ynet.co.il/Integration/StoryRss1854.xml";
const REFRESH_MS    = 5 * 60 * 1000; // data refresh interval (5 minutes)

// Red Alert
const ALERT_AREA    = "חדרה מזרח"; // area name from Pikud HaOref
const ALERT_POLL_MS = 3000;         // poll every 3 seconds
const SIREN_ENABLED = false;        // set true to enable audio siren
```

To find your **Google Sheet ID**: open the sheet in a browser — it's the long string between `/d/` and `/edit` in the URL.

---

## Running locally

Serve with any static file server. With .NET:

```bash
dotnet serve -p 8080
```

Then open: http://localhost:8080/index.html

> **Note:** The red alert feature will show CORS errors in a normal browser. It only works in kiosk mode with `--disable-web-security` (see below).

---

## TV kiosk setup (recommended)

For a dedicated display TV, launch Chrome in kiosk mode with autoplay and CORS bypass enabled:

```
chrome.exe --kiosk --autoplay-policy=no-user-gesture-required --disable-web-security --user-data-dir=C:\tmp\chrome-kiosk http://localhost:8080/index.html
```

Or point directly at the GitHub Pages URL:

```
chrome.exe --kiosk --autoplay-policy=no-user-gesture-required --disable-web-security --user-data-dir=C:\tmp\chrome-kiosk https://eugenekrapivin.github.io/yafeh-nof-25/
```

| Flag | Purpose |
|---|---|
| `--kiosk` | Full-screen, no browser chrome |
| `--autoplay-policy=no-user-gesture-required` | Background music starts without user click |
| `--disable-web-security` | Bypass CORS for oref.org.il red alert API |
| `--user-data-dir=C:\tmp\chrome-kiosk` | Required when using `--disable-web-security` |

---

## Orientation support

The display automatically adapts to both:
- **Landscape** — calendar on the left, messages + events on the right, hourly + daily weather in the strip
- **Portrait** — weather stacked (current summary + hourly), messages + events side by side at top, full-width calendar below, daily forecast hidden
