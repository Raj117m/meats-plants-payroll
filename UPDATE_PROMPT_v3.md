# Update Prompt v3 — Meats & Plants Staff & Payroll

Update the existing app. **Keep the current look, layout and all v2 features.** Only make the changes below, then do a light visual polish.

---

## 1. Break bonus choice moves to the Staff page

- Remove the "4 skipped breaks → extra pay / extra paid leave" choice from Settings and from the Staff edit drawer.
- On the owner's **Staff page**, each employee row/card shows a **"Break bank"** pill: ●●●○ (3 of 4 banked). When it hits 4, the pill turns green with **two inline buttons, "+1 Day Pay" and "+1 Paid Leave"**. The owner taps one and it's applied immediately (with an undo toast and an audit log entry).
- The bank resets after each redemption. Leftover banked breaks carry over to the next month.
- Settings keeps only the global number ("skipped breaks needed: 4").

## 2. Pending approvals visible on the Staff page too

- Leave requests still send a notification to the bell / inbox **and** appear on the Staff page:
  - each employee row shows a small butter-colored chip, "1 pending · Dec 9 (paid)";
  - clicking the chip opens a side drawer with the request (dates, paid/unpaid, how many paid leaves are left, reason) and **Approve / Deny** buttons, plus a **"Message"** button that opens `wa.me/<their phone>` with a pre-filled note ("Hi Ravi, about your leave on 9 Dec…").
- An approval made anywhere (inbox, Staff page, Attendance cell) updates everywhere instantly.

## 3. Geo-lock replaces the Wi-Fi lock

- Attendance actions (Start Shift, Start/End Break, End Shift) are only allowed when the phone's location is **inside the Meats & Plants geofence**.
- **Store location:** `LAT, LNG = <to be filled from Google Maps link>`. Store it in Settings → Location, with a small map preview, a draggable pin and a **radius slider (default 75 m)**.
- Flow:
  - On Today, ask for location permission with a friendly explanation ("We only check your location when you tap a button").
  - When a button is tapped, get a fresh high-accuracy position (`navigator.geolocation.getCurrentPosition`, `enableHighAccuracy: true`, timeout 10s).
  - Allowed if the distance to the store (haversine) is ≤ radius + accuracy buffer (cap the buffer at 50 m). If the reported accuracy is worse than 100 m, ask them to retry.
  - **Validate on the server too.** Send lat/lng/accuracy with each punch, recompute the distance server-side, and save it on the punch record.
- Status pill on Today:
  - green: "📍 You're at Meats & Plants";
  - red: "📍 You're 1.2 km away. Head to the café to start your shift";
  - grey: "Location off. Tap to allow".
- If location permission is denied, show a short how-to for turning it on in iPhone/Android settings.
- Remove the Wi-Fi SSID check entirely (also from Settings).
- Owner manual punch still bypasses the lock.

## 4. Attendance matrix = the main hub

Keep the page looking as it does now, but make it the place where everything can be reached:
- **Click an employee name** → a quick profile drawer with: pay type and rate (editable inline), paid leaves left this month, late marks, break bank (with the +1 Day Pay / +1 Paid Leave buttons when ready), pending requests, and this month's net pay. At the bottom are two links: **"Open in Staff →"** and **"Open in Payroll →"**, which go to that page with the employee pre-selected and highlighted.
- **Hover a day cell** (on mobile, tap and hold) → a small tooltip card:
  - date + status chip (Present / Late / Paid leave / Absent / Half-day / Pending);
  - In 12:04 PM · Out 11:52 PM (show "auto" or "manual" tags if they apply);
  - 3-hr break: Taken 3:10–6:10 PM / **Skipped (banked)** / Partial;
  - **Today's pay: ₹600** (or "−₹300 half-day", "₹0 unpaid");
  - a note, if there is one.
- **Click a day cell** → the existing edit drawer (unchanged).
- Add a row of totals at the bottom of the matrix (present days and pay for the month) and a column of totals on the right (net pay per employee). Clicking a total → Payroll.
- Everything stays editable from Staff, Payroll and Settings as well; these are just shortcuts.

## 5. Fonts and polish

- **Replace the current fonts.** Use one family everywhere: **"Manrope"** (Google Fonts), weights 400/500/600/700/800.
  - Headings: 700–800 with tight letter-spacing (−0.02em).
  - Body: 500.
  - Numbers: 700 with `font-variant-numeric: tabular-nums`.
  - Remove the serif font completely.
- Type scale: 28 / 20 / 16 / 14 / 12, line-height 1.4.
- Slightly increase the white space between sections (32px).
- Align everything to the 4px grid.
- Make the chip colors softer (tint backgrounds + darker text).
- Use a single accent for primary actions (plant green `#2F7D4F`).
- Use one consistent card radius (16px).
- Keep the friendly micro-copy.
- Don't change the overall layout or palette.

**Deliver:** the updated app running with seeded data. This should include at least one employee with 4 banked breaks (so the redeem buttons show) and two pending leave requests, so the new Staff-page features are visible in the demo.
