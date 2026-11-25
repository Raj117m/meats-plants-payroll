# Update Prompt v5 — Fix Attendance, Rules, Holidays, Theme

Keep all existing features unless a change below says otherwise. **Priority order: 1 → 2 → 3, then the rest.** Before finishing, test every flow end-to-end (see section 10).

---

## 1. 🔴 CRITICAL: Fix the Start Shift / attendance logic (staff AND owner side)

Right now, pressing **Start Shift** does not start the circular timer dial, and the owner's Start Shift / manual punch doesn't work. **Rewrite this logic from scratch** using one clear state machine shared by the staff app and the owner's manual punch.

**State machine (per employee, per day):**
```
NOT_STARTED → (startShift) → ON_SHIFT
ON_SHIFT    → (startBreak) → ON_BREAK      (only if no break taken yet today)
ON_BREAK    → (endBreak)   → ON_SHIFT      (auto-ends at 3h)
ON_SHIFT    → (endShift)   → ENDED
ON_BREAK    → (endShift)   → ENDED         (break auto-closed at that time)
any open    → (00:00 auto) → ENDED (source: auto)
```

**Requirements:**
- One backend endpoint: `POST /api/punch {employeeId, action, time?, source: app|manual, lat, lng, accuracy}`.
  - It validates the transition (invalid ones return a clear error message).
  - It writes the punch.
  - It returns the **full updated day state**: `{status, shiftStart, breakStart, breakEnd, breakUsed, shiftEnd, workedSeconds, breakRemainingSeconds}`.
- The frontend sets its state **only from that server response**. No optimistic state that can get stuck.
- On app load / refresh, call `GET /api/today/:employeeId` and restore the exact state. Refreshing the page must never lose or reset the timer.
- **Timer dial:**
  - Driven by `shiftStart` from the server: `worked = now − shiftStart − breakDuration`.
  - Uses a `setInterval` of 1s that starts **whenever status is ON_SHIFT or ON_BREAK**. Clear it on unmount.
  - Do NOT keep a local counter that starts at 0.
  - The ring progress is worked hours ÷ 12h.
  - During a break, the ring switches to a break countdown (3h → 0).
- The button shows a loading state while the request is in flight and is disabled to prevent double taps. Errors show as a toast with the reason (e.g. "You're 1.2 km from the café").
- **Owner manual punch** uses the **same endpoint** with `source: manual` and an optional custom time. It skips the location check and updates the Today roster and Attendance immediately.
- Store all times in UTC in the database and display them in **Asia/Kolkata**.
- Check for and remove the likely causes of the bug:
  - stale closure in `setInterval`;
  - timer started before the state is set;
  - timezone mismatch making `shiftStart` appear to be in the future;
  - a geolocation promise that never resolves (add a 15s timeout);
  - a missing `await`;
  - mismatched action names between the frontend and the backend.

## 2. Late rule (changed)

