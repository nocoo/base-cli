# base-cli brand assets

## Approved source

- Hexly study: `base-cli/2026-10-18-01`, finishing pass `01`.
- Archive revision: `6299f3e9274990444a932d1214ed8855082a146e` in `nocoo/hexly.ai`.
- [Source metadata](https://github.com/nocoo/hexly.ai/blob/6299f3e9274990444a932d1214ed8855082a146e/artwork/logo-family/base-cli/2026-10-18-01/material-metadata.json).
- Local archive: `hexly.ai/public/logos/family/base-cli/2026-10-18-01/01/`.
- Approved native source: **1024 x 1024 JPEG**, generated with the built-in
  `image_gen.imagegen` tool; the underlying model is **unknown** (`model: null`).
- Raw JPEG SHA-256: `3d585f0a2098004d6ae4f46023ad78001eaed49e07672106fd4cea0a2b0768cc`.
- The canonical transparent PNG is the exact native-size finished master, copied
  without re-encoding. The archive's 2048 px exports are declared upscales, not
  native output. This adoption performs no generation, redesign or recoloring.
- Owner approval and controller READY on 2026-10-18 authorize **local adoption
  only**. A source commit does not establish push, publication, npm release or
  deployment.

## Files and roles

| Repository path | Pixels | Role | SHA-256 |
| --- | --- | --- | --- |
| `logo.png` | 1024 x 1024 | Canonical transparent foreground; small UI and browser source | `d33fca8b5f824a6cffd14274524302f15beb82130381c77e196124370e6721a9` |
| `assets/brand/icon.png` | 1024 x 1024 | Square presentation; large artwork only | `bcd48b40ea288c6e7f03e5eccccca268c4f5e819abf7a1c19055713334f38723` |
| `assets/brand/icon-rounded.png` | 1024 x 1024 | Rounded presentation; both README headers at 128 CSS px | `ae5531edbf8f5cbc29fbf5d734a7a534c21b87a22807026508e7d07185f9ce68` |

Use the same transparent foreground on light and dark surfaces. Do not add a
background tile, corner mask, filter, recoloring or crop to small UI/browser
marks. Square/rounded presentations keep their designed background for large
README or gallery uses; transparent corners alone do not make a presentation a
foreground mark.

## Consumer audit

Both `README.md` and `docs/README.en.md` use the rounded presentation at 128 px
with centered title, introduction and navigation to the Hexly project archive
and the complete alternate-language README.

This repository publishes a library, not a branded application. There are no
sidebar/header, browser favicon, touch, platform-manifest or social-image
consumers. `src/login.ts` renders generic success/error SVG status symbols on
loopback OAuth callback pages; they convey authentication results and are not
product logos. They and the consumer-supplied accent color remain unchanged.

## Small-size limits

At 16 px the diagonal driver silhouette is readable, but knurling and resin depth disappear; at 24 px the collar is a small silver band.
