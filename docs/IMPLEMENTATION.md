# Implementation Notes — OldCord for Nexa

**Phase 11 revision (2026-11-08).** The implementation below replaces the
Fase 1–7 draft. That draft shipped selectors that never matched the real
Nexa DOM (`::deep()` pseudo-classes — invalid CSS, silently dropped),
fantasy class names copied from Discord (`sidebar__logo-wrap`,
`composer`, `profile-popout`), and a dead scope (`[data-look="oldcord"]`,
an attribute Nexa never sets). All of it is gone. What remains is
verified against the component sources at commit `3a2ceb95f` and against
the selectors the app's own Custom CSS documentation blesses.

---

## Deliverables (themes/oldcord-nexa/)

| File | About | Size |
| --- | --- | --- |
| `oldcord.css` | The installable file: tokens + component CSS + optional Google Fonts @import (first line). Install via **Settings → Custom CSS → Themes from Links** or paste into a **Snippet**. | 22,485 B (< 64 KB snippet, < 1 MB link) |
| `tokens.css` | Source of the token file above (everything after the @import line). | 7,874 B |
| `style.css` | Source of the component file above. | 14,504 B |
| `README.md` | Install instructions and feature list. | |
| `IMPLEMENTATION.md` | This file. | |
| `COMPATIBILITY.md` | Platform matrix, safe mode, failure modes. | |
| `LICENSES.md` | MIT (milbits/oldcord v2.1) + external-asset audit. | |

To rebuild `oldcord.css` after editing: prepend the Google Fonts
`@import` line to `tokens.css`, then append `style.css`. Keep the
`@import` at the very top (it is only legal as the first rule).

---

## Architecture

The theme is a single stylesheet loaded after every Nexa stylesheet,
so equal-specificity rules win by order. Two properties make it safe:

1. **Tokens first.** Everything colour, typeface, radius and dimension
   related is done by overriding Nexa's official custom properties
   (the 61 tokens of `web/src/lib/look/tokens.ts` plus geometry and font
   variables). Nexa's modules already consume those tokens, so the theme
   repaints the app without ever fighting a component rule.
2. **Component rules only where a token cannot express the 2020 look**
   (server-tile radius morph, selected glyph colour, amber mention row,
   squared reaction pills, browser-default-safe menu fills, flat buttons,
   embed bar colour, profile card surfaces).

All overrides are scoped to `[data-scheme="dark"]`. OldCord is a dark
theme; on a Light preset the stylesheet stays inert instead of producing
a hybrid. (`[data-look="oldcord"]` does not exist in Nexa — `data-look`
is set to the appearance id such as `dark`, `forest`, `custom-<id>` —
so the previous scoping was dead.)

### Token source (`tokens.css`)

OldCord v2.1 `vars.css` restores Discord's 2020 dark palette. The
mapping onto Nexa's names is one-to-one; the hsl() values are copied
verbatim so there is no hex approximation drift:

