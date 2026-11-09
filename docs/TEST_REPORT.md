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

---

# Addendum — Phase 15: V1.1.0 profile color adaptation (2026-11-09)

**Date:** 2026-11-09
**Target:** Nexa `main` @ `3a2ceb95f` (`zeygoober/nexa`), scratch Go
backend (`chatd` dev binary) at `127.0.0.1:18089`
**Theme revision:** `work/v1.1.0-visual-refinement` (this file's repo),
`oldcord.css` 23,505 B — candidate, **not published** (v1.0.0 tag
untouched)

## 8. What was wrong in V1.0.0 (discovered, not assumed)

V1.0.0's profile rules (style.css, specificity (0,3,0) with the
`[data-scheme]` ancestor) lost to inline styles but **beat** Nexa's
`.card[data-themed]` gradient rule (0,2,0) — flattening every prepared
two-color profile and avatar tint to `#2F3136`, and flattening themed
translucent sheets. Computed proof under the *true* V1.0.0 file
(extracted from tag v1.0.0, SHA-256 `1b75566e…`):

| Case | V1.0.0 computed |
| --- | --- |
| two-color profile | `data-themed=light` present, but `cardBgImage: none`, `cardBg: rgb(47,49,55)` — gradient gone |
| two-light profile | gradient gone AND black foregrounds on the `#2F3136` card (unreadable) |
| themed modal sheets | `rgb(40,42,46)` solid in all cases (translucency flattened) |
| one-color | banner keeps the accent (inline style beats the stylesheet — already correct in V1.0.0) |
| default | `#2F3136` shell, sheet `#282A2E`, banner = Nexa's `avatarColor` palette color (inline — the forced `#202225` banner rule never rendered) |

## 9. The fix (style.css only; tokens.css byte-identical to V1.0.0)

- Card shell now applies only to un-themed, un-tinted cards:
  `:not([data-themed]):not([data-tinted])`.
- Sheets now set the `--sheet-bg` token to `--oc-primary-660` on
  `popout|modal|panel` `:not([data-themed])`, instead of a forced sheet
  `background` — the same computed value for un-themed cards, and
  themed translucent sheets show through untouched.
- The dead `.banner` rule is removed (inline styles always win; see
  §8 table). No `.banner` stylesheet rule may be reintroduced without
  `!important`, which would break single-accent profiles.

## 10. Probe results (v1.0.0 → v1.1.0), popout + modal

Computed styles from the scratch backend with disposable profile
colors on bob (evidence: `docs/oldcord-nexa/evidence/phase-15/{v1,v1_1}/`).

| Case | V1.0.0 | V1.1.0 | Verdict |
| --- | --- | --- | --- |
| default | `#2F3136` shell, sheet `rgb(40,42,46)` | identical | ✅ unchanged |
| single `#1C86EE` | banner `rgb(28,134,238)` | identical | ✅ unchanged |
| two-dark `#8B0E1E→#FF6B6B` | flat `#2F3136`, no image | `linear-gradient(rgb(139,14,30) 0%, rgb(255,107,107) 100%)`, white text | ✅ fixed |
| two-dark modal sheet | `rgb(40,42,46)` | `rgba(0,0,0,0.3)` (Nexa translucent) | ✅ fixed |
| two-light `#FFFFFF→#FFD9E8` | flat `#2F3136`, black text on it | white→pink gradient, `rgb(0,0,0)` text | ✅ fixed |
| two-light modal sheet | `rgb(40,42,46)` | `rgba(255,255,255,0.42)` | ✅ fixed |

Side note: the dumped modal sheets include Nexa's own un-painted inner
side sheet (`.side` carries the paint via `--sheet-bg`,
`Profile.module.css:277/283`) — its `rgba(0,0,0,0)` is Nexa's design,
present in both versions.

## 11. Panel variant (DM column)

Runtime (DM created via `POST /api/v1/users/@me/channels`):

| Case | Result |
| --- | --- |
| two-dark panel | `data-themed=light`; gradient intact; sheet `rgba(0,0,0,0.3)` ✅ |
| default panel | `cardBg rgb(47,49,55)` via `--bg-base` tokens (no OldCord card rule needed), dark sheet ✅ |

## 12. Roles (15A-G)

Runtime (roles created + assigned via the API, worst-case: two-dark
gradient profile on bob):

- Bob's author name in #general: `rgb(237, 66, 69)` = `#ED4245`
  (Admins — highest role) ✅
- Popout role dots: inline `style="background: rgb(...)"` passes
  through untouched — red `#ED4245` + green `#57F287` ✅
- Role names (Admins/Mods): `rgb(245, 245, 247)` themed-light
  foreground, readable on the gradient ✅
- Member list: bob grouped under "Admins", name in `#ED4245` ✅

Multiple same-pair roles from repeated probe runs (scratch only) —
harmless; each dot kept its exact server color. No theme rule touches
role surfaces; this is structural (inline styles) plus now runtime-proven.

## 13. Regression surface

Only the PROFILE section of style.css changed (plus rebuilt
`oldcord.css`). No tokens, no other selectors. General UI (rail,
sidebar, member list, composer, message markdown/mentions/code,
search header) rendered normally in all matrix screenshots; the only
console errors were media 404s for avatar-less accounts (pre-existing,
not theme-related).

## 14. What was NOT exercised (honestly not claimed)

- Banner-image and avatar-tint runtime cases — scratch media pipeline
  returns 500 for image uploads (verified against
  `media_handlers_test.go` rawPut expectations). Static cover:
  `.banner` has **no stylesheet background rule at all**, so an inline
  `background-image` cannot be removed; tints share the exact cascade
  as the probed two-color gradient (`.card[data-tinted]` is in the same
  specificity war as `.card[data-themed]`, which V1.1.0 wins by scoping).
- Profile designer preview (`data-design`) and the public profile page
  (`variant="modal"`): same `ProfileCard` component and selectors as
  the probed modal; not runtime-driven (roles/buttons state was left
  default).
- Accessibility beyond computed foreground flips: Nexa's own
  `data-themed` text-shadow and focus-ring logic were preserved by
  construction; no new animations were added (`prefers-reduced-motion`
  untouched).

## 15. Final classification (Phase 15 / V1.1.0 candidate)

**PARTIAL** — every runtime-verifiable acceptance criterion passes
(gradients, single accents, sheets, panel, roles-by-construction), and
the style change is mechanically tiny; the classification stays at
PARTIAL (rather than PASS) because §14 lists runtime-unverifiable
cases (banner images, avatar tints, designer preview) and one
platform-CSS constraint (the default `avatarColor` banner, see
IMPLEMENTATION.md § Profile color adaptation). Nothing blocks
publishing once approved, but the claim is kept honest.
