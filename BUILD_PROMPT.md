# Build Prompt — Meats & Plants Attendance & Payroll

Build a clean, premium, **white-themed** web app called **Meats & Plants — Staff & Payroll** for a café with 10 employees. It has two sides: a **mobile-first Employee app** and a **responsive Owner Dashboard** (laptop + phone). This version is a **working UI prototype with a real backend** (database, auth, rules engine). The goal is to impress the owner: it must feel calm, simple and expensive, not cluttered.

---

## 1. Design direction

**Feel:** Apple-Health-meets-Linear. Lots of white space, one action per screen, big tap targets, soft shadows, rounded corners (16–20px), no heavy borders, no dark mode needed.

**Palette (earthy "meats + plants"):**
| Role | Color |
|---|---|
| Background | `#FFFFFF`, sections `#F7F6F3` (warm off-white) |
| Text primary / secondary | `#1C1C1A` / `#6B6B66` |
| Brand – Plant green (primary, Check-In, Present) | `#2F7D4F` (tint `#E8F3EC`) |
| Brand – Meat terracotta (accent, logo, highlights) | `#B5523B` (tint `#F8EAE6`) |
| Break / Late (amber) | `#D99A2B` (tint `#FBF3E2`) |
| Absent / Deduction (red) | `#C8463D` (tint `#FBE9E7`) |
| Paid leave / Manager edit (slate blue) | `#3E6FA8` (tint `#E7EEF7`) |
| Dividers | `#ECEAE5` |

**Type:** Inter (or SF Pro). Numbers in tabular figures. Large hero numbers (40–56px) for the clock and salary. Currency in **₹**, Indian number format (₹1,74,350).

**Motion:** subtle 150–200ms transitions, a satisfying press/ripple + check animation on Check-In, live pulsing dot for "on floor". No gimmicks.

**Logo:** wordmark "Meats & Plants", with the "&" in terracotta and a small leaf icon in green.

---

## 2. Core rules (the backend must enforce these)

- **Shift:** 12:00 PM → 12:00 AM (configurable).
- **Wi-Fi lock:** employees can only Check-In / Break / Check-Out while connected to the café network. **Trial setup:** only allow when connected to the Wi-Fi SSID **`ABHIGIT 5G`**; block on **`ABHIGIT 4G`** or any other network. Store the allowed SSID(s) in Settings so the owner can change them later.
  - ⚠️ Technical note: a normal browser **cannot read the Wi-Fi name (SSID)**, and both bands of one router share the same public IP. So wrap the employee app with **Capacitor (Android/iOS)** and read the SSID natively (Android: `WifiManager`, needs location permission; iOS: `NEHotspotNetwork.fetchCurrent`, needs the "Access WiFi Information" entitlement). Send the SSID with each punch and **validate it on the server** too. For the browser demo, show the detected network and include a hidden demo toggle to simulate "ABHIGIT 5G" vs "ABHIGIT 4G".
- **Grace period:** configurable (default 15 min). Check-In after 12:15 PM = **Late Mark**. **3 Late Marks = ½-day deduction** (configurable).
- **3-hour break:** each employee has one 3-hour break per shift that they can **start any time between shift start and shift end**. One tap to Start Break, one tap to End Break. Show a live countdown. If they **skip the break entirely**, it is banked. **4 skipped breaks = +1 day's pay OR +1 paid leave** (owner picks which in Settings or per employee).
- **Leaves:** **3 paid leaves per month**. After that, each absence is unpaid.
- **Deductions per day:** the owner can mark any day as **Full-day deduction** or **Half-day deduction** (or none). Daily rate = monthly salary ÷ days in month.
- **Salary:** the owner sets each employee's monthly base salary individually. All deductions and bonuses are calculated from that person's own base.
- **Auto check-out:** at 12:00 AM, anyone still checked in is automatically checked out (marked "auto" in the log so the owner can see it).
- **Reminders (push):** if the employee is on the café Wi-Fi and hasn't checked in 10 min after shift start → notification. Notifications also 10 min before the break ends and when it ends.
- **Every manual change is logged** (who, what, when, old → new) in an audit trail.

---

## 3. Employee app (mobile-first, max 3 tabs: **Today · My Pay · Leave**)

Login: phone number + 4-digit PIN (set by the owner).

