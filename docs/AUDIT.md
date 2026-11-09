# Audit Report: OldCord for Nexa

**Date:** 2026-11-08
**Reference:** OldCord v2.1 (`github.com/milbits/oldcord`, MIT)
**Target:** Nexa `main` branch (`github.com/zeygoober/nexa`)
**Status:** AUDIT COMPLETE — proceed to Fase 1

---

## 1. Repository Verification

| Item | Value |
| --- | --- |
| Remote origin | `https://github.com/zeygoober/nexa.git` |
| Branch | `main` |
| Commit SHA | `3a2ceb95f007372cdcc70138afb247df6e204df6` |
| Working tree | Clean (no uncommitted changes) |
| Target path | `nexa-src/` under `C:\Users\Vino\Downloads\` |

No pre-existing files were modified, removed, or reverted during this audit.

---

## 2. Nexa Architecture Summary

### 2.1 Stack
- **Frontend:** React + TypeScript, built with Vite
- **Desktop:** Electron wrapping the same web app (`desktop/`)
- **Backend:** Go server (`cmd/`, `internal/`)
- **Android:** Kotlin + Jetpack Compose (`android/`) — **Custom CSS does NOT apply on Android**

### 2.2 Theme System
The official customization mechanism lives in **Settings → Custom CSS**. Two input modes exist:

1. **Snippets** (typed editor): up to 64 KB, applied last so they win on equal specificity, follow user account across devices
2. **Themes from Links**: up to 20 HTTPS `.css` files, each ≤ 1 MB; auto-refreshed daily; each has its own toggle per device

CSS order (later wins):
```
Nexa's own styles  →  Plugins' stylesheets  →  Stylesheets from links  →  Your snippet (last)
```

### 2.3 Design Tokens
Defined in `web/src/lib/look/tokens.ts`. Sixty tokens organized into seven groups:

| Group | Tokens |
| --- | --- |
| Backgrounds (8) | `bg-base`, `bg-raised`, `bg-surface`, `bg-floating`, `bg-input`, `bg-composer`, `embed-bg`, `bg-overlay` |
| Interaction (6) | `bg-hover`, `bg-selected`, `bg-active`, `bg-message-hover`, `scroll-thumb`, `scroll-thumb-hover` |
| Text (8) | `text-strong`, `text`, `text-subtle`, `text-muted`, `text-channel`, `text-placeholder`, `icon`, `link` |
| Accent (9) | `brand`, `brand-hover`, `brand-active`, `on-brand`, `text-brand`, `focus-ring`, `mention-bg`, `mention-bg-hover`, `mention-fg` |
| Borders (3) | `border-subtle`, `border`, `border-strong` |
| Status (14) | `positive`, `positive-hover`, `on-positive`, `warning`, `text-warning`, `on-warning`, `danger`, `danger-hover`, `on-danger`, `text-danger`, `status-online`, `status-idle`, `status-dnd`, `status-offline` |
| Content (13) | `tooltip-bg`, `tooltip-text`, `spoiler`, `spoiler-revealed`, `code-bg`, `code-border`, `code-keyword`, `code-string`, `code-number`, `code-comment`, `code-title`, `code-type`, `code-variable` |

Plus global: `--radius-xs/sm/md/lg/xl/pill`, `--font`, `--font-display`, `--font-mono`, `--font-scale`

Plus stable message-text classes (never change):
`.md-paragraph`, `.md-link`, `.md-mention`, `.md-timestamp`, `.md-inline-code`, `.md-codeblock-wrap/.md-codeblock`, `.md-quote`, `.md-spoiler`, `.md-alert`, `.md-details`, `.md-table`, `.md-task`, `.md-hr`, `.md-subtext`, `.md-mark/.md-sup`, `.md-footnotes/.md-fnref`, `.md-emoji-box`

### 2.4 Stable Selection Strategies (from official docs)
1. Custom properties ✅
2. Attributes on page / messages ✅ (`[role="article"][data-mentioned="true"]`)
3. Classes of rendered message text ✅ (`.md-mention`)
4. Stable prefix of hashed class ⚠️ (`[class*="_rail_"]`)

### 2.5 Page Attributes
Sitting on `<html>`: `data-look`, `data-scheme`, `data-theme`, `data-reduced-motion`, `data-underline-links`, `data-animate-names`

### 2.6 Component Structure
Key component directories under `web/src/components/`:
- `sidebar/` — ServerSidebar, DMSidebar, server menu, DM menu
- `chat/` — ChatView, MessageList, MessageItem, MemberList, Embeds, Attachments, VideoPlayer, VoiceMessage, ReplyWords, ReactorsPopover, etc.
- `composer/` — Composer, Autocomplete, MessagePreview, FormatEditor
- `profile/` — ProfileCard, ProfileModal, RelationshipBar, ProfilePopoutHost, BlockDialog
- `mentions/` — MentionsModal
- `threads/` — ThreadsPanel, CreateThreadModal
- `follows/` — FollowDialog, FollowedChannels
- `common/` — Tooltip, Twemoji, ZoomPan
- `games/` — GamePage, GameChooser
- `discovery/`, `badges/`, `support/`, `legal/`

All components use CSS Modules with **hashed class names** that change between releases. Stable selectors are attributes and `.md-*` classes.

---

## 3. OldCord Architecture Summary

### 3.1 Version & License
- Version: **2.1**
- License: MIT (copyright: `milbits, kinggamingyt`)
- Source: `github.com/milbits/oldcord/tree/main`
- Entry point: `OldCord.theme.css` — single file that imports all component CSS

### 3.2 Module Structure
Entry imports 7 component CSS files in order:
```
vars.css      (token palette mapping)
color.css     (brand colors: blurple, green, red, brand-experiment)
imgs.css      (blurple background layer, base64 avatars)
other.css     (Whitney font @font-face declarations)
profile.css   (profile panels, popouts, custom-status prompts)
redesign.css  (layout: title bar removal, grid structure, code blocks)
main.css      (additional layout, platform-specific fixes, platform-web)
```

Plus optional addons: `url(...)` based on commented-out enable flags.

### 3.3 Token Remapping (key mappings from Discord defaults)
| OldCord token | Maps to Nexa token | Reason |
| --- | --- | --- |
| `--primary-500` (blurple) | `--brand` | Discord #5865f2 → Nexa #8000ff |
| `--primary-600` | `--brand-hover` / `--danger` | Hover variant; danger color |
| `--primary-630` | `--bg-composer`, `--message-bg` | Composer background, primary surface |
| `--primary-660` | `--bg-surface`, `--accent-fill` | Secondary surface, elevated panel |
| `--primary-700` | `--btn-secondary` | Button secondary fill |
| `--primary-800` | `--input-border` | Input border |
| `--background-primary` | `--bg-raised` | Main message area |
| `--background-secondary` | `--bg-surface` | Card surfaces |
| `--background-tertiary` | `--bg-floating` | Floating menus/popovers |
| `--interactive-text-default` | `--text-channel`, `--text-subtle` | Channel/secondary text |
| `--channels-default` | `--text-channel` | Channel name text |
| `--header-primary` | `--text-strong` | Headings |
| `--green` | `--positive` | Success state |
| `--red` | `--danger` | Danger state |
| `--background-gradient-lowerest` | `--input-background-default` | Input background |
| `--background-gradient-lowest` | `--primary-700` | Primary pill backgrounds |
| `--menu-item-default-active-bg` | `--bg-active` | Menu active background |

### 3.4 External Dependencies
1. **Whitney woff2 fonts** served from `milbits.github.io/oldcord/src/assets/` — license not explicitly stated in source; likely SIL OFL (Discord uses them). CanNOT be redistributed verbatim without verification.
2. **Base64 blurple image** in `imgs.css` — embedded data URI, no external dependency, but copyright of original Discord asset is not ours to copy.
3. **Fonts: code** referenced via `var(--font-code)` — Nexa's own monospace stack.

---

## 4. Correspondence Analysis

### 4.1 Directly Adaptable Components
These map cleanly and can be translated with high fidelity:

| Nexa Component | OldCord Element | Strategy |
| --- | --- | --- |
| Sidebar server icons | `.guilds_pog62O` (icon containers) | Match stable wrapper classes; override icon sizing/active ring |
| Channel list items | `.themed_6zZYhF` (channel rows) | Use `.md-*` or stable prefix selectors |
| Category headers | `.header__16eb0` with `eyebrow` pattern | Semantic class matching |
| Message actions bar | `.buttonContainer_2i8Bfm` → equivalent in Nexa | Style container + inner buttons |
| Code blocks | `.hljs` elements with `.hljs-*` tokens | Match `.md-codeblock-wrap`, style keyword/string colors |
| Reply flash highlight | `.highlighted__5126c`, `.replying_SY_P2T` | CSS animation on existing states |
| Embeds | `.container_seLrDE` → Nexa `.md-embed` or card | Box model adaptation |
| Profile panel | `.panels_pog62O .avatarWrapper__37e49` | Profile overlay |
| Dropdown menus | `.menu_M3qk4j` → Nexa dropdown variants | `!important` + semantic selectors |
| Buttons | `.button_9OQtot` → Nexa generic button | Color via `--brand` mapping |
| Avatar rings | `.outer_ERB-Vv` (server icon active ring) | SVG mask-like approach via border-radius |

### 4.2 Differently Implemented Components
These require substantially different implementations:

| Nexa Component | Challenge | Approach |
| --- | --- | --- |
| **Server sidebar layout** | Nexa uses different grid structure; OldCord's 3-column grid (guilds/channels/page) must be rebuilt using Nexa's existing panel structure | Adapt border-radius, heights, sidebar collapse behavior |
| **Title bar removal** | Nexa has its own chrome; OldCord removes title bar via grid trickery which won't apply | Hide via display:none on title components when appropriate; adjust spacing |
| **Channel tab bar** | Nexa uses different tab rendering with `--custom-guild-sidebar-width`; OldCord's pill-style tabs may clash | Use Nexa's built-in tab component styling; override visual aspects only |
| **Voice channel indicators** | Nexa has `VoiceView.tsx` with different architecture | Style the existing voice indicators semantically |
| **Profile overlay** | Nexa ProfileModal is different from OldCord's popout system | Style overlay appearance, layout adjustments |
| **Emoji picker** | Nexa has separate emoji picker component; OldCord's integrated drawer differs | Adapt picker styling only, keep functionality |
| **Code block syntax highlighting** | OldCord remaps hljs tokens; Nexa has its own code token system | Map old `.hljs-*` colors to `.md-code-keyword` etc., but skip where Nexa already handles |
| **Popout/popover styling** | Different component hierarchy | Shadow/elevation adaptation, border-radius |

### 4.3 Selectors That Must Be Restructured
Critical risk areas:
- **Hashed class names**: OldCord relies heavily on `.sidebar_pog62O`, `.guilds_pog62O`, `.channelName__754bd`, etc. These do NOT exist in Nexa's current codebase. Must find equivalent semantic targets.
- **Platform-specific fixes** (`html.platform-web`, `html.platform-osx`, etc.): Nexa doesn't use these platform classes; should be replaced with `[data-scheme="dark"]` equivalents or scoped appropriately.
- **Vencord compatibility hacks** (`vc-betterFolders-sidebar-grid`): Nexa isn't a Vencord fork; this code is dead weight and should be dropped.
- **Data URI assets** (`imgs.css` blurple sprite, `redesign.css` emoji SVGs): Replace with Nexa-native approaches (CSS gradients or inline SVG using Nexa's design tokens).

---

## 5. Compatibility Risks

### 5.1 High Risk
| Risk | Impact | Mitigation |
| --- | --- | --- |
| Hashed class names change in Nexa releases | Parts of theme break after update | Use only semantic selectors (attributes, `.md-*`, stable prefixes) |
| `!important` chain conflicts | Specificity wars cause unpredictable results | Minimize `!important`; scope overrides carefully |
| Token namespace collision | OldCord custom vars vs Nexa `--brand`, `--danger` etc. | Full remap to Nexa tokens, don't reuse old var names |
| `data-scheme="dark"` scoping | Multiple themes (Dark, Light, Ash, Onyx, presets) | Scope by `[data-scheme]` appropriately |

### 5.2 Medium Risk
| Risk | Impact | Mitigation |
| --- | --- | --- |
| Font licensing (Whitney) | Legal issue if redistributed | Use Nexa system font stack; accept minor typographic difference |
| External font URLs (woff2 from github.io) | Broken if OldCord repo moves | Localize fonts or use Google Fonts alternative (Atkinson Hyperlegible — Nexa's built-in) |
| Android incompatibility | OldCord CSS doesn't apply on mobile | Document limitation; don't touch `android/` directory |
| Backdrop-filter / macOS specific CSS | May not work in web build | Test web target primarily; Electron inherits browser engine |

### 5.3 Low Risk
| Risk | Mitigation |
| --- | --- |
| Line-height differences between Discord/Nexa | Test and adjust; preserve at least 16px row height |
| Border-radius differences | Adapt radii per Nexa standard (6px md, 8px lg) |
| Scrollbar styling | Nexa manages scrollbars; suppress or adapt if needed |

---

## 6. External Resource Dependencies

| Resource | Source | License Status | Action |
| --- | --- | --- | --- |
| Whitney woff2 fonts | `milbits.github.io/oldcord/src/assets/*.woff2` | Unverified (likely OFL) | Replace with Nexa-available fonts (Atkinson Hyperlegible fallback) |
| Blurple background sprite | `imgs.css` base64 data URI | Copyright of Discord asset — NOT ours | Replace with CSS gradient equivalent |
| Discord logo in custom-status modal | `redesign.css` URL to `discord.com/assets/` | NOT ours — DO NOT COPY | Remove; use a placeholder or Nexa-native icon |
| Go live modal art | `redesign.css` URL to `static.discord.com/assets/` | NOT ours — DO NOT COPY | Remove entirely |
| Lottie animations (voice) | Various CDN URLs | Unknown | Evaluate; could potentially be replaced by static icons |

**Implication:** The final CSS must not depend on any external resource except optionally Google Fonts (which is widely available and commonly accepted). All Discord-origin assets must be stripped.

---

## 7. Test Status of Original OldCord

- No automated tests exist for OldCord's CSS components
- Community tests report partial functionality on Discord 2020–2022 era; broken on newer Discord due to hash regeneration
- No regression tests documented

**Implication:** No pre-existing test infrastructure to leverage. All validation must happen manually against Nexa's live app.

---

## 8. Audit Conclusions

1. **Can proceed to Fase 1.** The project is clearly within scope. No backend, auth, E2EE, or permission systems need modification.

2. **The primary technical challenge is selector migration.** OldCord's approach of copying Discord's hashed classes with `!important` won't transfer directly to Nexa. The theme must be built on Nexa's stable selection strategies (custom properties, attributes, `.md-*` classes, stable prefixes).

3. **Token mapping is well-defined** — Section 3.3 provides a complete mapping table. The major decisions are: which old tokens map to which Nexa tokens (done above), and whether to also create a legacy "oldcord" token namespace for backward compatibility or keep things simple.

4. **Recommended decision: keep things simple.** Do NOT carry over the OldCord token variable names beyond the mapping table. The final CSS uses only Nexa's official token namespace. This keeps maintenance straightforward when Nexa updates its tokens.

5. **Font strategy:** Use Nexa's default font stack (Twemoji Text first, then system-ui sans-serif; monospace available). Accept that Whitney wasn't ported — the typographic differences are minor for the purposes of recreating the 2020 Discord aesthetic.

6. **External dependencies:** Strip ALL external resources except optional Google Fonts (loaded once, not repeated). This means:
   - Drop the Whitney `@font-face` rules
   - Replace `data:image/...` blurple sprites with CSS gradients
   - Remove Discord logo references
   - Keep all other functionality (icons, illustrations handled by Nexa natively)

7. **Estimated scope:** The theme can reasonably achieve good coverage of the main UI surfaces (sidebar, channel list, chat, messages, buttons, menus, popovers, profiles) in V1. Full parity with every feature (especially complex ones like polls, threads, discovery pages) would require significantly more effort than stated in this brief.

8. **Recommended final classification: PARTIAL** unless the Fase 1–7 work achieves comprehensive coverage across all 10 required screens without breaking functionality. PARTIAL is honest given the constraints and the scope of work remaining.

---

## Files Created During Audit

- `docs/oldcord-nexa/AUDIT.md` — this document

## Files Modified

- None
