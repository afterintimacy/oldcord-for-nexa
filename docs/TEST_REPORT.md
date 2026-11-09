# Test Report — OldCord for Nexa (Phase 11)

**Date:** 2026-11-08
**Target:** Nexa `main` @ `3a2ceb95f` (`zeygoober/nexa`)
**Theme revision:** `themes/oldcord-nexa/` (Phase 11)

This report covers only the Phase 11 rewrite and only what was
actually executed. The result is honest about what was not exercised
(§7).

---

## What was checked

| # | Check | Method | Result |
| --- | --- | --- | --- |
| 1 | CSS syntax | PostCSS (project toolchain) parse of all three files | ✅ OK |
| 2 | Selector audit vs source modules | `oc-audit.py`: every `[class*="_local_"]` fragment resolved against `web/src/components/**/*.module.css` locals; every `var(--…)` reference resolved to Nexa's `styles/*.css` or the theme itself; no Discord-hashed classes; no `::deep(`; no `url()`/`http(s)` | ✅ 25 fragments OK, 0 errors, 2 MD classes OK |
| 3 | Selector audit vs **compiled** CSS | `oc-fragments.py`: same fragments resolved against the actually built `dist/assets/*.css` (4,213 hashed classes) | ✅ 25/25 fragments match ≥1 generated class, 0 failures |
| 4 | Live render with the theme applied | Playwright + the built app's `demo.html` (full fake API; no Go backend required) at 1366×768 and 411×914; theme injected as Nexa's own loader does | ✅ tokens computed correctly, screenshots captured, 0 desktop console errors |
| 5 | Repository integrity | `git -C nexa-src status` | ✅ 0 changes (only gitignored `web/node_modules` + scratch dist) |

Finding during check 3 (fixed before delivery): the intermediate
draft used *module-name-prefixed* fragments (`Button_btn_`,
`ContextMenu_menu_`, `Composer_box_`, `MemberList_list_`,
`Picker_pop_`). The **actually emitted** class names are
`_local_<hash>` with **no module prefix** (e.g. `_rail_1143b_339`,
`_btn_q2lzi_111`), so those selectors matched zero generated classes —
the compiled-CSS audit caught what the source audit could not. Every
such rule was rewritten to `[class*="_local_"]` with ancestry/attribute
narrowing and re-verified with check 3 plus a live render.

---

## 1. Syntax

```
$ node -e "require('postcss').parse(fs.readFileSync(...))"
tokens.css: OK, 2 rules    ( :root + [data-scheme="dark"] )
style.css:  OK, 34 rules
oldcord.css: OK, 36 rules   ( @import + tokens + style )
```

## 2. Source-level audit

```
$ python <scratch>\\oc-audit.py

oc-* internal vars: 29; official tokens set: 72
FRAG OK  '_add_' '-'_banner_' '_btn_' '_card_' '_catHead_' '_danger_'
'_embeds_' '_hasIcon_' '_icon_' '_item_' '_pill_' '_positive_' '_primary_'
'_rail_' '_reactions_' '_row_' '_secondary_' '_sheet_' '_sidebar_'
'_ta_' '_toolbar_' … (= 25 fragments matched to real modules; see script output)

PASSED: 25 fragments OK; all var() refs resolve; no forbidden patterns
```

(The earlier prose-only failure `_local_` / `.menu_M3qk4j` existed
only because the first version grepped comments; the audit now strips
comments before extraction.)

## 3. Compiled-CSS verification (the decisive check)

```
$ python <scratch>\\oc-fragments.py

OK   '_add_'                -> 2 generated classes
OK   '_attach_'             -> 1 generated classes
OK   '_banner_'             -> 7 generated classes
OK   '_box_'                -> 8 generated classes
OK   '_btn_'                -> 3 generated classes
OK   '_btns_'               -> 5 generated classes
OK   '_card_'               -> 26 generated classes
OK   '_catHead_'            -> 1 generated classes
OK   '_danger_'             -> 7 generated classes
OK   '_embeds_'             -> 2 generated classes
OK   '_grid_'               -> 16 generated classes
OK   '_hasIcon_'            -> 1 generated classes
OK   '_icon_'               -> 14 generated classes
OK   '_item_'               -> 11 generated classes
OK   '_menu_'               -> 2 generated classes
OK   '_pill_'               -> 6 generated classes
OK   '_pop_'                -> 3 generated classes
OK   '_positive_'           -> 1 generated classes
OK   '_primary_'            -> 5 generated classes
OK   '_rail_'               -> 5 generated classes
OK   '_reactions_'          -> 1 generated classes
OK   '_row_'                -> 33 generated classes
OK   '_secondary_'          -> 3 generated classes
OK   '_sheet_'              -> 7 generated classes
OK   '_sidebar_'            -> 3 generated classes

fragments checked: 25, failures: 0
```

Findings while writing this verification (defects that would have
shipped otherwise):

1. **The draft's "module-scoped" fragments were dead.** The compiled
   CSS has *no* module-name prefix; `_btn_` etc. match only
   `_<local>_<hash>`. `Button_btn_`, `ContextMenu_menu_`,
   `Composer_box_`, `MemberList_list_`, `Picker_pop_` matched **zero**
   generated classes — the palette survived only because it was
   token-driven; the entire component layer was silently inert.
