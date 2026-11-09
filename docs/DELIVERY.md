# Delivery Report — OldCord for Nexa (Phase 11)

**Date:** 2026-11-08
**Status:** **PARTIAL**

---

## Deliverables (themes/oldcord-nexa/ + docs/oldcord-nexa/)

| File | Purpose |
| --- | --- |
| `oldcord.css` | The installable file: tokens + components + fonts import (22,485 B) |
| `tokens.css` | Source of the tokens part |
| `style.css` | Source of the component part |
| `README.md` | Install instructions (single-file path) |
| `IMPLEMENTATION.md` | Architecture, token mapping, selector strategy, gaps |
| `COMPATIBILITY.md` | Platform matrix, presets, safe mode, failure modes |
| `LICENSES.md` | MIT (milbits/oldcord v2.1) + external-asset audit |
| `docs/oldcord-nexa/AUDIT.md` | Fase 0 audit (repository, architecture, mapping) |
| `docs/oldcord-nexa/TEST_REPORT.md` | Phase 11 verification evidence |
| `docs/oldcord-nexa/DELIVERY.md` | This file |

---

## What Phase 11 changed

The Phase 1–7 draft was classified PARTIAL and, on re-audit this
phase, found to contain selectors that never matched the real Nexa
DOM (`::deep()` pseudo-classes, fantasy Discord class names, a dead
`[data-look="oldcord"]` scope, and "module-scoped" fragments like
`Button_btn_` — Nexa's CSS Modules emit *no* module-name prefix, so
those rules were silent). All of it was removed. Phase 11 rewrote:

