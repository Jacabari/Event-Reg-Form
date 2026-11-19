# Event Registration + Prize Wheel

A static web app (no build step) for booth/event registration with a one-spin prize wheel.
One deployment serves many events. Each event is a JSON file in `events/`.

## How it works

- `index.html` loads `events/<id>.json`, chosen by the URL: `https://your-site/?event=paft2026`.
  With no `?event=`, the `default` in `events/index.json` is used.
- The form, wheel, prizes, colours and rules all come from that JSON file.
- `apps-script/Code.gs` is the backend (Google Apps Script). It checks for duplicates, **picks the winner**, and
  decrements stock under a lock, so two devices can never win the same last prize.
- One entry and one spin per person. A duplicate (same normalised email/phone) is refused. Someone who registered
  but never finished spinning can re-enter the same details to resume the same prize.

## Add a new event

1. Copy `events/_template.json` to `events/<my-event>.json` (letters, numbers, `-`, `_`).
2. Edit the name, colour, form fields, prizes and rules.
3. Set `scriptUrl` (see backend setup below).
4. Open `https://your-site/?event=<my-event>`.

Prize notes: `weight` is relative odds, `stock` is the starting amount, `grand: true` shows a "Grand Prize" tag,
`noPrize: true` makes an unlimited "No Prize" segment.
Stock is seeded into the sheet the first time an event runs. Changing `stock` in the JSON later does not change
existing sheet stock (use Admin > Reset, or edit the Inventory tab). New prize ids are added automatically.

Field types: `text`, `email`, `tel`, `number`, `textarea`, `select`, `radio`, `checkboxes`.
`form.dedupeField` decides who counts as "the same person" (default: the email field, else phone).

## Backend setup (once)

1. Create a Google Sheet. Open **Extensions > Apps Script** and paste `apps-script/Code.gs`.
2. **Project Settings > Script properties**: add `ADMIN_PIN` (your PIN for the Admin panel).
3. **Deploy > New deployment > Web app**. Execute as **Me**, access **Anyone**. Copy the `/exec` URL.
4. Paste that URL into `scriptUrl` in each event file.
5. After editing `Code.gs` later, use **Deploy > Manage deployments > Edit > New version** so the URL stays the same.

Each event gets two tabs in the sheet: `Inventory - <event>` and `Entries - <event>`.

## Try it without a backend

Add `?demo=1`, e.g. `http://localhost:5173/?event=paft2026&demo=1`. Data stays in the browser, Admin PIN is `1234`,
and a "Fill Sample" button appears.

## Offline behaviour

After the first visit, a service worker (`sw.js`) lets the page and event settings load even with no internet.
Registration itself needs the connection, because the server picks the prize and checks for duplicates. For a
venue with no signal, use the offline version in `../v3-offline/`.

`styles.css` is compiled Tailwind (no CDN). If you change `index.html`, rebuild it: `cd ../tools && npm install && npm run build`.

## Run locally

```bash
python3 -m http.server 5173
```

Then open http://localhost:5173/?demo=1 (the JSON files need http, not `file://`).

## Deploy (GitHub + Vercel)

Push this folder to GitHub, then import it in Vercel with Framework Preset **Other**, no build command, and the
root directory as output. `vercel.json` is included.

## Booth tips

- Gear icon: Admin (stock, CSV export, reset). Arrows icon: full screen.
- `behavior.autoResetSeconds` returns the screen to a blank form after inactivity.
- If the connection drops, registration is blocked (so no duplicate or oversold prizes), and "spin done"
  confirmations are queued and retried automatically.
