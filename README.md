Kotoba synchronization and Monday email setup
Read this first
The downloaded app works locally. No server has been deployed, no real email has
been sent, and no weekly email job is active for you. A calendar file is a
separate personal reminder; it is not an email scheduler.
To synchronize a desktop and a phone, both must connect to the same running
server. To send a Monday report while your desktop is off, this server must run
on an always-on machine with internet access. No paid hosting service is required
or included by this code; use infrastructure you already control.
The Android folder contains source code, not a compiled APK. It must be built
and device-tested using its BUILD-GUIDE.md. Phone networking requires a real HTTPS
origin, not your desktop's `localhost`.
1. Run the server on your desktop for a local test
Install Python 3.11 or newer. Open a terminal in this `server` folder:
```sh
python -m venv .venv
```
Windows PowerShell:
```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe setup.py
.\.venv\Scripts\python.exe run.py
```
macOS / Linux:
```sh
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python setup.py
.venv/bin/python run.py
```
Choose `http://127.0.0.1:8765` as the app origin for this desktop-only test.
Open that address in a browser, then choose Daily accountability → Connect
desktop & mobile. Enter the same origin and the private access token printed by
the local setup wizard. Read and accept the data-upload consent, then connect.
The public app page has no private progress in it. All progress API endpoints
require the private token. Keep that token private, just like a password.
The package uses a WSGI application with Waitress. Do not expose a development
HTTP server directly to the internet. The launcher starts both the web server
and the email-schedule worker; running a different WSGI command by itself will
not start the email worker.
2. Enable Monday emails on your own server
The setup wizard asks for your own report recipient, an approved sender, SMTP
host, port, username and password/app password. Enter these locally, not in
ChatGPT, an APK, a HTML file, GitHub source, or a public URL.
It creates a private `config.local.json`, excluded by `.gitignore`.
The important settings are:
```json
{
  "WEEKLY_EMAIL_ENABLED": true,
  "DAILY_EMAIL_ENABLED": false,
  "SMTP_HOST": "your-provider-smtp-host",
  "SMTP_PORT": 465,
  "SMTP_MODE": "ssl",
  "SMTP_USERNAME": "your-smtp-username",
  "SMTP_PASSWORD": "your-provider-password-or-app-password",
  "EMAIL_FROM": "your-approved-sender@example.com",
  "REPORT_TO": "your-own-address@example.com"
}
```
This is a configuration excerpt, not a file to overwrite your token with.
Use your provider's real settings. TLS modes supported by this implementation
are `ssl` (commonly port 465) and `starttls` (commonly port 587). There is no
unencrypted-SMTP mode and no built-in OAuth login flow; OAuth-only providers need
an additional authentication integration.
Set `WEEKLY_EMAIL_ENABLED` to `true` only when you approve sending to the
configured recipient. Restart `run.py` after changing the settings.
Prepared schedule: every Monday at 16:00 Asia/Kolkata.
The report covers the previous Monday–Sunday and the current outstanding
backlog. When tracking starts on 14 October 2026, the first eligible report slot
is Monday, 19 October 2026, at 4:00 PM India time. Earlier untracked dates in
that partial first week are excluded from the accounting.
The worker checks due work every 30 seconds. This is a target send time, not a
guarantee of exact inbox delivery. A stopped, sleeping, offline or misconfigured
server cannot send on time. On restart it checks the latest eligible missed
weekly slot, without sending an entire backlog of historical emails.
The database records successful SMTP acceptance and retries failed jobs after
backoff. A lease and stable Message-ID reduce duplicates. Exactly-once delivery
cannot be guaranteed: for example, the server could crash after the provider
accepts a message but before the database is updated. SMTP acceptance is not a
read receipt or an inbox-delivery guarantee.
Preview a report without sending anything:
```sh
python run.py --preview-report
```
After enabling the server, open Daily accountability and inspect its email
status: configured recipient (masked), worker activity, and last SMTP-accepted
job. No report is sent for a period entirely before tracking began.
3. Connect your phone and desktop to a real HTTPS deployment
For use away from your desktop, deploy this server on a persistent machine you
control, with a valid HTTPS domain/reverse proxy. Keep Waitress bound to
`127.0.0.1:8765` behind that proxy. Use your operating system's service manager to
restart `python run.py` after failures and machine restarts.
Set `PUBLIC_ORIGIN` to the real `https://...` origin, with no trailing slash, path,
query or credentials. The Android client requires the standard HTTPS port 443.
Do not turn off certificate checks or Android cleartext protections.
On both devices enter that same HTTPS origin and private access token under
Daily accountability → Connect desktop & mobile. Use the updated APK built
from this package on Android, or the web page in a mobile browser.
The allowed cross-origin clients are the bundled Android app origin and
standalone HTML (`null` origin), plus your exact `PUBLIC_ORIGIN`. Authentication
uses a bearer token, never an ambient login cookie; wildcard CORS is not used.
Remove `null` from ALLOWED_ORIGINS when you do not need standalone HTML access.
A permanent URL, TLS certificate, machine/service setup, recipient and SMTP
credentials are not supplied or configured by the assistant. That is the
remaining deployment work. A localhost test is not a remote mobile deployment.
4. Use the weekly routine
Monday–Friday: use the desktop checklist and course. Mark the four skill
groups you actually practised and log actual minutes. Saving the existing focus
timer also records minutes in the tracker; do not log the same time a second time.
Weekends and your marked holidays: use the mobile course and the same tracker.
Default targets are 90 minutes for weekdays and 45 minutes for mobile days.
These are editable starting suggestions, not a guarantee of N2 readiness.
Targets can be changed from tomorrow onward. Mark holidays manually; they are
mobile study days, not automatically excused absences.
Before closing a session: check the sync message. Offline work stays on its
device until it is opened and synchronized. In particular, synchronize the
phone after your Sunday work so Monday's email can include it.
Monday 4:00 PM: review the prior week's report, choose catch-up dates for
outstanding work, and adjust tomorrow's time target when needed.
Accountability rules
A day ends at midnight India time. It is done only when all four tasks are checked
and its allocated minute target is met. Until the day closes, unfinished work is
pending/partial, not a missed day.
A past incomplete date remains in Open missed work. Select it, record a reason
and catch-up date, and complete its checklist/time allocation. It then moves to
Completed late. Its original missed deadline is retained. A catch-up plan
alone does not complete any work.
Late minutes have two dates: the date of work they are allocated to and the actual
date they were logged. Weekly actual study time counts them only on the latter.
The report labels allocated minutes separately, so catch-up is not counted twice.
Records use device timestamps and manual check-offs. They are a personal
accountability log, not a tamper-proof attendance system or a language assessment.
No activity is inferred before the tracker was first opened.
Notifications
The in-app missed-work banner is always present. You can enable a daily local
reminder (default 20:00 India time; editable) and grant desktop notification
permission where the browser supports it. The app must be open; browser sleep,
throttling and permission settings can delay or suppress browser alerts.
This Android build has in-app reminders but not native background push
notifications. For optional reminders when apps are closed, enable
`DAILY_EMAIL_ENABLED` on the running server and set `DAILY_REMINDER_TIME`. It sends
at most one unfinished-study reminder per day from synchronized data.
Importing the included calendar file adds a separate Monday review event in your
calendar only when you choose to import it. It does not send the weekly report,
connect accounts, or activate anything in ChatGPT.
Synchronization and privacy
Tracker events are merged by unique ID, so retrying an upload does not duplicate
time or check-offs. Concurrent edits to the same check-off use the latest device
timestamp; keep both devices' clocks correct.
Course snapshots use revision checks. Switching devices sequentially synchronizes
normally. Editing the course offline on both devices can produce a conflict.
The app keeps local changes and asks whether to use the server course or this
device's course. Export a portable backup first. Up to 20 older server course
snapshots are retained for recovery; this is not a full automatic lesson merge.
The synchronized data includes lesson scores, reviews, saved writing, speaking
logs and the tracker. The weekly email includes metrics, accountability notes and
catch-up plans, not the full notebook, saved conversation text, SMTP secrets or
access token. Put only relevant study information in accountability notes.
The app access token is session-only and may need re-entry after reopening a
browser/app. It is not exported with progress. Mail credentials stay only in
private server configuration. There is no third-party analytics or AI API call.
A local reset disconnects that device and does not delete the server copy.
Disconnecting one device does not disable server emails; set
`WEEKLY_EMAIL_ENABLED` / `DAILY_EMAIL_ENABLED` to false and restart the server to
stop them. Importing a backup while connected counts as a course edit that may
synchronize; disconnect before importing to keep the server copy untouched.
Back up both the app's JSON and the database. Use a consistent database backup:
```sh
python backup_db.py /a/private/path/kotoba-backup.sqlite3
```
Do not put configuration, databases or backups under `static/` or in a public
repository. The database and configuration are not encrypted at rest by this
app; use your host's file permissions, disk encryption and secure backup policy.
Testing and remaining checks
```sh
python -m unittest discover -s tests -v
```
All test email sending is mocked. Unit tests cover the Monday timezone boundary,
late completions, no double-counted catch-up time, merge idempotency, authentication,
course conflicts, and scheduler retry/deduplication. The browser integration test
uses a WSGI-backed transport mock, not a live public HTTPS connection.
The Android SDK is not available in the preparation environment. APK compilation,
installation, native networking, real SMTP, public TLS deployment and real-device
background behavior remain untested. These checks must be done on your setup.
Primary documentation
Waitress deployment/usage: https://docs.pylonsproject.org/projects/waitress/en/stable/usage.html
Python SMTP and TLS: https://docs.python.org/3/library/smtplib.html
Browser notifications: https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API
Android WebView safety: https://developer.android.com/privacy-and-security/risks/insecure-webview-native-bridges
