# Oldcord for Nexa v1.0.0 — Release Notes

**Discord, as it looked in 2020 — on Nexa.** One CSS file ports OldCord's
Discord-2020 look onto Nexa's official Custom CSS system.

## Install
1. Host `oldcord.css` over HTTPS (or copy it locally for pasting).
2. Nexa → **Settings → Custom CSS → Themes from Links → Add a Link**,
   or paste the file's contents into the **Snippet** editor and save.
3. In **Settings → Appearance** use the **Dark** preset.
   (OldCord is dark-only; under a Light preset the stylesheet stays inert.)

## What changed in this release
- Full 2020 palette: #36393F chat area, #2F3136 sidebar, #292B2F cards,
  #202225 rail, #18191C floating surfaces, blurple `#7288DA`
  (`hsl(227 58% 65%)`), channel names `#8E9297`, github-dark code colors.
- 2020 geometry: rail 72 px, sidebar 240 px, header 48 px, corner radii
  2–6 px, token-driven shadow flattening.
- Server tiles: circle at rest, 16 px hover / 15 px selected rounded
  morph, flat blurple fill and flat white edge indicator.
- Amber mentioned rows, `brand-experiment-560` menu hover, flat buttons,
  `#4F545C` embed bar, profile card surfaces, squared reaction pills,
  8 px picker, `@import` for optional Atkinson Hyperlegible (deletable).
- Every selector is verified against the actual build artifact of Nexa
  (fragment audit + compiled-CSS verification); no Discord hashes,
  Vencord hacks, or external assets are shipped.

## Verified with
- Nexa `zeygoober/nexa` `main` @ `3a2ceb95f007372cdcc70138afb247df6e204df6`
- Scratch Go backend 1.27.3 (`chatd` built and seeded per
  `tools/seed.py` pattern) and Chromium (Playwright)
- Some runs also used a Nexa Desktop **development** build
  (Electron 44.9.3) purely as a dev-quality environment; it is not a
  release platform and no compatibility is claimed for it.
- Zero theme-caused console errors; repo untouched.

## Known limitations
- Dark-only (Light preset stays stock).
- Some modern Discord surfaces have no 2020 equivalent and keep Nexa's
  own rendering (threads, polls, voice stage quirks).
- Whitney is not bundled; Atkinson Hyperlegible + system-ui is the
  documented substitute.
- Theme follows accounts through Custom CSS sync; Android does not
  run Custom CSS (Nexa-side constraint).

See `CHANGELOG.md` for the fuller history and `docs/` for the audit,
implementation notes and test evidence.
