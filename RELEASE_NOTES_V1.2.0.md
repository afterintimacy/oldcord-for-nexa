# OldCord for Nexa — v1.2.0 release notes

**Status:** staged on `work/v1.2.0-implementation` — **not published**.
Tags `v1.0.0` / `v1.1.0` are untouched; no force-push; no tag created.
Do not claim v1.2.0 as released until this is explicitly promoted.

## What changed

V1.1.0 established the Discord-2020 look on Nexa's Custom CSS system
and adapted profile colors to Nexa's per-user customization. V1.2.0 is
a minimal, targeted refinement: exactly two component rules (no token
changes, no selectors outside settings/tooltip/picker), under the
Phase-18 CSS budget (+2 rules, +2,126 B → 25,631 B total).

### T1 — settings shell (flatten the layer)

With the theme actually installed via Settings → Custom CSS, every
settings control already draws from the 2020 tokens — switches use the
blurple `--grad-brand` (`#7288DA`), radios/sliders/selection use
`--brand`, hover uses `--bg-hover`, text uses `--text`/`--text-muted`,
and primary buttons are flat blurple. The one real residue was the
shell itself: Settings.module's `.layer` paints a
`radial-gradient(... var(--canvas-glow) ...)` over `--bg-canvas`, and
`--canvas-glow` is 13% of `--brand` — a blurple wash flat 2020 settings
never had. V1.2.0 flattens it to `var(--bg-base)` (`#2F3136`),
scoped to the settings layer only:

```css
[data-scheme="dark"] [data-modal-layer]:has(> [class*="_navWrap_"],
> [class*="_contentWrap_"]) {
  background: var(--bg-base);
}
```

### T2 — tooltip bubble (black, like 2020)

Nexa's `Tooltip.tsx` computes bubble colors per tooltip at runtime and
writes them as inline custom properties on the bubble, so an ordinary
token rule could never win (measured `rgb(27, 25, 36)` / `#1b1924`
under V1.1.0). V1.2.0 re-asserts the token's value on the bubble:

```css
[data-scheme="dark"] [class*="_tip_"] { --tooltip-bg: #000 !important; }
```

The component's own `background: var(--tooltip-bg)` and its arrow's
`border-*-color: var(--tooltip-bg)` then resolve to #000, while the
JS-picked foreground stays white and readable — measured
`rgb(0, 0, 0)` with white text and a black arrow, not clipped, at both
reference viewports.

### T3 — emoji picker radius (verified, no change)

OldCord's 2020 reference uses 8 px on the emoji picker
(`--custom-emoji-picker-border-radius: 8px`, and 8 px on the
expression-picker wrapper), so V1.1.0's existing 8 px rule on
`[class*="_pop_"]:has([class*="_grid_"])` is correct as-is; the Phase-17
proposal of 4 px was not implemented and is not implemented here.

## What V1.2.0 deliberately does not change

- Member list, search, notifications, profiles, chat layout, reactions,
  server nav, composer, backend, auth, persistence — out of scope; no
  rules touch them (verified regression spot-checks pass).
- T4 (selected reaction pill solid blurple), T5 (settings nav bar 3 px),
  T6 (uppercase group headings) are deferred to a later revision.
- V1.2.0 remains dark-only, like V1.1.0; the install URL is unchanged
  (`https://raw.githubusercontent.com/afterintimacy/oldcord-for-nexa/main/oldcord.css`)
  and points at the published V1.1.0 file until v1.2.0 is promoted.

## Verification

Phase 18 acceptance was run against the real scratch backend through
Nexa's real Custom CSS UI (snippet pasted and saved via the settings
editor), measuring computed styles at 1366×768 and 411×914:

- Settings shell flattened to `rgb(47, 49, 55)` /
  `background-image: none` on privacy/notifications/appearance/customCss,
  including the narrow list→page swap; controls and tokens unchanged.
- Tooltip `rgb(0, 0, 0)` / white text / black arrow, positioned sanely,
  not clipped, dismissed on mouse-out/Escape as before.
- Picker 8 px on all corners, 498×498 (411×548 narrow, not clipped),
  working search ("smile" → 26 results), category nav, scroll, and cell
  pick inserting the emoji into the composer.
- Persistence: reduced-motion toggle survives reload on and off.
- Regression spot-checks (context menu, search panel, profile modal,
  member list, chat) unchanged vs the Phase-17 baseline.
- Zero theme-caused console errors; one pre-existing, theme-unrelated
  404 (update-check endpoint on the scratch server).

Full evidence: `docs/oldcord-nexa/evidence/phase-18/`;
`docs/TEST_REPORT.md` §22–30; `docs/IMPLEMENTATION.md`.

## Install

Same as V1.1.0: **Settings → Custom CSS → Themes from Links** (paste the
URL) or **Snippet** (paste the file contents) → Save → pick the Dark
preset in Settings → Appearance. The published file remains V1.1.0
until v1.2.0 is promoted; a staged build lives at
`work/v1.2.0-implementation` in this repository.

## License

MIT — see `LICENSE`. Based on milbits/oldcord (MIT); the OldCord/Discord
logos and third-party artwork in the original were **not** redistributed
and are documented in `docs/LICENSES.md`.
