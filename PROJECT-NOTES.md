# AR Walking Tour of Salisbury NC — Project Notes

Working notes and hard-won lessons for the AR walking tour. Read this before
changing the AR pages so we don't re-learn the same things.

Live base URL: https://ackepley.github.io/AR-Walking-Tour-of-Salisbury-NC/
GitHub user: ackepley · Hosting: GitHub Pages · Tools: A-Frame, AR.js (all free)

---

## The big lesson: elevation / altitude (June 2026)

**Symptom:** The Bell Tower sign was completely invisible on-site even though
GPS, distance, and compass all looked correct.

**Cause:** AR.js `gps-entity-place` positions objects using GPS *altitude*.
Salisbury sits ~233 m (≈764 ft) above sea level. The phone reported its real
altitude (~236 m), but the sign had no altitude set, so AR.js placed it at
"sea level" — i.e. **236 m straight down, underground.** The diagnostic readout
showed this as `y -236.0`.

**Root cause (exact):** AR.js `gps-entity-place` subtracts your GPS altitude from
the entity's y — but ONLY when that y is non-zero
(`if (position.y !== 0) { position.y = position.y - altitude; }`). A sign at
`position="0 1.6 0"` therefore became 1.6 − 236 ≈ −234 (underground).

**Fix that works:** keep the GPS-anchored entity at **y = 0** so AR.js skips the
altitude math, and raise the *visible panel* to eye level with its OWN local
position:

```html
<a-entity gps-entity-place="latitude: ..; longitude: .." position="0 0 0" face-camera>
  <a-plane id="signPlane" position="0 1.6 0" width="4" height="3"
           material="shader: flat; side: double"></a-plane>
</a-entity>
```

An earlier `fixed-height` per-frame y-override was tried and **removed** — it
never attached (see the head-registration lesson below), and this source-level
fix is cleaner anyway.

**Why this is elevation-proof:** the panel height is relative to the GPS anchor,
not an absolute sea-level number. Works uphill/downhill anywhere in town.

Verified June 2026 via the diagnostic readout: `anchor ... y 0.0`,
`sign panel height y: 1.6m`, `billboard: on`.

---

## Lesson: register custom components in <head>, before <a-scene>

Custom A-Frame components (`face-camera`, `smooth-heading`, any new ones) MUST be
registered with `AFRAME.registerComponent(...)` in a `<script>` in the `<head>`,
BEFORE `<a-scene>` is parsed. If defined at the bottom of `<body>`, the scene
initializes the entities first and the components silently never attach — the
diagnostic showed `billboard: OFF` and the sign didn't billboard / the old
height override did nothing. (GPS components work only because they load from the
AR.js file in `<head>`.) Confirm with the diagnostic's `billboard:` line.

## Pattern for adding a NEW sign (e.g. Rowan Museum / Old Courthouse)

1. Get the target latitude/longitude for that spot (calibrate on-site, or a map).
2. Entity: `gps-entity-place="latitude: <lat>; longitude: <lon>" position="0 0 0" face-camera`.
3. Inner `<a-plane>` carries the height: `position="0 1.6 0"` (eye level).
4. That's it — elevation is handled automatically (entity y=0 skips altitude math).

Only change the plane's y if you *want* a sign deliberately high or low
(e.g. floating at the top of the actual tower → plane `position="0 12 0"`).

---

## Pattern: media pop-up (YouTube video, historic photos) + proximity trigger

Used on **rowan-museum.html** (the Old Courthouse stop). Instead of a 3D sign,
the live AR camera view is the backdrop and an HTML card pops up when you arrive.

**Why an HTML overlay, not a floating 3D screen:** a YouTube video is an
*iframe* and CANNOT be painted onto a WebGL / A-Frame surface — only a raw mp4
can be a 3D texture, and this stack dislikes video textures anyway. So media
(video, photos) shows as a 2D card layered over the camera feed. Reliable on
iPhone.

**How rowan-museum.html works:**
- Load the free YouTube IFrame API in `<head>`:
  `<script src="https://www.youtube.com/iframe_api"></script>` — it lets us
  autoplay / mute / unmute from JavaScript instead of making the visitor tap play.
- Keep the proven `gps-camera` scene; there is NO sign entity.
- Three constants near the top of the page script control everything:
    - `MUSEUM = { lat, lon }` — the trigger location.
    - `VIDEO_ID = "vRhn56OkTUQ"` — the part after `watch?v=` in the YouTube URL;
      change ONLY this to swap videos.
    - `TRIGGER_RADIUS = 15` — meters; how close you must be before it opens.
- `updateDiag()` runs every 0.5s, computes haversine distance to `MUSEUM`, and
  calls `openVideo()` once you're within `TRIGGER_RADIUS` (fires once; resets if
  you close it and walk back).
- `openVideo()` builds a `YT.Player` with `playerVars` `autoplay:1, mute:1,
  playsinline:1`. It autoplays MUTED; the green "Sound on" button calls
  `player.unMute()`.