2. **`_local_` in comments** fooled the naive regex of the first
   run; the audit now strips comments.
3. **`_hasIcon_` audit false-positive** was the audit's regex (it
   ignored compound `.btn.hasIcon` selectors); the class exists
   (`Rail.tsx` lines 297/437/463 apply `s.hasIcon`).

## 4. Live render (built app, demo realm, no backend)

`demo.html` boots `DemoApp` — the Nexa web app with a complete
in-browser fake API (auth, E2E crypto, messages, reactions, pins,
threads, servers, channels, roles; `src/demo/runtime.ts`). The theme
CSS was injected exactly as Nexa's own loader does it (a `<style>`
element at the end of `<body>`; `web/src/lib/look/styles.ts`), with
`document.documentElement.dataset.scheme='dark'` (which a real dark
preset sets):

```
$ python <scratch>\\oc-visual.py

built css: 4213 hashed classes
  FRAG OK   '_add_' ... -> 2 generated classes   (… 25 fragment lines, all OK …)
[desktop] computed: {'scheme': 'dark', 'brand': 'hsl(227 58% 65%)' (blurple),
                     'bgBase': 'hsl(223 7% 20%)' (#2F3136),
                     'radiusMd': '4px', 'title': 'Nexa'}
[desktop] console errors: 0
[desktop] screenshot: ...\oc-shots\desktop.png
[phone]   computed: same token values (dark scheme applied)
[phone]   console errors: 3   (below)
[phone]   screenshot: ...\oc-shots\phone.png

ALL FRAGMENTS MATCH BUILT CSS
```

Phone console errors (all three, verbatim):
`net::ERR_CONNECTION_REFUSED` ×3 — the demo page attempted backend
endpoints (no Go server running). Zero theme-related errors.

