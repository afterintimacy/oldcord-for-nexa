# Changelog

All notable changes to Oldcord for Nexa are documented in this file.

## [1.0.0] — 2026-11-08

### Added
- Initial standalone release: single-file theme `oldcord.css` (22,485 B)
  that ports OldCord v2.1's Discord-2020 dark palette onto Nexa's official
  Custom CSS system (`Settings → Custom CSS`).
- Design tokens: all 61 Nexa tokens remapped 1:1 from OldCord's shipped
  2020 values (background ladder #36393F/#2F3136/#292B2F/#202225/#18191C,
  blurple `hsl(227 58% 65%)` = #7288DA, github-dark code palette); layout
  geometry (rail 72 px, sidebar 240 px, header 48 px, radii 2–6 px).
- Component rules: server-rail circle→16 px hover/15 px selected morph with
  dark outer ring and flat indicator, amber mentioned rows, squared
  reaction pills, OldCord menu/button fills, composer focus, profile
  popout/modal surfaces, embed `#4F545C` bar, emoji picker radius 8 px.
- Starter docs: README (install/uninstall/update), IMPLEMENTATION,
  COMPATIBILITY, LICENSES, TEST_REPORT, DELIVERY. Evidence screenshots
  from acceptance runs under `screenshots/`.

### Fixed (from pre-release development)
- Replaced the Fase 1–7 draft's inert selectors (module-prefixed
  `[class*="Button_btn_"]`, `::deep()`, fantasy Discord class names,
  dead `[data-look="oldcord"]` scoping) with verified
  `[class*="_local_"]` fragments, attribute and role anchors, and
  `:has()` narrowing — every fragment proven to exist in the actual
  compiled CSS artifact of Nexa (25/25).
- Corrected the 2020 blurple documentation (OldCord ships
  `hsl(227 58% 65%)` = #7288DA, the 2020-era blurple, not #5865F2).

### Verified
- Backend-backed acceptance (scratch Go stack, seeded accounts,
  real channel interactions); theme activation through
  Settings → Custom CSS; persistence across reload; toggle off/on
  round-trip; zero theme console errors on the tested surfaces;
  `?safe` recovery path documented.
- Final classification at this release: PASS (see
  `docs/DELIVERY.md` for the Phase-12 run record; the Electron dev
  runs recorded there are development evidence, not a platform
  support claim — see Phase 13.1). Supported platform for this
  release: **Nexa Web only**.