- **Grace period: 15 minutes after shift start (editable by the owner).**
- Check-in within the grace period = on time.
- **Check-in after the grace period = Half Day** (half a day's pay deducted). Remove the old "3 late marks = half day" rule.
- The owner can override any day (remove the half day / mark it as full).

## 3. Leave calculation (paid → unpaid automatically)

- The number of paid leaves per month is **set by the owner** (default 3, editable globally and per employee).
- Leaves are counted in date order within the month. Leaves 1–3 are **Paid**. The **4th and onward are automatically Unpaid**, and one full day's pay is deducted for each.
- **Government holidays (section 4) never count as leave and never deduct pay.**
- Extra paid holidays earned from the break bank (section 5) add to that month's paid-leave balance.
- Recalculate whenever a leave is added, approved, denied or edited.

## 4. Holiday calendar (government holidays)

- Add an **owner page: Holidays**, with a calendar view plus a list.
- Pre-load the **Indian national/gazetted holidays for 2026 and 2027** (Republic Day, Holi, Independence Day, Gandhi Jayanti, Diwali, Christmas, etc.).
- The owner can **add, edit or delete** holidays (date + name) and mark each one as paid holiday (default).
- Holidays show on **every calendar in the app** (staff My Month, staff Leave, owner Attendance) as a distinct marker, for example a violet-red outline with the holiday name in the tooltip.
- No attendance is needed on a holiday and no pay is deducted. If someone works on a holiday, show it as "Worked on holiday".

## 5. Break system — recheck and fix

**Rules:**
- Each shift includes **one 3-hour break**, which can be started at any time during the shift.
- Break taken: the timer counts down 3h. It auto-ends at 3h. Ending early is allowed (the unused time is not banked).
- **Break skipped** (the shift ended without starting a break): +1 to the **break bank**.
- **4 banked breaks = 1 extra full-day paid holiday.** It's credited automatically, the bank resets to 0, and a notification is shown to the staff member and the owner. (Remove the "+1 day pay" option. It is a paid holiday only.)
- The extra holiday shows in the staff's My Month view ("🎁 1 bonus paid holiday available") and adds to their paid-leave balance.
- The owner sees the bank on the Staff page and in the Attendance profile drawer.

**Write unit tests** for: break taken, break skipped, break auto-ending at 3h, ending the shift during a break, the 4th skip crediting a holiday, and the bank carrying across months.

## 6. Leave tab (staff) — simplified

Show **only** a clean list/table with these columns: **Date · Reason · Paid / Unpaid** (plus a status chip: Pending / Approved / Denied).
- Paid/Unpaid is decided automatically by the leave balance set by the owner.
- Keep one "+ Request leave" button that opens a sheet with: date picker, reason, and a preview of whether it will be paid or unpaid.
- Nothing else on this tab.

## 7. "How it works" rules screen

- Add a **"Rules" info page**, linked from the staff app (an ⓘ icon) and the owner sidebar. It explains the rules in plain language, with small icons:
  - **Working hours:** "Your shift runs 12:00 PM – 12:00 AM. Tap Start Shift when you arrive at the café. If you forget, you're checked out automatically at midnight."
  - **Late rule:** "You have a 15-minute grace period. Arriving after 12:15 PM counts as a half day."
  - **Breaks:** "You get one 3-hour break per shift. Start it any time. If you skip it, it's saved to your break bank. Skip 4 breaks → get 1 extra paid holiday."
  - **Leaves:** "You get 3 paid leaves each month. From the 4th leave onward, one day's pay is deducted per leave. Government holidays are free and never deducted."
  - **Location:** "Attendance works only when you're at the café."
- All numbers come from Settings automatically, so the text updates when the owner changes the rules.

## 8. Payroll page — professional structure

Rebuild it as a clean accounting-style statement:
- Header: month selector (‹ Nov 2026 ›), status badge (Draft / Finalized), and buttons for **Export PDF**, **Copy for WhatsApp**, **Finalize**.
- **Summary strip:** Total gross · Total deductions · Total bonus · **Total net payable**.
- **Main table**, grouped columns with right-aligned numbers, tabular figures and zebra-free thin row dividers:

  | Employee | Rate (₹/day) | Working days | Present | Paid leave | Holidays | Unpaid | Half days | Gross | Deductions | **Net pay** |

- Totals row at the bottom (bold).
- Click a row → a **payslip drawer** that looks like a real payslip: Earnings section (days × rate, paid leaves, holidays, bonus holiday) and Deductions section (unpaid days, half days), with Net pay at the bottom. This is the same layout the PDF uses.
- PDF: A4, one payslip per employee plus a summary page, with the Meats & Plants header.

## 9. New theme — "Quartz" (replace the current look completely)

The current look feels generic and AI-made. Make it minimal, sharp and editorial:
- **Primary: quartz white and black.**
  - Background `#FAFAF8` (quartz white), surfaces `#FFFFFF`, sunken areas `#F2F2EF`.
  - Text and primary buttons `#0A0A0A` (black).
  - Muted text `#6E6E6A`, dividers `#E6E6E2`.
- **Accent: violet-red `#B0235F`, used only in touches:**
  - the active nav item;
  - the focus ring;
  - the timer ring progress;
  - holiday markers;
  - small highlights;
  - the "&" in the logo.
  - Never use it for large fills.
- Status colors must be muted and small (dots and chips only):
  - present `#2E7D4F`;
  - half day / break `#B7791F`;
  - unpaid `#B42318`;
  - paid leave `#3B5BA9`.
- **Remove:** gradients, emoji-heavy copy, colorful cards, big soft shadows, blobs, confetti, and generic illustrations.
- **Use:**
  - 1px hairline borders;
  - 10–12px radius;
  - no shadow (or a single 1px subtle one);
  - lots of white space;
  - a strong black heading hierarchy.
- **Font:** **Inter** (or Geist), weights 400/500/600, numbers in tabular-nums. Headings are 600 with tight tracking. Optionally use small uppercase labels (11px, +0.06em tracking) for section titles.
- **Primary buttons:** solid black with white text. **Secondary:** white with a black hairline border. **Destructive:** text-only red.
- **Timer dial:** a thin black track with a violet-red progress arc and a big black tabular time in the centre.
- Apply the same theme to the staff app and the owner dashboard.

## 10. Acceptance checklist (test every item before delivering)

- [ ] Staff taps Start Shift → the dial starts counting immediately → the page is refreshed → the dial continues from the correct time.
- [ ] Owner manual Start Shift for an employee → it appears as On shift in the Today roster, and the staff's dial is running.
- [ ] Start Break → a 3h countdown runs → End Break → back to the shift timer. The break can't be started a second time that day.
- [ ] Shift ended without a break → bank +1. The 4th → +1 paid holiday is credited and shown.
- [ ] Check-in at 12:14 → on time. Check-in at 12:16 → half day. The owner can override it.
- [ ] 4th leave in a month → Unpaid, with one day's pay deducted in Payroll.
- [ ] Government holiday → no deduction, shown on all calendars, editable by the owner.
- [ ] Payroll totals match the payslip drawer and the PDF.
- [ ] Midnight auto check-out works.

**Deliver:** the fixed and re-themed app running with seeded data, plus a short note listing what caused the timer bug and how it was fixed.