Observed on the screenshots (in `<scratch>\\oc-shots\`):
dark 2020-grey surfaces, blurple accents (Sign in, rail selection
ring), flat square corners, whitespace-consistent chrome.

### Interactive pass

A deeper Playwright pass created a server through the rail ("add
server" flow), asserted `railButtons: 3` and box-inside-page
selectors existed, and saw the server tile + channels appear
("General", "Lobby" channels; server dropdown menu rendered). The
demo flow did not reach a live text channel with a composer in this
sandbox (the created server's default navigation lands on the DM
view), so the composer-context-menu/profile-popout screenshots were
not produced. Those rules stand on their fragment proof (§3) and the
source reading, not on screenshots. CSS-only themes cannot alter
app logic, so nothing exercised above can regress sending, auth,
encryption, calls, search, or persistence.

---

## Phase 12 addendum (real scratch backend + Electron)

**Scope note (Phase 13.1):** Electron evidence in this section is a
dev-environment record (repo dev client against the scratch backend).
The v1.0.0 support promise covers **Nexa Web only**; see
`docs/COMPATIBILITY.md`.

Environment (this machine, 2026-11-08):
- Git: `nexa-src` @ `3a2ceb9` (origin `zeygoober/nexa`), `git status` clean (0 tracked changes).
- Go: `go1.27.3 windows/amd64` (matches `go.mod` requirement 1.27.3);
  installed portable from the official zip, extracted to the scratch
  dir (no global install). Used only to compile the throwaway `chatd`.
- Node v24.18.0 / Python 3.14.6 / Playwright 1.64.1 (Chromium 1248).
- Electron 44.9.3 devDependency, `dist-app/main.js` built; binary ran
  and reported `Electron/44.9.3`, `Chrome/152.0.7977.167`.
- Scratch chatd (Go 1.27.3 build; one build-time `-overlay` stub for
  `internal/api/disk.go` — Windows free-space syscall — touching only
  the scratch binary, never the repo; `CLAUDE.md` forbids dev
  deployments that touch the live server/data). Env: `LISTEN`,
  `PUBLIC_URL`, `DATA_DIR`, `WEB_DIST`, `EGRESS_PROXY=off`,
  `STUN_URLS=none` — throwaway data under the scratch dir only.
- Seed (equivalent of `tools/seed.py`, locale-en): alice+bob,
  server `113171154704793600`, `#general` `113171154725765122`,
  voice `…765123`; alice posted 6 seeded messages
  (welcome + bold/code/quote/mention + ```js code + link/spoiler).

### Tests A–E (Chromium web, real backend, theme via real Settings UI)

```
- before-theme: brand #8000ff, bgBase #1c1825, radiusMd 14px (stock Nexa)
- theme-on (saved via Settings > Custom CSS, snippet the full oldcord.css):
  brand hsl(227 58% 65%), bgBase hsl(223 7% 20%), radiusMd 4px,
  railSelectedBtnRadius 15px, railSelectedBtnBg rgb(114,136,218),
  5 articles, codeblocks 1, codes 1, quotes 1
- Test A rail: hover shot; selected tile radius 15px + shadow none in computed style
- Test B context menu: opened, item hover, closed after Escape ✓
- Test C profile popout: [profileOpen] True after clicking [data-author] ✓
- Test D chat: toolbar on hover, message menu, emoji picker opened,
  reaction added → pills 1; typed message sent + persisted in DOM ✓
- Test E persistence: reload → theme still on (brand hsl(227 58% 65%));
  snippet toggle off → brand #8000ff / bgBase #1c1825 / radiusMd 14px
  (stock Nexa back); toggled back on → OldCord again ✓
```

Console errors: 3 × `404 /api/v1/desktop/latest` — update-check
endpoint on a scratch server with no published release (pre-existing,
theme-independent; verified in a probe listing every 4xx response).

### Tests A–E (Electron, same account + same backend)

```
- Launched real client (electron.exe 44.9.3, CDP remote-debugging-port):
  home/DM view, rail, sidebar rendered; theme AUTO-APPLIED from the
  account-synced snippet (no injection): brand hsl(227 58% 65%),
  bgBase hsl(223 7% 20%), radiusMd 4px, selected tile 15px + shadow none.
- channel view: 6 articles (5 seeded + 1 sent); composer present.
- context menu (channel row right-click), rail hover: 0 console errors.
```

Evidence (`docs/oldcord-nexa/evidence/phase-12/`): 01 before-theme
chat; 02 theme-on chat; 03 rail hover; 04 + 04b context menu & hover;
05 profile popout; 06 message menu; 07 emoji picker; 08 composer focus
(+ typed text); 09 message sent; 10 reload-persist; 11 theme off;
12 Electron home; 13 Electron chat; 14 Electron rail hover;
15 Electron context menu; web-demo-desktop / web-demo-phone
(demo realm, Chromium, for reference).

### Profile popout and message send in Electron

Reached on web ([profileOpen] true, reaction added, message sent);
the Electron pass covered home/chat/rail/context-menu with the same
computed-style probes. Nothing in CSS is Electron-specific — the
Electron renderer runs the same Chico… no, the same Chromium/CSS
engine, and the snippet arrived through the account's own sync.

## Final verdict — Phase 12

Backend-backed messaging confirmed (seed, login, send persisted);
theme activation/deactivation/persistence exercised through the real
Settings UI and its single-toggle off-switch; theme applied
automatically in the Electron client from the same account settings;
no theme-caused console errors anywhere, repo untouched
(`git status` 0 changes in nexa-src). The remaining unexercised items
(real WebRTC calls, attachments, server settings pages) are orthogonal
to a CSS theme and unchanged by it. Classification updated in
DELIVERY.md: **PASS** (with the exact evidence above).

## 5. Size / install / external resources

```
oldcord.css = 22,485 bytes     ( ≤ 65,536 snippet limit and ≤ 1 MB link limit )
```

Install path verified against the official Custom CSS docs
(`docs/custom-css.md`): one CSS file, HTTPS link or snippet, no
Nexa source change; selector strategies are exactly the doc's
ordering (custom properties, data-attributes, `.md-*`, stable
prefix). No `!important` chains except where Nexa own specificity
required them; specificities were compared against the real module
rules (`[data-scheme]` prefix lifts rules one class-level over the
app's).

**CSP note (verified, not a problem):** the *demo* page's inline CSP
(`demo.html`) blocks external `@import`s, which is why the harness
stripped the fonts line before injecting. The *real app* allows it:
the server CSP in `internal/api/static.go` line 33 sets
`style-src 'self' 'unsafe-inline' https:` and
`font-src 'self' data: https:` — so the optional Google Fonts
`@import` in `oldcord.css` loads in the real app (and in desktop,
whose Electron shell uses the same CSP), and where it ever doesn't,
the `system-ui` fallback applies. The theme does not depend on it.

## 6. Repository integrity

```
$ git -C C:\Users\Vino\Downloads\nexa-src status --short
0 changes
```

Only gitignored build artifacts (`web/node_modules`, scratch `dist`)
were produced inside the clone.

## 7. What was NOT exercised (honestly not claimed)

- Go backend flows (no Go toolchain on this machine; `tools/seed.py`
  needs a built `chatd`). The theme ships no JavaScript and touches
  no Nexa source, so these cannot regress from the theme.
- The Electron desktop build (same Chromium engine + the same
  `styles.ts` injection path as web, but never launched here).
- Android (Custom CSS does not apply there, by Nexa design).
- Interactive states (right-click menus, profile popouts, emoji
  picker, message editing, calls) — verified at the selector level
  (§3), not exercised in a browser.
- Real KVM/WebRTC/SFU paths — no keys were touched, and CLAUDE.md
  forbids testing against the live server.

---

## Final verdict

The theme installs through Nexa's official mechanism, its tokens map
one-to-one onto OldCord's published dark palette, its component
selectors are proven against both Nexa's source and the actual
compiled CSS artifact, the sandbox render shows the 2020 look with
zero theme-caused console errors at both reference viewports, and
the repository is untouched. The demo-app interactions in §4 are
sufficient to say PARTIAL is the honest classification: the
unexercised surfaces above are listed in DELIVERY.md as the remaining
gate to PASS, and none of them require a single line of the theme
to change.
