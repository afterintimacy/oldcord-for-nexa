# Changelog

All notable changes to Oldcord for Nexa are documented in this file.

## [1.2.0] — 2026-11-09 (published)

Published on `main` at commit `bf2734b`, annotated tag `v1.2.0`.
Tags `v1.0.0` / `v1.1.0` remain unchanged; no force-push, no history
rewrite.

### Added
- T1 — settings shell flattened to `--bg-base` (`#2F3136`): removes the
  `--canvas-glow` (13% of `--brand`) blurple wash on Settings.module's
  `.layer`; scoped to
  `[data-modal-layer]:has(> [class*="_navWrap_"], > [class*="_contentWrap_"])`
  only; every control keeps its 2020 token (1 rule, no `!important`).
- T2 — tooltip bubble: re-asserts `--tooltip-bg: #000 !important` on
  `[class*="_tip_"]`, because `Tooltip.tsx` writes the bubble colors
  inline per tooltip (measured `rgb(27, 25, 36)` → `rgb(0, 0, 0)`,
  white text, black arrow; 1 rule, 1 `!important` scoped to the bubble).

### Corrected premises (not implemented literally)
- T3 — emoji picker radius stays 8 px: the 2020 reference uses 8 px, so
  Phase 17's proposed 4 px was not implemented (TEST_REPORT §22–25).
- Phase 17 D-1 was a false positive (theme applied after `page.goto`);
  E-1's 4 px is unsupported by the reference.

### Deferred (out of Phase-18 scope)
- T4 — selected reaction pill solid blurple; T5 — settings nav bar 3 px;
  T6 — group headings uppercase.

### Verified (Phase 18 acceptance)
Installed through the real Custom CSS UI and measured via computed
styles at 1366×768 and 411×914: settings shell `rgb(47, 49, 55)` /
`background-image: none` on privacy/notifications/appearance/customCss
(including narrow list→page), tooltip `rgb(0, 0, 0)`/white/not clipped,
picker 8 px on all corners with working search/categories/scroll/cell
pick, reduced-motion persistence across reload, no regressions in member
list/search/context menu/profile/chat. Evidence:
`docs/oldcord-nexa/evidence/phase-18/`; TEST_REPORT §22–30.

CSS budget: +2 rules, +2,126 B (23,505 → 25,631 B — under the 64 KB
snippet and 1 MB link limits); braces balanced.

## [1.1.0] — 2026-11-09 (published)

### Added
- Profile color adaptation (Phase 15): the theme no longer overrides
  Nexa's per-user profile system. Un-themed profiles keep the classic
  OldCord surfaces (`#2F3136` shell, `#292B2F` interior sheet via the
  `--sheet-bg` token, 5 px radius from `--radius-lg`); two-color profiles
  show Nexa's own gradient (`--profile-top → --profile-bottom`,
  `data-themed=light|dark` foreground flip), single-accent profiles keep
  their banner color, banner images and avatar-derived tints are
  untouched, and server role colors remain exactly as defined by the
  server (they are inline-styled; nothing in the theme repaints them).
- Evidence: `docs/oldcord-nexa/evidence/phase-15/` in the worktree
  (v1.0.0 baseline vs v1.1.0 computed-style probes + screenshots for
  popout/modal/panel variants).

### Fixed
- V1.0.0's unscoped `background` on profile cards (`[class*="_card_"]
  [data-variant="popout|modal"]`, specificity (0,3,0) vs Nexa's
  `.card[data-themed]` (0,2,0)) flattened every user's two-color
  gradient and avatar tint to solid `#2F3136`; the forced sheet
  background flattened themed sheets' translucency. Both rules are now
  scoped with `:not([data-themed])` (card additionally
  `:not([data-tinted])`), so user customizations survive while default
  profiles look exactly as before.
- Removed the inert V1.0.0 `.banner` rule: the banner element always
  carries an inline style (image, `transparent`, or accent/fallback
  color) which beats any stylesheet rule — the forced `#202225` never
  rendered. Banner images and single accents are therefore safe *by
  construction*, and no stylesheet `background` rule may be added back
  without an `!important` (which would break single accents).
- README install URL now points at the real repository
  (`afterintimacy/oldcord-for-nexa/main/oldcord.css`) instead of a
  placeholder owner.

### Known limitation (CSS-only platform constraint)
- A profile with no custom colors and no banner keeps Nexa's
  deterministic per-user banner color (`avatarColor` palette). CSS
  cannot distinguish that inline fallback from a user's single chosen
  accent (identical DOM shape), so the theme deliberately preserves
  Nexa's banner rather than break single-accent profiles. A one-line
  Nexa-side attribute (`data-banner-accent`) would unlock it in a
  future theme revision; it is out of scope for a CSS-only theme.
  See `docs/IMPLEMENTATION.md` § Profile color adaptation.

### Verified
- Backend-backed profile matrix (scratch Go stack, seeded accounts):
  default / single-accent / two-color-dark / two-color-light ×
  popout/modal, plus the DM `panel` variant under two-color-dark.
  Computed styles prove gradients, `data-themed` foreground flips and
  the translucent themed sheets are intact under v1.1.0 and were
  flattened under v1.0.0.
- Roles runtime proof: two colored roles on a gradient profile —
  author name in the highest role's color, role dots exact, names
  readable on the themed card.

### Verified (Phase 15.1 — runtime acceptance, no CSS change)
- Banner images rendered (popout + modal): inline + computed
  `background-image`, integration with the avatar cut-out and shell.
- Avatar-derived tint: `data-tinted` + `--profile-tint` exactly
  `#1ea05a` (the fixture avatar's dominant color, computed at runtime
  by Nexa's own canvas path), radial glow over `#18191C`.
- Profile Designer live preview over real hex inputs + real Save
  flows: stage glow / banner thumb / preview card all track
  single/two-dark/two-light/banner/avatar-derived states.
- Public profile at `/@bob` (logged-out): themed card with banner
  image, translucent sheets, and a 411×914 one-column layout.
- Media pipeline caveat: the scratch backend 500s all image uploads;
  media bytes were served via Playwright route interception, fixtures
  documented in `docs/TEST_REPORT.md` (Phase 15.1 addendum).
- Classification: **PASS** (see TEST_REPORT §20).

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
  `docs/DELIVERY.md` for the Phase-12 run record; the dev-client
  runs recorded there are development evidence, not a platform
  support claim — see Phase 13.1). Supported platform for this
  release: **Nexa Web only**.
