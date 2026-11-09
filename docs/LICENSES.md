# Licenses — OldCord for Nexa

## Theme itself (this directory)

**License:** MIT

**Source:** [milbits/oldcord](https://github.com/milbits/oldcord) v2.1

Copyright © 2026 milbits, kinggamingyt

```
MIT License

Copyright (c) 2026 milbits, kinggamingyt

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```

This license applies to all files shipped under `themes/oldcord-nexa/`. The token remapping
and component adaptations are derivative works of OldCord's styling; they carry the same
license as their source.

## External assets

| Asset | Source | License | Action taken |
| --- | --- | --- | --- |
| Whitney font woff2 | milbits/oldcord MIT | Likely SIL OFL (Discord uses Whitney under OFL) | Replaced with Nexa built-in `Twemoji Text` / `Atkinson Hyperlegible`; not redistributed |
| Blurple base64 sprite | Discord original asset | Not our copyright — we did not create it | Replaced with CSS gradient |
| Discord logo | discord.com | Not our copyright — DO NOT COPY | Removed entirely |

All third-party artworks (Discord logos, blurple sprites, etc.) that exist on platforms we
do not control have been eliminated from the final output. No external resource dependencies
remain except optionally Google Fonts (loaded once, per the official Nexa custom CSS docs).

## Build environment

The theme was produced using:

- Nexa source at commit `3a2ceb95f007372cdcc70138afb247df6e204df6` (MIT, zeygoober/nexa)
- Custom CSS documentation from `nexa-src/docs/custom-css.md`
- Design tokens from `nexa-src/web/src/lib/look/tokens.ts`
