# QR Attendance Register

A daily attendance system built with **n8n**, **Google Sheets** and a small self-hosted web page. Staff scan a QR code, type their name, and their attendance is logged with a timestamp. It is in live use at my workplace.

## The problem

Every day, someone had to check a paper sign-in form against a list of names and tick attendance in Excel by hand. It was slow, repetitive and easy to get wrong.

## The solution

1. A printed QR code sits at the entrance.
2. Staff scan it with their phone and open a simple scan page.
3. They enter their name and surname and submit.
4. The submission is logged to a Google Sheet with the **date, weekday and time**.
5. Every morning, the previous day's records are archived to a permanent log and the daily sheet is cleared.

No app to install, no accounts for staff, and no manual ticking.

## How it works

```
QR code --> scan page (served by n8n) --> webhook (POST) --> n8n workflow
                                                              |
                                      duplicate check (device ID, Google Sheet)
                                                              |
                                           append row to "today" sheet
                                                              |
        07:00 daily (SAST) --> archive to permanent log --> clear "today" sheet
```

### Components

| Part | What it does |
|------|--------------|
| **Scan page** | A custom HTML page, served directly by an n8n GET webhook, so no third-party hosting is needed. |
| **Webhook trigger (POST)** | Receives the name, surname and device ID from the page. |
| **n8n workflow** | Checks for duplicates, formats the timestamp and writes to Google Sheets. |
| **Google Sheets** | One tab for the current day and one permanent archive tab. |
| **Schedule trigger** | Runs at 07:00 South African time, archives the day's records, then clears the daily sheet. |

## Features

- **Timestamp fields:** separate Date, Day (weekday name) and Time columns.
- **Correct timezone:** the workflow runs in Africa/Johannesburg (SAST). Early versions logged times in the server's default timezone, which I found and fixed.
- **Permanent history:** each day's records are archived before the daily sheet is wiped, so no data is lost.
- **Buddy-punching protection:** an early version blocked repeat names, but that could be bypassed by typing a different name. The current version limits each phone to **one sign-in per day**, whatever name is typed, using a persistent per-device ID.
- **Self-hosted scan page:** not every employee has an account on a particular platform, so the page is served from the same n8n instance as the workflow.
- **Printable QR sign:** a ready-to-print sign (HTML and PDF) links to the scan page.

## Tech stack

- n8n (workflow automation, webhooks, scheduling)
- Google Sheets (storage)
- HTML and JavaScript (scan page)
- Railway (hosting for n8n)

## Results

- Replaced the manual daily sign-in check with an automated process.
- Used daily by **40+ people** at my workplace, with the workplace's approval.
- Time saved per day: **Used daily by 40+ people. Logged 2,000+ sign-ins since launch. Saves roughly 20 minutes of manual checking per day.**

## Lessons learned

- **Timezones matter.** Servers default to a different timezone, so set it explicitly in the workflow.
- **Test before launch.** A pre-launch review found a broken daily archive schedule rule and fixed it before it affected real data.
- **Expired credentials break live systems.** A Google OAuth expiry briefly stopped live scans. It recovered, and I now check the connection when something stops logging.
- **Some phones block browser storage.** A small number of scans came from browsers that blocked local storage, so the once-per-phone check did not apply to them. The name and timestamp were still logged correctly.

## Limitations

- The device check relies on browser storage, so it can be bypassed by clearing it or using a different browser. It deters casual buddy-punching but is not a security guarantee.
- It records who signed in. It does not verify identity.

## Privacy

The system stores names and timestamps only. Access to the Google Sheet is limited to the people who need it, and no personal details beyond the name are collected.

## Possible improvements

- Add a simple dashboard for daily and weekly attendance summaries.
- Send a daily report by email or Telegram.
- Add stronger identity checks, for example a one-time code or signed links.

## Author

**Mnqobi**, automation engineer based in Cape Town, South Africa.
GitHub: https://github.com/nmnqobi29-hub
