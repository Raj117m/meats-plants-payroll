# Update Prompt v2 — Meats & Plants Staff & Payroll

Update the existing app. Keep everything that works (punch flow, 3-hour break, checkout, projected salary, payroll table, leave requests, manual punch, staff page). The changes below are about **access, the calendar, daily wages, approvals, and above all a full visual redesign** so the app feels warm, unified, friendly and easy, not cold or cluttered.

---

## 1. One link for everyone + owner access

- Everyone opens the **same URL**.
- The first screen is the **Staff app** (mobile layout).
- The Owner Dashboard is reached from a small, tasteful **"Owner" button** (top-right lock icon). Tapping it opens a password sheet. Owner password = `meats2026` for now, stored hashed in Settings and changeable later. A wrong password shows a gentle shake animation. The owner session is remembered on that device until they log out.

## 2. Staff app is ALWAYS a mobile view

- The staff side must always render as a phone layout (max-width ~430px, centred). On a laptop it appears as a centred phone-width column on a soft warm background, **never stretched into a desktop layout**.
- A bottom tab bar with 3 tabs: **Today · My Month · Leave**.

## 3. Staff login: by name, no passcode

- Login screen: "Who's working today?" with a grid of 10 friendly name cards (initial avatar + first name). There's also a search box.
- Tap your name → a confirm screen: "Hi Ravi! 👋" with a checkbox, **"Remember me on this phone"** (checked by default) → Continue.
- When remembered, opening the link goes straight to their Today screen. A small "Not you? Switch" link at the top logs them out.
- The owner adds and removes names from the Staff page.

## 4. Shift does NOT start automatically

- On **Today**, the network pill shows Wi-Fi status (allowed SSID: `ABHIGIT 5G`; `ABHIGIT 4G` is blocked).
- While not on the café Wi-Fi: a friendly locked state, "Connect to the café Wi-Fi to start your shift", with all buttons disabled.
- While on the café Wi-Fi: one big button, **"Start Shift"**. Nothing is recorded until they tap it. The flow after that is the same as now: Start 3-hr Break / End Break / End Shift (checkout).
- The 10-minute "you're on Wi-Fi but haven't started your shift" reminder stays.

## 5. "My Month" (renamed from My Pay)

- Keep the projected salary hero card. Add a line underneath it: **"Today's earning: ₹600"**, and show the earning per day in the breakdown, because staff think in **daily pay**.
- **Attendance calendar with dots:**
  - Has **‹ › arrows** to move through **any past, present or future month** without limit, plus a "Today" pill to jump back.
  - All history is stored permanently in the database per employee, not just the current month.
  - **Tap any future date** → a bottom sheet with two choices:
    - **Request Paid Leave**. Max **3 per month**. The sheet shows "2 of 3 left this month"; once 3 are used, it says "This will be unpaid".
    - **Request Leave (unpaid)**.
  - Requests made here are **pending until the owner approves**. Pending dates show a dotted outline; approved paid leave is solid blue; denied dates go back to normal and show a small toast notification.
  - **Tap a past date** → a read-only detail: in/out times, break taken or skipped, late or not, the day's earning or deduction.
- Each employee sees all of their own data (every month, every day). They can't see anyone else's.

## 6. Leave tab

- Keep it, but make it use the **same calendar component** as My Month (tap a date → request). Underneath: a list of "My requests" with Pending / Approved / Denied chips.

## 7. Owner Dashboard changes

