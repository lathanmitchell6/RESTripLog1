# MTM Daily Trip Log

A Google Apps Script web app that lets residential-home staff file daily AM/PM transport trip logs from any phone or computer. Each submission is verified against a driver registry, turned into a PDF, filed in Google Drive by date, emailed to the right people, and monitored for compliance.

## Features

- **Guided mobile-friendly form** with required-field validation, a live license character counter and an illustrated help guide
- **Driver registry**: a driver's name and license are registered on first use; later submissions must match, with a live mismatch warning before submit
- **PDF generation**: fills a temporary copy of the home's AM or PM tab, exports it to PDF, then deletes the copy (the original tab is never edited)
- **Automatic filing** in Drive: `Root / Year / MM - Month / MM-DD to MM-DD / MM-DD`
- **Email delivery** of each PDF to a central inbox and to the home's own contacts
- **Submission log** on a `Submissions` sheet tab
- **Compliance alerts** per home: midday check (expects 1 log), evening check (expects 2), next-morning reminder
- **Weekly report** to Program Managers showing each home's total out of 10
- **Contacts read live** from a Google Doc, so recipient changes need no code edits

## Repository structure

| File | Purpose |
|------|---------|
| `Code.gs` | Server side: config, contacts lookup, submission handling, PDF export, Drive folders, alerts, weekly report |
| `Index.html` | Front end: form, validation, loading/success screens, driver mismatch popup |

## How it works

1. Staff choose a home, AM or PM, and the date of service, then enter driver details, trip times, attendant signature and the individuals who rode.
2. The server verifies the driver, copies the matching `<code> AM` / `<code> PM` tab, and fills in the copy. Individuals who did not ride are marked `NS`.
3. The copy is exported as a PDF to the home's Drive folder, logged on the `Submissions` tab, and emailed.
4. The temporary copy is deleted.
5. Time-based triggers count the files in each day folder and email alerts when logs are missing.

## Setup

### 1. Google resources

- A **Google Sheet** containing, for each home, two tabs named `<code> AM` and `<code> PM` (for example `LN AM`, `LN PM`).
  - Column A holds row labels such as `Date of Service`, `Driver's License`, `MTM Trip Number`, `Driver Signature`, `Driver Printed Name`.
  - Column B (`Member's Printed Name`) lists the individuals for that home. Edit names here to change the roster.
  - The header row is detected automatically, so shifted rows and columns still work.
- A **Drive root folder** per home.
- A **Google Doc** with a tab named `Residential Matrix`, containing a table. A row whose first cell is a home code or name alerts that home; a row containing "Program Manager" or "Res PM" alerts all homes.
- A **Gmail label** per home (referenced in config).

### 2. Configure `Code.gs`

Set these values at the top of the script:

```javascript
var HOMES = [
  { name: "LN", gmailLabel: "Leno Trip Logs", driveFolderId: "<FOLDER_ID>" },
  // one entry per home
];
var SUBMISSION_RECIPIENT = "triplog@example.org";
var CONTACTS_DOC_ID = "<DOC_ID>";
var SHEET_ID = "<SHEET_ID>";
var IT_ADMIN_EMAIL = "itadmin@example.org";
```

Other options: `LOGS_PER_DAY`, `NOON_HOUR`, `COPY_TO_HOME_CONTACTS`, `COPY_TO_PROGRAM_MANAGERS`, `SIGNATURE_FONT`, `DEPARTURE_MIN_MINUTES` / `DEPARTURE_MAX_MINUTES`.

> **Do not commit real IDs or email addresses to a public repository.** Keep them in a private repo, or move them into Script Properties and read them with `PropertiesService`.

### 3. Deploy

1. Create a new Apps Script project (or use [clasp](https://github.com/google/clasp)) and add `Code.gs` and `Index.html`.
2. Run `testContacts()` and `testSheetRead()` once from the editor to grant permissions and verify the contacts doc and every home's tabs.
3. **Deploy > New deployment > Web app.** Execute as the owner. Restrict access to your organization.

### 4. Add triggers

Add these time-driven triggers in the Apps Script editor:

| Function | Schedule |
|----------|----------|
| `checkDailyTripLogs` | Daily, 12pm to 1pm |
| `checkDailyTripLogs` | Daily, 7pm to 8pm |
| `resendMorningTripLogReport` | Daily, 6am to 7am |
| `sendWeeklyTripLogReport` | Weekly, Friday, 8pm to 9pm |

Alerts skip weekends automatically.

## Adding a home

1. Add the `<code> AM` and `<code> PM` tabs to the sheet.
2. Add an entry to `HOMES` with its code, Gmail label and Drive folder ID.
3. Add a row for the home to the `Residential Matrix` table.

## Notes and limitations

- **Pickup departure time is calculated**, not entered: it is the pickup time plus a random 1 to 3 minutes. Review whether that is appropriate for your compliance or billing requirements before relying on it.
- Licenses are stored in plaintext on the `Drivers` and `Submissions` tabs. Limit sheet access accordingly, or store a hash instead.
- Driver registration is trust-on-first-use. A typo at first registration must be corrected by editing the `Drivers` tab.
- Email failures are logged but do not block the submission.
- The weekly report counts files, so duplicate submissions can mask a missing log.

## Tech stack

Google Apps Script (Web App, Drive, Gmail, Sheets, Docs and Lock services) with a plain HTML, CSS and JavaScript front end. No external servers or libraries.

## License

Add a license of your choice (for example MIT) before publishing.
