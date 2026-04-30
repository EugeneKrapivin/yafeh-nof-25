# בניין יפה נוף 25 — Digital Signage Display

A full-screen building information display designed for a lobby TV. Built as a single static HTML file — no server, no build tools, no dependencies.

**Live display:** https://eugenekrapivin.github.io/yafeh-nof-25/

---

## What it shows

| Section | Source | Refresh |
|---|---|---|
| Clock & Hebrew/Gregorian date | Browser | Every second |
| Weather + 4-day forecast | [open-meteo.com](https://open-meteo.com) | Every 15 min |
| Building announcements | Google Sheets | Every 15 min |
| Upcoming events | Google Sheets | Every 15 min |
| Birthday banner | Google Sheets | Every 15 min |
| Monthly calendar with holidays & Shabbat times | [Hebcal API](https://www.hebcal.com) | Every 15 min |
| Scrolling news ticker | Ynet RSS via rss2json | Every 15 min |

---

## Google Sheets setup

The display reads from a single Google Sheet with **three tabs**.  
Sheet ID in use: `1wBrfbGsQGP-CTnb-OcNKdwyuRgnAxVLCwPtM6H_5DiQ`

> The sheet must be **shared as "Anyone with the link can view"**.  
> Data is fetched via [opensheet.elk.sh](https://opensheet.elk.sh) — no API key needed.

### Tab 1 — `messages`

Building-wide announcements shown in the left panel.

| Column | Name | Description |
|---|---|---|
| A | `date_added` | Date the message was added — `DD/MM/YYYY` |
| B | `last_date_show` | Last day to show this message — `DD/MM/YYYY` (leave blank to show forever) |
| C | `active` | `TRUE` to show, `FALSE` to hide |
| D | `content` | The message text |

**Example:**

| date_added | last_date_show | active | content |
|---|---|---|---|
| 01/05/2026 | 15/05/2026 | TRUE | ניקיון מרתף ביום חמישי בין 10:00-14:00 |
| 10/04/2026 | | TRUE | ועד הבית — פגישה ראשונה בחודש הבא |

---

### Tab 2 — `events`

Upcoming building events shown in the left panel and on the calendar.

| Column | Name | Description |
|---|---|---|
| A | `date_added` | Date the event was added — `DD/MM/YYYY` |
| B | `event_date` | Date of the event — `DD/MM/YYYY` |
| C | `active` | `TRUE` to show, `FALSE` to hide |
| D | `content` | Event description |
| E | `location` | Optional location (e.g. `חדר ועד`) |

**Example:**

| date_added | event_date | active | content | location |
|---|---|---|---|---|
| 01/05/2026 | 20/05/2026 | TRUE | אסיפת דיירים | חדר ועד |
| 01/05/2026 | 25/05/2026 | TRUE | יום הולדת לבניין | גג הבניין |

---

### Tab 3 — `birthdays`

Residents whose birthday is **today** get a banner at the top of the screen.

| Column | Name | Description |
|---|---|---|
| A | `name` | Resident's name |
| B | `apt` | Apartment number |
| C | `birthday` | Birthday in `DD/MM` format (year is ignored) |

**Example:**

| name | apt | birthday |
|---|---|---|
| ישראל ישראלי | 12 | 14/06 |
| רחל כהן | 7 | 30/04 |

---

## Configuration

All settings are at the top of `index.html` in the `CONFIG` block:

```javascript
const SHEET_ID         = "your-google-sheet-id";
const LATITUDE         = 32.432033;   // building GPS latitude
const LONGITUDE        = 34.946715;   // building GPS longitude
const RSS_URL          = "https://www.ynet.co.il/Integration/StoryRss1854.xml";
const NEWS_INTERVAL_MS = 9000;        // ms between news headlines (unused — ticker is continuous)
const REFRESH_MS       = 15 * 60 * 1000; // full data refresh interval (15 minutes)
```

To find your **Google Sheet ID**: open the sheet in a browser — it's the long string between `/d/` and `/edit` in the URL.

---

## Running locally

Serve with any static file server. With .NET:

```bash
dotnet serve -p 8080
```

Then open: http://localhost:8080/index.html

---

## TV kiosk setup (recommended)

For a dedicated display TV, launch Chrome in kiosk mode with autoplay enabled so the background music starts automatically:

```
chrome.exe --kiosk --autoplay-policy=no-user-gesture-required http://localhost:8080/index.html
```

Or point directly at the GitHub Pages URL:

```
chrome.exe --kiosk --autoplay-policy=no-user-gesture-required https://eugenekrapivin.github.io/yafeh-nof-25/
```

---

## Orientation support

The display automatically adapts to both:
- **Landscape** — two-column layout (messages + events | calendar)
- **Portrait** — stacked layout (weather → messages | events → calendar → news)
