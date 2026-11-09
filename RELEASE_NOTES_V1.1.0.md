# OldCord for Nexa — v1.1.0 release notes

**Status:** release candidate — **not yet published**. Tagged and
published separately after explicit approval. The published v1.0.0
release is unchanged.

## What changed

V1.0.0 established the Discord-2020 look on Nexa's Custom CSS system.
V1.1.0 fixes the one place where it fought Nexa's own per-user
customization: **profiles**.

### Profile color adaptation

Nexa gives every user an identity: a two-color gradient card
(`accent_color` + `accent_color_2` rendered as
`--profile-top → --profile-bottom` with a light/dark foreground flip),
a single accent banner, a banner picture, or an avatar-derived tint.
V1.0.0's profile rules forced a fixed `#2F3136` card background on
popouts and modals. At specificity (0,3,0) they beat Nexa's
`.card[data-themed]` gradient rule (0,2,0) — so **every user's custom
gradient and avatar tint rendered as a flat grey card**.

V1.1.0 scopes the classic surfaces to cards with neither
`data-themed` nor `data-tinted`:

| Profile | V1.0.0 | V1.1.0 |
| --- | --- | --- |
| No custom colors | Flat `#2F3136` card, `#292B2F` sheet | Same |
| Two custom colors | Gradient flattened to `#2F3136` | Nexa's gradient intact, sheet keeps Nexa's translucent sheet + blur |
| One custom color | Banner accent survived (inline style) | Same |
| Banner image | Image survived (inline style) | Same |
| Avatar tint | Glow flattened to `#2F3136` | Nexa's glow intact |
| Server roles | Colors intact (inline styles) | Same |

The V1.0.0 banner rule (`#202225`) was inert — `.banner` always
carries an inline style, which beats every stylesheet rule — so it
was removed instead of being "fixed".

### What V1.1.0 deliberately does not change

- A profile with no custom colors keeps Nexa's deterministic per-user
  banner color (`avatarColor` palette). No CSS selector can tell that
  inline fallback from a user's single chosen accent, so preserving
  the banner is the only correct CSS-only behavior. Default profiles
  otherwise use the classic `#2F3136` / `#292B2F` surfaces.
- No other component rule changed: tokens are byte-identical to
  v1.0.0, and every non-profile selector is untouched.

## Verification

Backend-backed profile matrix on the scratch stack (seeded accounts,
disposable profile colors): default / single / two-dark / two-light
× popout + modal, and the DM `panel` variant under two-dark.
Computed styles and screenshots: evidence under
`docs/oldcord-nexa/evidence/phase-15/` (see `docs/TEST_REPORT.md`).
The avatar-image and banner-image matrix cases are statically
verified — the scratch backend's media pipeline returns 500 for
image uploads, and inline `background-image` cannot be overridden
by stylesheet rules anyway.

## Install

Same file, same URL. Refresh or re-paste; Nexa re-fetches links
daily (or use **Refresh** in Settings → Custom CSS):

`https://raw.githubusercontent.com/afterintimacy/oldcord-for-nexa/main/oldcord.css`

Supported platform: **Nexa Web only**.
