# Update Prompt v4 — Map-based Location Lock

Keep everything else exactly as it is. Only fix and upgrade the GPS lock.

## 1. Owner picks the location on a map

- In **Settings → Location** (and as a shortcut button "📍 Set café location" on the Today page of the owner dashboard), show a **full-width interactive map**. Use **Leaflet + OpenStreetMap tiles**, which are free and need no API key.
- The owner can set the location in any of these ways:
  - **search** an address or place (Nominatim geocoding search box);
  - **click anywhere on the map** to drop the pin;
  - **drag the pin** to fine-tune it;
  - **"Use my current location"** button (for when the owner is standing in the café);
  - **paste a Google Maps link** or raw coordinates. Parse the `@lat,lng`, `?q=lat,lng` or `!3d…!4d…` formats from the URL.
- A **circle overlay** shows the allowed zone. A **radius slider** (25–300 m, default 75 m) resizes the circle live.
- Show the coordinates (6 decimals) and the radius under the map, then a **Save location** button. Save to the database (`settings.store_lat`, `store_lng`, `radius_m`). It applies to all staff instantly and the change is written to the audit log.
- Optional: allow **more than one location** (e.g. a second branch), each with a name, pin and radius.

## 2. Make the staff GPS check accurate and reliable

- When the staff member taps Start Shift / Break / End Shift:
  - Use `navigator.geolocation.watchPosition` with `{ enableHighAccuracy: true, maximumAge: 0, timeout: 15000 }`.
  - **Collect readings for up to 8 seconds** and keep the one with the **best (lowest) accuracy**. Stop early if accuracy is ≤ 30 m.
  - Show a small "Finding you… ±42 m" spinner while it searches.
- **Decision:** `distance = haversine(user, store)`. Allow if `distance − min(accuracy, 50) ≤ radius`.
  - If accuracy is > 150 m, show "Weak GPS signal. Step near a window and tap retry". Do not reject outright.
- **Server-side check:** recompute the same rule on the backend using the lat/lng/accuracy that was sent, and use the store location from the database (never from the client). Save lat, lng, accuracy and distance on every punch.
- Fix common bugs:
  - Make sure lat/lng are not swapped and are numbers, not strings.
  - Make sure the haversine formula uses radians and gives metres.
  - The site must be served over **HTTPS** (geolocation is blocked on http).
  - Handle the permission states `prompt`, `granted` and `denied` with clear messages and how-to-enable steps for iPhone (Safari → Location → Allow) and Android (Chrome → Site settings → Location).

## 3. Staff can see where they are

- On the staff Today screen, add a small **map card**. It shows the café circle (green) and the staff member's blue dot with its accuracy ring, plus the text "You're 40 m from Meats & Plants ✅" or "You're 1.3 km away ❌".
- There's a **Refresh location** button.

## 4. Owner can check it

- On every punch in the Attendance hover tooltip and day drawer, show "📍 32 m from café (±18 m)".
- A mini-map link opens the spot where the punch was made.
- Punches made close to the edge of the circle are flagged with a small amber dot so the owner can double-check them.

**Deliver:** the map picker working in Settings, the store pin pre-set to a default location (the owner will move it), and the improved GPS check on the staff side with distance shown.