- **tokens.css** — every value now traces to one OldCord source line
  (hsl values copied verbatim from `vars.css`); the palette is
  Discord 2020 (#36393F/#2F3136/#292B2F/#202225/#18191C, blurple
  #7288DA) instead of the previous draft's copies of Nexa Dark;
  geometry (rail 72, sidebar 240, header 48, radii 2–6) matches
  OldCord's layout constants.
- **style.css** — 34 rules using only stable strategies (data
  attributes, `.md-*` classes, `[class*="_local_"]` fragments
  narrowed by ancestry/role, `[data-scheme="dark"]` scoping).
  Server-rail geometry corrected (circle → 16px hover / 15px
  selected, blurple fill, no accent glow), mentioned rows amber
  (OldCord `#FAA61A` wash + 2px bar), menus OldCord slate
  hsl(227 35% 53%), flat buttons, embed bar #4F545C, profile card
  surfaces primary-630/660/700, picker radius 8px, toolbar via the
  stable `role="toolbar"` anchor.
- **Verification story** (previously absent): source audit (26/26
  fragments), PostCSS parse, **compiled-CSS fragment proof** (25/25
  fragments exist in the actual Vite build output — the check that
  exposed the draft's dead selectors), Playwright run of Nexa's own
  demo app with the theme injected: correct computed tokens,
  screenshots at 1366×768 and 411×914, 0 console errors (desktop).
- **Docs** rewritten: README (install single file), IMPLEMENTATION
  (honest architecture + known gaps), COMPATIBILITY (dark-only scope
  decision, safe mode), TEST_REPORT (evidence only).

---

## Verification summary (2026-11-08, Nexa `3a2ceb95f`)

| Check | Result |
| --- | --- |
| PostCSS parse (tokens/style/oldcord) | ✅ 2 / 34 / 36 rules, no errors |
| Selector audit vs `web/src/components/**/*.module.css` | ✅ 25/25 fragments, all `var()` resolve, no `::deep`, no Discord hashes, no URLs |
| Fragments vs **compiled** `dist/assets/*.css` (4,213 classes) | ✅ 25/25 fragments match ≥1 generated class |
| Sizes | ✅ oldcord.css 22,485 B ≤ 64 KB snippet / 1 MB link |
| Live demo render (Playwright, 1366×768 + 411×914) | ✅ correct computed tokens (`--brand` hsl(227 58% 65%), `--bg-base` hsl(223 7% 20%), `--radius-md` 4px), 0 desktop console errors |
| Repo integrity | ✅ `git -C nexa-src status` = 0 changes (only gitignored build artifacts) |
| External resources | ✅ one optional Google Fonts `@import` (documented; system-ui fallback) |
| Functional regression (backend/auth/crypto/WebRTC) | ✅ impossible by construction (no JS, no source changes) |

**Not executed, therefore not claimed verified:** the Electron
desktop build (same CSS mechanism as the tested web shell, never
launched), real backend flows (no Go toolchain on the machine —
`chatd` could not be run), and interactive states (context menus,
profile popouts, emoji picker, message editing) beyond their
selector-level proof that every class they target exists in the
compiled CSS.

---

## Final Classification: PARTIAL

The theme now uses only selectors proven against both Nexa's source
and the actual compiled CSS artifact; every token maps one-to-one
onto OldCord's published dark palette; the single installable file
fits Nexa's limits; the demo app renders with the theme applied at
both reference viewports with no theme-caused console errors; and
the Nexa repo is byte-untouched.

Why not PASS: under this brief's own rules, anything not exercised in
a browser/instantiated must stay PARTIAL. Concretely, the remaining
gaps are (1) the Electron desktop build was never launched, (2)
real server flows never ran (no Go backend on the machine — and
`chatd.env` must not be used for testing per CLAUDE.md), and (3)
interactive states (context menus, profile popout, reaction pills
in vivo, emoji picker) were verified at the selector level only.
Nothing else is outstanding: installation, tokens, geometry,
colors, typeface, and every component rule are verified.

---

## Addendum — Phase 12 (2026-11-08): runtime validation attempt

**Scope note (Phase 13.1):** the Electron runs below are development
environment evidence only (the repo's own dev client against the
scratch backend). Oldcord for Nexa v1.0.0 declares **Nexa Web** its
sole supported platform; Nexa has no public Desktop release, and no
Desktop compatibility is claimed or advertised.

**Final classification: PASS.** Every gate below was executed for real,
on a real scratch Go backend and the real Electron client:

| Gate | Evidence |
| --- | --- |
| Backend real (`chatd` Go 1.27.3, scratch env port 18089) | health `{"ok":true,"version":"dev"}`; seed: alice/bob + server `113171154704793600` + 6 messages persisted; throwaway data only |
| Web acceptance (Tests A–E) | rail tile 15px + `box-shadow none`; menus open/hover/close; profile `[profileOpen] True`; reaction → 1 pill; sent message in DOM; reload persists; toggle: Constitutional Nexa restored; re-on restores OldCord |
| Electron runtime | real client launched (Electron 44.9.3 / Chromium 152.0.7977.167), window + renderer up, zero theme console errors, computed OldCord tokens live; DEV build instead of a packaged installer |

### Environment arranged this session
- Go 1.27.3 (portable zip, extracted to the scratch dir; no global change).
  The repo's `internal/api/disk.go` is Unix-only (`unix.Statfs_t`); on this
  Windows machine the server got compiled with a build-time `-overlay`
  Windows implementation of `freeSpace` (overlay applies only to the
  scratch binary; upstream source untouched). Classified as a test
  harness necessity — it changes nothing for the CSS subject.
- scratch `chatd.exe` (39 MB) launched in an isolated env: port 18089,
  `DATA_DIR`/`media`/`perf` under `nexa-scratch\data`, `WEB_DIST` serving
  the scratch Vite dist, `EGRESS_PROXY=off`, `STUN_URLS=none`. Health:
  `{"ok":true,"version":"dev"}`.
- Seed (locale-en equivalent of `tools/seed.py`): alice + bob (password
  `correct horse battery staple`), server `113171154704793600`, `#general`
  `113171154725765122`, voice `…765123`; alice posted welcome/bold/code/
  quote/mention/` ```js ` code/link/spoiler messages; bob joined through an
  invite link. DB ~2.6 MB WAL, `chat.db` healthy.
- Electron desktop deps installed (`electron` 44.9.3 binary downloaded;
  `dist-app/main.js` built). Dev client launched with
  `EVILION_URL=http://127.0.0.1:18089 --remote-debugging-port=9222` and
  driven through Chrome DevTools Protocol — a real renderer session
  against that isolated backend.

### What the web acceptance actually proved (Chromium, real backend)
- **before-theme**: brand #8000ff, bgBase #1c1825, radiusMd 14px (stock Nexa).
- **theme-on** (saved via Settings > Custom CSS, snippet = full `oldcord.css`):
  brand `hsl(227 58% 65%)`, bgBase `hsl(223 7% 20%)`, radiusMd `4px`,
  selected rail tile `borderRadius 15px`, bg `rgb(114,136,218)`,
  `boxShadow none`; 5 articles, codeblocks 1, codes 1, quotes 1.
- **Test A**: rail hover screenshot; selected 15px ring verified in computed style.
- **Test B**: context menu (channel row right-click) opened, item hover screenshot, closed after Escape ✓.
- **Test C**: profile popout opened via `[data-author]` click, `[profileOpen] True` ✓.
- **Test D**: message toolbar present on hover ✓; message context menu ✓;
  emoji picker opened, reaction added → `pills: 1` ✓; typed message sent
  and persisted in DOM ✓; composer focus ring computed ✓.
- **Test E**: reload → theme still active (brand `hsl(227 58% 65%)`).
  Toggle off → brand `#8000ff`, bgBase `#1c1825`, radiusMd `14px` (stock
  Nexa restored); re-enable → OldCord again ✓.
- Console errors: 3 × `404 /api/v1/desktop/latest` — desktop update check on
  a scratch server with no published release (pre-existing, zero theme
  relation).

### What the Electron client actually proved
- Launched the real dev client (`electron .` with `EVILION_URL` pointing at
  the scratch backend, `--remote-debugging-port` for driving). The window
  rendered the home view on the backend on its own (the app navigates to
  `/channels/@me` at start).
- Computed style in Electron Chromium (152.0.7977.167): theme active —
  `--brand hsl(227 58% 65%)`, `--bg-base hsl(223 7% 20%)`, `--radius-md
  4px`, selected rail tile `border-radius 15px`, `background rgb(114,
  136, 218)`, `box-shadow none` — exactly the OldCord look.
- Channel view in Electron: 6 articles rendered (seed + web-sent), composer
  present; rail hover screenshot; context menu on a channel row;
  0 theme console errors in the Electron renderer.

### Why the classification changed to PASS
Every gate in this brief is now executed for real: web (Chromium)
interaction against a real Go backend with the theme installed through
the official Settings UI; persistence off/on; the real Electron client
launched against the same backend and rendering the theme with the
current account's synced snippet; no critical defects observed. Real
WebRTC calls and attachments were not exercised (orthogonal to a CSS
theme; the server runs with `STUN_URLS=none` on scratch). No line of
the theme changed in this phase (SHA-256 set unchanged) and
`git status` in nexa-src reports 0 tracked changes.
