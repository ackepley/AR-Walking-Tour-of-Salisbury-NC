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

## Key technical lesson (the altitude gotcha)
- AR.js places objects by GPS *altitude*. Salisbury is ~233 m above sea level,
  so signs with no altitude get buried ~236 m underground (diagnostic showed
  `y -236.0`). Fix = `fixed-height` component locking the entity's y to eye
  level every frame (`fixed-height="y: 1.6"`). It's relative to the phone, so it
  works at every elevation across Salisbury's ridge — no per-location tuning.
- New signs only need their own lat/lon + `fixed-height`.

## Status (June 2026)
- Bell Tower sign: coordinates correct (35.668851, -80.472087); altitude bug
  fixed via `fixed-height`. Awaiting Aaron's on-site confirmation next day.
- Open: calibrate-mode "Drop" doesn't visually reposition the sign live (bug).
- Next stop planned: Rowan Museum / Old Courthouse.
- Planned business step: pitch the tour to the Rowan County TDA as a B2B managed
  service (annual maintenance + content fee, free to public, NO consumer
  paywall). Pitch must align to the TDA's Tourism Master Plan & Program of Work;
  grant-funded pilot → analytics → annual contract. Full detail in
  PROJECT-NOTES.md "Create a pitch" item.
