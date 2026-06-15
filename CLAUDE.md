# Memory — AR Walking Tour of Salisbury NC

Persistent context for any future session working in this project. See
PROJECT-NOTES.md for full detail.

## About the user / working style
- Aaron (ackepley@gmail.com). Coding beginner — explain every terminal command
  and what could go wrong. Build ONE feature at a time and wait for confirmation
  it works on his iPhone/Safari before moving on. Free tools only (A-Frame,
  AR.js, GitHub Pages, YouTube). Commit to Git after every working feature.
- Workflow: Claude edits files locally → Aaron commits/pushes via GitHub Desktop
  (he authenticates GitHub via Google passkey; command-line git push needs a
  Personal Access Token, so prefer GitHub Desktop for pushing).
- My environment is READ-ONLY on the repo's `.git` folder — I can read git state
  but cannot run commits/pushes/locks. All git actions happen on Aaron's Mac.

## Project = AR walking tour of Salisbury, NC
- Live: https://ackepley.github.io/AR-Walking-Tour-of-Salisbury-NC/
- Proven AR.js stack: A-Frame 1.3.0 + AR.js 3.4.5, classic `gps-camera` +
  `gps-entity-place`, NO `videoTexture` (kills 3D), `positionMinAccuracy: 1000`,
  iOS Start button calling `DeviceOrientationEvent.requestPermission()`.

## Key technical lessons
- Altitude gotcha: AR.js gps-entity-place subtracts GPS altitude from the
  entity's y ONLY when y != 0. Salisbury ~233 m ASL buried signs ~236 m down.
  FIX: anchor entity at `position="0 0 0"` (skips altitude math) and put the
  eye-level height on the inner `<a-plane position="0 1.6 0">`. Elevation-proof.
  (Old per-frame `fixed-height` override was removed — it never attached.)
- Custom components (face-camera, smooth-heading) MUST be registered in a
  <script> in <head> BEFORE <a-scene>, else they silently don't attach
  (symptom: diagnostic `billboard: OFF`).
- New signs: own lat/lon + entity `position="0 0 0"` + inner plane at `0 1.6 0`.
- Media stops (video/photos): NOT a 3D object. A YouTube iframe can't be a WebGL
  texture, so media shows as an HTML card over the camera feed, auto-opened by a
  GPS proximity check. iOS blocks autoplay-with-sound → autoplay MUTED + a
  "Sound on" tap to unmute. Knobs: `MUSEUM` (location), `VIDEO_ID`,
  `TRIGGER_RADIUS`. `?preview` opens it instantly for off-site testing. Full
  recipe in PROJECT-NOTES.md "media pop-up" section.

## Status (June 2026)
- Bell Tower sign: coordinates correct (35.668851, -80.472087); altitude bug
  fixed via y=0 anchor + inner-plane height; custom components moved to <head>
  so the billboard attaches. All diagnostics green from the office (y 0.0,
  panel 1.6, billboard on). PENDING: final on-site visual check at the tower.
- Open: calibrate-mode "Drop" doesn't visually reposition the sign live (bug).
- Rowan Museum / Old Courthouse (rowan-museum.html): DONE. Built as a YouTube
  video pop-up (coords 35.668582, -80.468758; muted autoplay + Sound-on button;
  proximity trigger 15m). Confirmed working on iPhone.
- TODO: capture the office test-anchor lat/lon (read `you:` off the diagnostic)
  so future stops like historic photos can be tested from the office.
- Planned business step: pitch the tour to the Rowan County TDA as a B2B managed
  service (annual maintenance + content fee, free to public, NO consumer
  paywall). Pitch must align to the TDA's Tourism Master Plan & Program of Work;
  grant-funded pilot → analytics → annual contract. Full detail in
  PROJECT-NOTES.md "Create a pitch" item.
