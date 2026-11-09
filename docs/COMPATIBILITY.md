# Compatibility — OldCord for Nexa

**Phase 11 revision.** The scope decision changed in this phase; the
previous draft's `[data-look="oldcord"]` scoping was dead (no such
appearance id exists). The theme now applies via `[data-scheme="dark"]`
and plain token overrides, mirroring how Nexa's own documentation
expects a theme to be written.

---

## Target Platform

| Aspect | Status | Notes |
| --- | --- | --- |
| Web (browser) | ✅ Supported | Chrome 100+, Firefox 118+, Safari 16+, Edge — all features used (custom properties, color-mix, :has) are baseline since 2023 |

Other platforms are not supported and no compatibility is claimed or
verified for them. Mentions of other platforms under `docs/` are
historical development evidence — see TEST_REPORT — not compatibility
promises.

**Rule:** Oldcord for Nexa declares **Nexa Web** the supported and tested platform for this release. Custom CSS only works in the browser. Mobile users keep Nexa's default appearance.

---

## Nexa Version Compatibility

Targets `main` @ `3a2ceb95f007372cdcc70138afb247df6e204df6`
(origin `zeygoober/nexa`). Selectors follow Nexa's official ordering:

| Selector strategy | Stability |
| --- | --- |
| Custom properties (61 tokens) | Always — core contract |
| `data-scheme`, `role=`, `data-*` attributes | Always — part of the app's public contract |
| `.md-*` rendered-markdown classes | Always — Nexa documents they never change |
| `[class*="_local_"]` hashed-prefix | Until a CSS Modules local is renamed or the module is renamed; re-audit after every major release and re-run the audit script |

Keep an eye on: `.row` (sidebar channel row), `.btn` (rail + Button),
`.pill`, `.card`, `.item`, `.pop`, `.box`, `.ta` — high-traffic local
names shared by several modules. Every use in this theme is scoped by
an ancestry module class (`[class*="_rail_"]`, `[class*="_sidebar_"]`,
`[class*="_menu_"]`, `[class*="_reactions_"]`, `[class*="_embeds_"]`),
a role/data anchor (`[role="toolbar"]`, `[role="article"][data-mentioned]`,
`[data-variant="popout"]`, `[data-selected]`, `[data-me="true"]`), or
a `:has()` of distinctive descendants (`[class*="_box_"]:has(_attach_):has(_btns_)`,
`[class*="_pop_"]:has([class*="_grid_"])`) to prevent cross-module bleed.

---

## Appearance Preset Compatibility

**The theme is dark-only.** Under `[data-scheme="dark"]` it overrides
all 61 tokens; on a Light preset (`data-scheme="light"`), every rule
goes idle.

| Preset | Behaviour |
| --- | --- |
| Dark | ✅ Full OldCord look |
| Any dark preset (Forest Dark, Midnight, Ash preset, etc.) | ✅ Same overrides; presets' own accents are overwritten by blurple |
| Light family | ⏸️ Inert — stock Nexa Light is shown |
| Guest call page (`data-dark-scope`) in a Light preset | Keeps Nexa's own dark tokens internally; visually unchanged from stock, tuned by the Light preset's CSS, not by this theme |

**Recommendation:** choose the **Dark** preset (or any dark variant)
in Settings → Appearance; the theme handles the rest.

---

## Conflict with Other Themes

If several Themes-from-Links entries are enabled, the last loaded wins
on equal specificity. The theme's component rules read Nexa tokens, so
an accent change in another link recolors only what tokens drive;
geometry rules (rail morph, radii, header height) may interact.
This theme is designed to be loaded alone or last.

---

## Safe Mode

Browsing with `?safe` at the end of the address disables Custom CSS
and plugins for that tab — the app stays usable even with a broken
theme, and settings can then be reverted (Settings → Custom CSS →
toggle or delete the entry).

---

## Dependency Audit

| Dependency | Level | Notes |
| --- | --- | --- |
| Google Fonts `@import` (Atkinson Hyperlegible) | Optional | Single request at first paint; remove `oldcord.css` line 1 if offline access only or to eliminate third-party requests entirely (`system-ui` fallback is indistinguishable in practice for prose) |
| Everything else | **Self-contained** | No images, no data URIs, no external CSS; all Discord-origin assets were stripped at source |
