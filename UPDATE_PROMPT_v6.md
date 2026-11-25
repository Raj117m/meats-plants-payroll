# Update Prompt v6 — Location, Glass Buttons, Responsive Owner Dashboard

Keep everything from v5 (logic, Quartz theme, rules). Only make these changes.

---

## 1. Set the café location (hard default)

- **Meats & Plants Thane** — Shop No. 21, Hiranandani Estate, Thane
- **Latitude `19.2584498`, Longitude `72.977272`** (Google place ID `0x3be7bb88c17beae1:0x6bb86305fdf8c469`)
- **Radius: 150 m** (raised from 75 m to allow for indoor GPS drift). The slider range stays 25–300 m.
- Save these values as the default `store_lat`, `store_lng` and `radius_m` in the database and seed. The owner can still move the pin from Settings → Location.
- The server must always use these values from the database for the punch check, never values sent by the phone.
- The Settings map opens centred on this pin at zoom level 17, with the 150 m circle visible.

## 2. Theme polish + glass button animations

Keep the Quartz palette (quartz white `#FAFAF8`, black `#0A0A0A`, violet-red `#B0235F` accents only) and the Inter font. Add a refined **glassmorphism layer on interactive elements only**:

- **Glass buttons:**
  - Background `rgba(255,255,255,0.55)`, `backdrop-filter: blur(14px) saturate(160%)`.
  - Border 1px `rgba(255,255,255,0.7)` with an inner hairline `inset 0 1px 0 rgba(255,255,255,0.8)`.
  - Very soft outer shadow `0 4px 16px rgba(10,10,10,0.06)`.
- **Primary glass button** (Start Shift, Approve, Save): black glass `rgba(10,10,10,0.88)` with white text, a subtle top highlight and a faint violet-red glow on hover (`0 0 0 4px rgba(176,35,95,0.10)`).
- **Animations** (150–250ms, `cubic-bezier(.2,.8,.2,1)`):
  - hover: lift by 1px and brighten;
  - press: `scale(0.97)` with an inner shadow;
  - release: a springy bounce back;
  - a **light sheen sweep** across the button on hover (a diagonal white gradient moving left → right in 600ms);
  - a ripple from the tap point on mobile;
  - focus: a violet-red ring that is 2px wide with a 2px offset.
- **Start Shift button:** a large circular glass button with a slow, subtle breathing glow while waiting. When pressed, it shrinks into the timer dial (a smooth morph), then the violet-red arc starts drawing.
- Apply glass to the bottom tab bar (staff), the top bar (owner), floating action buttons, bottom sheets and modals. Cards stay solid white with hairline borders, with no glass on cards, to keep it clean.
- Behind the glass, use a very faint background texture: a soft quartz noise or a light radial gradient of `#FFFFFF → #F2F2EF`. No colorful blobs.
- Respect `prefers-reduced-motion` (turn off the sheen, bounce and glow).
- If `backdrop-filter` isn't supported, fall back to solid `#FFFFFF`.

## 3. Owner dashboard — fully responsive (PC + mobile website)

The owner dashboard must work perfectly on **desktop (≥1024px), tablet (768–1023px) and phone (<768px)**:

- **Desktop:** a collapsible left sidebar, top glass bar, max content width 1280px, multi-column grids.
- **Tablet:** the sidebar collapses to icons only; the grids go to 2 columns.
- **Phone:**
  - The sidebar becomes a **glass bottom tab bar** with the 5 main items: Today · Attendance · Payroll · Staff · More. "More" holds Holidays, Leave Requests, Rules and Settings.
  - The top bar shrinks to a page title + bell + month arrows.
  - **Today:** employee cards stack in one column.
  - **Attendance matrix:**
    - the name column stays sticky and the days scroll sideways with momentum;
    - "today" is auto-scrolled into view;
    - tapping a cell opens a **bottom sheet** instead of a side drawer;
    - long-press shows the hover tooltip.
  - **Payroll:** the table becomes stacked cards (name, net pay in large text, a "View payslip" button). The summary strip becomes a 2×2 grid. Export buttons sit in a sticky bottom bar.
  - **Staff / Settings:** single-column forms, full-width inputs, a sticky Save bar above the tab bar.
  - All drawers become bottom sheets with a drag handle.
  - Tap targets are ≥ 44px. No horizontal page scroll except inside the matrix.
- Use `viewport-fit=cover` and `env(safe-area-inset-*)` padding for iPhone notches and home bars.
- Test at 1440, 1024, 768, 390 and 360px widths.

**Deliver:**
- the app running with the Thane location and 150 m radius preset;
- the glass buttons and animations on both the staff and owner sides;
- the owner dashboard verified on desktop and phone widths (include screenshots of each).