| OldCord | Nexa token |
| --- | --- |
| `--background-primary` hsl(220 8% 23%) ≈ #36393F | `--bg-raised`, `--embed-bg` input hover base |
| `--background-secondary` hsl(223 7% 20%) ≈ #2F3136 | `--bg-base` (sidebar, members), `--code-bg` |
| `--background-secondary-alt` hsl(220 7% 17%) ≈ #292B2F | `--bg-surface` |
| `--background-tertiary` hsl(216 7% 14%) ≈ #202225 | `--scroll-thumb` + sidebar header blocks |
| `--background-floating` hsl(220 8% 10%) ≈ #18191C | `--bg-floating` |
| `--channels-default` #8E9297 | `--text-channel` |
| `--interactive-text-default` #B9BBBE / default `#DCddde` | `--text-subtle` / `--text` |
| `--text-muted` #72767D | `--text-muted`, `--text-placeholder` |
| `--text-link` (blue-345) #00AFF4 | `--link` |
| `--brand-experiment` hsl(227 58% 65%) = #7288DA (2020 blurple) | `--brand`, `--text-brand`, `--focus-ring` |
| `--green` hsl(153 46% 49%) | `--positive`, `--status-online` |
| `--red` hsl(0 85% 61%) ≈ #ED4245 | `--danger`, `--status-dnd` |
| `--yellow` hsl(38 96% 54%) ≈ #FAA61A | `--warning`, `--status-idle` |
| `background-mod-subtle/-muted/-strong` hsla(217 8% 34% / 0.16/0.24/0.32) | `--bg-hover`, `--bg-active`, `--bg-selected` |
| `background-message-hover` hsla(240 11% 2% / 0.07) | `--bg-message-hover` |
| `background-mentioned` hsla(38 96% 54% / 0.051) | `--mention-bg` (fg #FEE0B3, Discord 2020) |
| `.hljs` github-dark roles (redesign.css) | `--code-keyword/string/number/comment/title/type/variable` |
| `.hljs` bg `background-base-lowest`, border `primary-700` | `--code-bg`, `--code-border` |
| `--__spoiler-background-color--hidden` primary-700 | `--spoiler` |

**Nexa's own experimental hue (--brand #8000FF) is replaced by blurple.**
Nexa Dark's violet palette (#1C1825 etc.) intentionally gave way to the
2020 greys; chrome-less corners and 2020 shadows (near-black elevation)
are flattened through the tokens Nexa derives (`--bg-rail`, `--glow-brand`,
gradients all recompute from the overridden base/brand).

### Geometry & type (`[data-scheme="dark"]` block)

| Variable | Value | Why |
| --- | --- | --- |
| `--rail-w` | 72px | matches OldCord `--custom-guild-list-width` |
| `--sidebar-w` | 240px | matches `--custom-guild-sidebar-width − 72 − 1` |
| `--members-w` | 240px | OldCord `--custom-member-list-width` |
| `--header-h` | 48px | OldCord `--custom-channel-header-height` |
| `--radius-xs/sm/md/lg/xl/pill` | 2/3/4/5/6/999px | 2020 corners (buttons 3, menus 4, modals 5) |
| `--font`, `--font-display` | Twemoji Text, Atkinson Hyperlegible, system-ui… | Whitney is not redistributed; see §6 |
| `--font-mono` | JetBrains Mono, Fira Code, ui-monospace… | OldCord `--font-code` stack |

---

## Selector strategy (verified)

Each rule was cross-checked against the component `.module.css` files
with the audit script `oc-audit.py` (ran against the scratch checkout
during the verification session; the same checks are reproducible as
documented in TEST_REPORT.md) and its failures debugged:

| Strategy | Used for | Examples |
| --- | --- | --- |
| `data-*` / `role=` attributes (stable, always) | message rows, profile variants, scheme | `[role="article"][data-mentioned="true"]`, `[class*="_card_"][data-variant="popout"]`, `[data-scheme="dark"]`, `[role="toolbar"]` |
| `.md-*` rendered markup classes (stable, documented) | inline code, quotes | `.md-inline-code`, `.md-quote` |
| `[class*="_local_"]` hashed-class prefix | module internals | `[class*="_rail_"] [class*="_btn_"]`, `[class*="_sidebar_"] [class*="_row_"]` |
| `:has()` + ancestry/attribute narrowing | shared locals (`_box_`, `_btn_`, `_pop_`, `_card_`, `_item_`) | `[class*="_box_"]:has(_attach_):has(_btns_)`, `[class*="_menu_"] [class*="_item_"]`, `[class*="_btn_"][class*="_primary_"]`, `[class*="_pop_"]:has([class*="_grid_"])` |

**Critical verification finding (Phase 11).** Nexa's CSS Modules
output has *no* module-name prefix — a generated class is
`_<local>_<hash>` (e.g. `_rail_1143b_339`). The Fase 1–7 draft used
`ModuleName_local_` fragments (`Button_btn_`, `ContextMenu_menu_`,
`Composer_box_`) which matched **zero** generated classes in the
actual build; every component rule built on them was silently
inert. This was caught by asserting every fragment against the
compiled `dist/assets/*.css` (25/25 fragments now match ≥1 generated
class; see TEST_REPORT.md §3).

**Explicitly removed:** `::deep()` (not plain CSS — web browsers dropped
those rules entirely), `[data-look="oldcord"]` scoping, Discord
hashed classes (`.menu_M3qk4j`, `.wrapper_JJ-hgR` …), and all fantasy
classes that do not exist in Nexa (`composer`, `post-container`,
`channel-list`, `profile-popout`, `threads-panel`, ...).

**Specificity.** Custom CSS loads after every app stylesheet, so ties go
to the theme. Where a component rule is more specific than a natural
theme rule (e.g. `.box:focus-within:not([data-quiet])`), the theme rule
adds the `[data-scheme="dark"]` ancestor to lift it one level. No
`!important` chains targeting non-existent elements remain.

---

## Server rail fix (Phase 11 §2)

Discord 2020 guild tiles: circle at rest, morphing `border-radius`
25px → 16px (hover) / 15px (selected), `transition: border-radius 0.2s`.
Nexa tiles are 48px and already morph 16→14px; the morph direction and
the indicator glow belonged to Nexa, not to 2020. The fix:

- Resting tiles are circles (`border-radius: 50%`), morphing to **16px**
  on hover and **15px** when selected (OldCord values applied to Nexa's
  48px metric; radius percentage keeps the circle exact).
- Hover/selected fill is flat blurple (`var(--brand)`), text forced
  white, hover translate removed (2020 tiles don't lift).
- Selected picture tiles keep their picture, with the OldCord dark
  outer ring (`box-shadow: 0 0 0 2px #202225` = `.outer_ERB-Vv`'s
  `primary-700` background) instead of Nexa's accent glow ring.
- The left-edge indicator keeps Nexa's bar but with the glow removed —
  2020's bar is plain white for unread, hover-grown, taller for active.
- The add-server plus turns solid green on hover (OldCord circle icon).

Everything's readable at both breakpoints; no layout values were changed
(sidebar/rail widths come from tokens, already fixed in §Geometry).

---

## What was NOT touched (regression surface)

The theme outputs no JS and touches no Nexa source file. Backend, DB,
auth, E2EE, WebRTC, persistence, Android (Custom CSS does not apply
there) are all unaffected by construction. `git status` in nexa-src
remains clean (verified in TEST_REPORT.md §6).

---

## Known gaps

| Gap | Impact | Mitigation / status |
| --- | --- | --- |
| Voice/call stage in Light preset uses Nexa's purple (data-dark-scope) | Only when mixing OldCord CSS with a Light preset | OldCord is dark-only; README instructs the Dark preset |
| Profile `.sheet` background uses --oc-primary-660 | Sheet tint fixed in tokens; per-user profile themes still drive their own accents | Documented; `data-themed` cards keep their own palette tokens |
| Menu hover uses OldCord's *current* brand-experiment-560 slate-blue | Slightly stronger than Discord's 2020 menu grey | Faithful to what OldCord ships today |
| Animated replies/threads/timelines follow Nexa's own rules | Some 2020 sub-behaviours (compact density, Lottie) have no token analogue | Cosmetic only; function unchanged |
| Whitney substitute | Minor typographic difference | Atkinson Hyperlegible via optional Google Fonts @import (see below) |

---

## Fonts & external resources

OldCord loads Whitney `.woff2` from `milbits.github.io` (license of
the files unverified — treat as unredistributable). The theme instead:

- `--font` keeps `"Twemoji Text"` first (Nexa's emoji rule), then
  Atkinson Hyperlegible, then `system-ui`.
- The **only** external request is the single Google Fonts
  `@import` at the top of `oldcord.css` — the exact pattern Nexa's
  Custom CSS docs use for fonts. Removing it costs nothing: the
  `system-ui` fallback still applies.
- No images, no data URIs, no third-party assets, no Discord logos.

---

## Audit & verification (how this revision was checked)

1. **Selector audit (`oc-audit.py`)**: every `[class*="…"]` fragment in
   `style.css` is resolved against the `.module.css` files of
   `web/src/components/**` — 25/25 resolve to real locals (see table §Selector).
   Every `var(--x)` resolves to tokens.css or Nexa's `styles/*.css`.
   No `::deep(`, no Discord-hashed class, no `url()`, no http(s) in
   `tokens.css`/`style.css` (the single `https://` in the repo is the
   documented Google Fonts `@import` in `oldcord.css`).
2. **Syntax check**: both files parse in PostCSS (vite pipeline):
   `tokens.css` 2 rules, `style.css` 34 rules, `oldcord.css` 36 rules.
3. **Compiled-CSS check** (vite build → dist): *the generated class
   names* in `dist/assets/*.css` were extracted and every fragment in
   `style.css` was asserted to match at least one emitted class name
   (`oc-fragments.py`, 25/25 OK).
4. **Live rendering**: `demo.html` (Nexa's in-browser demo, full fake
   API incl. E2EE message crypto) served statically; the theme injected
   as a `<style>` element exactly as Nexa's own loader does; screenshots
   at 1366×768 and 411×914 with console-error capture (`oc-visual.py`).
   Computed token values were verified to land on the page
   (`--brand: hsl(227 58% 65%)`, `--bg-base: hsl(223 7% 20%)`,
   `--radius-md: 4px`).
5. **Regression**: zero changes inside `nexa-src` (`git status` clean),
   so backend/auth/messaging/search/voice codepaths cannot regress.
   A deeper interactive pass also created a server through the rail
   and verified the rail, sidebar rows, and ContextMenu render with
   the theme; a full channel conversation could not be reached in
   this sandbox (the demo's default navigation lands on the DM view),
   so composer/menu/popout screens stand on their fragment proof
   (§3) rather than screenshots.

Each check is reproducible; commands and outputs are quoted verbatim in
`docs/oldcord-nexa/TEST_REPORT.md`.