- **Month navigation everywhere:** replace the "November 2026" dropdown with **‹ Month Year ›** arrows plus a "This month" button. This applies on the Attendance, Payroll and Today history views. Past, present and future months all load from saved data, and edits are saved per month.
- **Attendance:** future leave requests appear on the grid as dotted "Pending" cells and can be approved or denied right from the cell drawer. Nothing counts as leave until it's approved.
- **Notifications / Approvals inbox:** a bell icon in the top bar with a badge. One unified inbox shows:
  - leave requests (paid/unpaid),
  - **manual punch requests from staff** (add a "Forgot to punch? Request a fix" link on the staff Today screen: pick the time and reason, and it's sent to the owner),
  - auto check-outs at midnight to review.
  Each item has **Approve / Deny** buttons inline. Approved items update attendance and payroll instantly.
- **Staff → Edit:** add a **Pay type toggle: Daily wage | Monthly salary**.
  - Daily wage: the owner enters ₹/day. Payroll = present days × daily wage + paid leaves × daily wage − half-days × ½ daily wage + break bonus days × daily wage.
  - Monthly: as now (daily rate = monthly ÷ days in month).
  - Show both on the staff card ("₹600/day · ≈ ₹18,000/month").
- **Settings: redesign it.** It's cluttered now. Use grouped, collapsible cards with a plain-English sentence for each setting, for example:
  - 🕛 **Shift** — "Shift runs from [12:00 PM] to [12:00 AM]"
  - ⏱ **Lateness** — "Allow [15] minutes of grace. [3] late marks = half a day."
  - ☕ **Breaks** — "Each shift has a [3]-hour break. Skip [4] breaks → [+1 day pay ▾]."
  - 🌴 **Leave** — "[3] paid leaves per month. Owner must approve: [on]."
  - 📶 **Wi-Fi** — "Allowed networks: [ABHIGIT 5G ×] [+ add]"
  - 🔒 **Owner password** — [Change]

  One setting per row, inline inputs inside the sentence, and a single sticky "Save changes" bar that appears only when something has changed.

---

## 8. VISUAL REDESIGN (most important)

The current dashboard layout feels cold, boxy and inconsistent. Redesign both sides so they feel **warm, friendly, calm and premium**: think Notion × Headspace × a nice café menu.

**Fonts — unified, two fonts only:**
- Headings and big numbers: **"Fraunces"** (soft serif, warm, a bit playful) or **"DM Serif Display"**.
- Everything else: **"Plus Jakarta Sans"** (friendly, rounded, very readable).
- Use one type scale everywhere: 32 / 24 / 18 / 15 / 13. Numbers use tabular figures. No other fonts anywhere.

**Palette — keep it white, add warmth:**
- Page `#FFFDF9` (warm white), cards `#FFFFFF`, sunken areas `#F6F2EA` (cream).
- Text `#22201C`, muted text `#7A746A`.
- Plant green `#2F7D4F` (primary: Start Shift, Present, Approve).
- Terracotta `#C0573E` (brand accent: logo "&", highlights, the active tab).
- Butter `#E8B44A` (break, late), Rose `#D9534A` (absent, deduct), Sky `#4A7BB7` (paid leave, pending).
- Every status color also gets a light tint for chips and backgrounds.

**Layout (owner dashboard):**
- A slim left sidebar (icons + labels, collapsible). A top bar with a **greeting** ("Good afternoon, {owner name} ☀️", with the owner name set in Settings), month arrows, the bell, and the profile.
- A maximum content width of ~1200px, a generous 24–32px gutter, and one clear grid of 12 columns.
- Every page starts with a **one-line friendly summary** in the serif font, for example: "8 of 10 are in today. Sana's on break till 6:10 PM."
- Cards: 20px radius, a very soft shadow (`0 1px 2px rgb(0 0 0/.04), 0 8px 24px rgb(0 0 0/.04)`), no hard borders.
- Tables become **airy lists**: 56px rows, avatar + name on the left, chips in the middle, the amount on the right. Hovering shows the actions.
- Today page: employee **cards** in a 2–3 column grid, each with a live status ring (green = on floor, butter = break with countdown, grey = not in).
- Attendance grid: rounded squares with tooltips; today's column is highlighted; there's a sticky name column and a sticky header.

**Fun to read:**
- Friendly micro-copy everywhere: "All caught up 🎉" (empty inbox), "No lates this month — legend." (staff ledger), "Payday in 6 days 💸".
- Small illustrations or emoji for empty states.
- Satisfying micro-interactions: confetti on the 4th banked break, a check-mark draw on Start Shift, smooth number count-ups on salary.
- Keep it **uncluttered**: at most one primary button per screen section, progressive disclosure (details in drawers or bottom sheets), and no walls of numbers.

**Consistency:**
- Build a small design system first (tokens for color, type, radius, spacing in 4px steps, shadow) and use it everywhere on both the staff and owner sides.
- Use one icon set (Lucide, 1.75 stroke).
- Use the same chips, buttons, calendar and bottom-sheet components across the app.

**Deliver:** the updated app running, with seeded data across **at least 3 past months plus the current month** so that month navigation is visible, and a few pending leave and manual-punch requests in the inbox for the demo.