**Today (home)** — one hero screen:
- Greeting + name, today's date, shift time.
- Network pill at the top: green "Connected · ABHIGIT 5G ✓" or red "Not on café Wi-Fi — connect to ABHIGIT 5G". When red, every button is disabled with a lock icon.
- A large live clock / timer ring in the centre (time worked today).
- **One big primary button that changes state:** `Check In` → after check-in, two buttons: `Start 3-hr Break` (amber) and `Check Out` (red outline). During a break: a countdown ring "2:14:05 left" + `End Break`.
- A small strip below: "Checked in 12:04 PM · On time" or "Late (12:21 PM) · Late mark 2 of 3".

**My Pay (Transparency Ledger, read-only):**
- Hero card: **Projected salary this month** (big number) + base salary underneath.
- 3 small stat tiles: Paid leaves left (e.g. 1 of 3), Late marks (2 of 3), Breaks banked (3 of 4, as a 4-dot progress bar).
- Clean line items: Base, Unpaid leave days (−), Half-day deductions (−), Late-mark deductions (−), Break bonus (+), **Net**.
- Mini month calendar with colored dots (present / late / paid leave / absent / half-day).

**Leave:**
- Calendar to pick future date(s), optional reason, and a "Request Leave" button. Show live whether the request will be **Paid** or **Unpaid** based on leaves left.
- A list of my requests with status chips: Pending / Approved / Denied.

---

## 4. Owner Dashboard (responsive; left sidebar on desktop, bottom tabs on phone)

Sidebar: **Today · Attendance · Payroll · Leave Requests (badge) · Staff · Settings**.

**Today (Live Roster):**
- 4 summary tiles: On floor · On break · Not in yet · On leave.
- A list/card grid of all 10 employees: avatar initials, name, status chip (On floor / On break with time left / Checked out / Absent / On leave), check-in time, check-out time, late badge. Live updates (websocket or 10-second polling).
- Each row has a "⋯" menu → **Manual Punch In / Out / Start Break / End Break** (for when Wi-Fi is down or a phone is dead). It asks for the time and a reason, and gets a small "manual" tag.

**Attendance (the editable calendar):**
- A month grid: rows = employees, columns = days. Each cell is a soft colored square: Present (green), Late (amber), Paid leave (blue), Absent/unpaid (red), Half-day (half-filled), Upcoming approved leave (blue outline).
- **Click any cell → a side drawer** showing the punch times for that day, and lets the owner:
  - edit check-in / check-out / break times,
  - **remove a Late mark**,
  - **change Absent → Paid Leave / Present**,
  - set the deduction to **None / Half day / Full day**,
  - add a note.
  The payroll recalculates instantly. The change is logged.
- Filter by employee; export the month.

**Payroll:**
- A table: Employee · Base · Paid leaves used · Unpaid days · Half-days · Late marks · Break bonus · Deductions · **Net pay**. A totals row at the bottom.
- Click a row → the full breakdown for that person.
- Buttons: **Export PDF** (one clean page per employee + a summary page, with the Meats & Plants branding) and **Copy for WhatsApp** (pre-formatted text per employee or for the whole team, with a "Share to WhatsApp" link using `wa.me`).
- "Finalize month" locks the numbers.

**Leave Requests:** a queue of cards (name, dates, paid/unpaid preview, reason) with **Approve** / **Deny**. Approved leave appears on the calendar in advance.

**Staff:** add/edit the 10 employees: name, phone, PIN, **monthly salary (₹, editable inline)**, joining date, active toggle, bonus preference (extra pay vs extra leave).

**Settings:** shift start/end, grace minutes, late marks per half-day, paid leaves per month, break length (3h), skipped breaks needed for a bonus, auto check-out time, **allowed Wi-Fi SSIDs** (default: `ABHIGIT 5G`).

---

## 5. Tech

- Frontend: React + Vite + Tailwind + shadcn/ui, lucide icons. PWA (installable). Capacitor wrapper for the employee app (SSID check + push notifications).
- Backend: Node (Express or Next.js API routes) + PostgreSQL (or Supabase). Auth with roles `owner` / `employee`.
- Tables: `employees`, `punches` (type, time, ssid, source: app/manual/auto), `days` (status, deduction: none/half/full, late flag, note), `leave_requests`, `settings`, `audit_log`, `payroll_months`.
- Cron jobs: auto check-out at 00:00, reminder push notifications, recompute payroll whenever a punch or day changes.
- Seed data: 10 sample employees with salaries between ₹14,000 and ₹22,000 and a half-month of realistic attendance so the demo looks alive.
- All payroll math must be server-side, deterministic, and covered by unit tests (grace period, 3 lates → ½ day, 4th paid leave → unpaid, 4 skipped breaks → bonus, owner overrides).

**Deliver:** a running app with both interfaces, seeded demo data, an owner login, and one employee login printed in the README.
