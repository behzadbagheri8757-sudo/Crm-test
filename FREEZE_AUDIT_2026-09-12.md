# Bagheri CRM — Freeze Candidate Re-Audit
Date: 2026-09-12

## Scope
Re-audit after the Shamsi Date Picker repair. Focus:
- Shamsi Date Picker implementation/presentation integration
- Invoice V6 regression surface
- JavaScript syntax
- local asset references
- Service Worker precache integrity
- CSS syntax/balance
- changed-file containment

## Shamsi Date Picker
### Confirmed
- Existing Jalali conversion/calculation and picker behavior in `js/ui.js` were not changed.
- Existing DOM generation and event handling were not changed.
- Added dedicated Shamsi Picker CSS in `css/visual-grammar-components.css`.
- Picker is mounted to `document.body`; CSS gives it `position: fixed` and `z-index: 90`, above the shared modal layer (`z-index: 60`) and Invoice V6's known higher modal layer (`z-index: 80`).
- Wheel layout, 36px item geometry, center highlight, touch scrolling, bottom-sheet geometry, RTL direction, and iOS safe-area handling are now explicitly defined.

## Invoice V6 regression containment
- `js/ui.js` unchanged.
- `js/views/invoice.js` unchanged.
- `js/views/invoices.js` unchanged.
- `css/app.css` unchanged.
- No Invoice V6 selector was modified by the Shamsi CSS; all new selectors are scoped to `.shamsi-*`.
- No business/data logic was changed.

## Additional confirmed issue found during re-audit
### Stale `icons.js` references
The candidate contained references to `./js/icons.js` in `index.html` and `sw.js`, but that file was absent from the repository. Source search showed the application does not depend on a global icon registry from that file; the actual `ICO` objects used by Dashboard/Game Center are locally defined.

Fix:
- Removed the stale `<script src="./js/icons.js">` from `index.html`.
- Removed `./js/icons.js` from SW precache.
- Bumped SW shell cache from v36 to v37 because static assets changed.

This prevents a known missing-asset request and an avoidable SW precache failure.

## Automated/static verification
- Project JavaScript syntax: PASS (all JS files).
- Service Worker syntax: PASS.
- CSS brace balance: PASS for all CSS files.
- HTML parsing: PASS.
- Local relative asset references: PASS; no missing referenced local assets.
- Service Worker precache: PASS; all listed precache assets exist.
- Shamsi conversion/date logic runtime checks: PASS for representative Jalali/Gregorian round-trips and month-length/leap cases.
- ZIP integrity: PASS.
- Changed files are limited to:
  - `index.html`
  - `sw.js`
  - `css/visual-grammar-components.css`

## Environment limitation
A real iPhone/Safari UI interaction test could not be executed in this environment because local browser navigation is blocked by the execution sandbox. Therefore real-device rendering/gesture behavior remains the only external validation step.

## Freeze verdict
**SOURCE-LEVEL READY TO FREEZE.**

No source-level blocker was found after the Shamsi repair and re-audit. The remaining validation is real-device Safari smoke testing, especially:
1. Invoice V6 → date field → picker opens above the invoice.
2. Select Jalali date → confirm → correct date returns.
3. Cancel leaves the original date unchanged.
4. Same picker behavior in transaction, check, purchase, and payment sheets.
5. Scroll year/month/day wheels on iPhone Safari without page-scroll interference.

If those five device checks pass, the candidate is suitable for freeze.