**iOS rules baked in (don't fight these):**
- Autoplay WITH sound is blocked by Safari. MUTED autoplay is allowed. Turning on
  audio needs a user tap → that's the "Sound on" button.
- `playsinline:1` keeps it playing in-page instead of jumping to fullscreen.

**Testing:**
- `?preview` opens the media instantly with no GPS — test from home / the office.
- Real mode shows the green diagnostic; watch `distance` count down to the radius.
- Bump `?v=<n>` on each test to dodge Safari's cache.

**To add a NEW media stop:**
- Copy rowan-museum.html; set its own location constant + `TRIGGER_RADIUS`.
- VIDEO stop → just change `VIDEO_ID`.
- PHOTO stop → swap the YouTube card for an `<img>` (or a small slideshow of
  `<img>` tags) inside `#videoCard`; the proximity-trigger logic is identical.
  Photos can live in the repo (free on GitHub Pages — keep them web-sized).

**Reusable test anchor (Aaron's office):** to test new stops without traveling,
reuse a fixed spot near the office as the trigger location. Capture its
coordinate ONCE: open any stop in real mode, sit at the test spot, and read the
`you: <lat>, <lon>` line off the green diagnostic; paste it into the location
constant and use a generous `TRIGGER_RADIUS` (~20–30) while developing.
Office coordinate: **TBD** — read it off the diagnostic and we'll bake it in.

---

## AR.js config that WORKS on iPhone/Safari (don't change casually)

- A-Frame **1.3.0** + AR.js **3.4.5**.
- Use classic `gps-camera` + `gps-entity-place` — NOT the `gps-new-*` components.
- Do **NOT** set `videoTexture: true` — it kills all 3D rendering.
- `gps-camera="positionMinAccuracy: 1000"` (relaxed accuracy so it doesn't reject fixes).
- iOS needs a "Start" button that calls `DeviceOrientationEvent.requestPermission()`.
- Sign is drawn as ONE canvas image on a single flat plane (avoids text z-fighting).
- Helper components (registered in `<head>`): `smooth-heading` (compass jitter),
  `face-camera` (billboard). Sign height comes from the inner plane's local y.

## URL modes
- plain link → real GPS placement at the tower.
- `?preview` → pins the sign 12 m north, no GPS, for testing from home.
- `?calibrate` → on-site drop/nudge buttons + live coordinate readout.
- The diagnostic readout (green box, top of screen) shows after Start in real
  GPS modes: GPS accuracy, your coords, sign coords, distance, bearing, turn
  direction, and **in-world placement** (watch the `y` value — should be ~1.6).

---

## Git workflow lesson (don't repeat the divergence)

The Bell Tower coordinate fix once got "stuck" because the file was edited in two
places — on github.com from the phone AND locally — creating two diverged
histories that blocked the good commit from publishing.

**Rule:** edit in ONE place at a time, and **pull before you edit** if you've
touched it elsewhere. Workflow: Claude edits files locally → commit & push via
GitHub Desktop. If you must edit on the phone via github.com, pull those changes
into GitHub Desktop before editing locally again.

---

## Open items / TODO
- **Calibrate-mode live reposition:** the "Drop" button computes new coordinates
  but the sign doesn't visually move until reload. Live reposition in calibrate
  mode is buggy and worth fixing (low priority now that real mode works).
- DONE: Rowan Museum / Old Courthouse stop (rowan-museum.html) — built as a
  YouTube video pop-up (muted autoplay + Sound-on), trigger 15m. Confirmed
  working on iPhone. See "media pop-up" pattern above.
- Capture the office test-anchor coordinate (read `you:` off the diagnostic) so
  future stops (e.g. historic photos) can be tested from the office.
- Build additional stop(s) using the sign or media pattern above.

- **Create a pitch to the Rowan County Tourism Development Authority (TDA).**
  Business model is B2B managed service (NOT a consumer paywall): they pay an
  annual fee for hosting/maintenance + per-stop content; tour stays free to the
  public. Primary buyer = Rowan County TDA / Visit Rowan County (funded by the
  hotel room-occupancy tax; mandate is to promote & develop tourism assets).
  Likely co-sponsor/advocate = Downtown Salisbury Inc (Main Street nonprofit,
  owns the downtown footprint incl. Bell Tower Green / Old Courthouse).
  Pitch must be **aligned specifically to the TDA's published Tourism Master
  Plan and annual Program of Work** — quote their own objectives back to them and
  show how the AR tour advances each. Key framing to include:
    - One-time grant funds the build-out (NC Arts Council, NC Humanities,
      historic-preservation / local foundation grants); the ANNUAL fee covers
      maintenance + new stops.
    - Start with a small grant-funded pilot (Bell Tower + a few downtown stops),
      instrument QR scans / page views, and convert usage numbers into an
      ongoing contract.
    - Structure as a service, not a sale: license the platform annually, keep the
      code, bill content per stop. Service lapses if they stop paying.
    - Price on their budget / available occupancy-tax & grant money, not on cost
      (hosting is basically free). Don't name a number until scope is known.
    - Cautions: public-sector budget cycles are slow (time the ask to their
      fiscal year); get a lawyer + accountant for contract/licensing/IP; disclose
      any board conflicts (Rowan Museum, etc.).
  TDA contact: 204 East Innes St., Suite 280, Salisbury NC 28144 · (704) 638-3100.
  Deliverable when ready: a one-page pitch / pilot proposal aimed at the TDA.
