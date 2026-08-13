# ROMs served to the Project64 play page

These three files are the C64 system ROMs that
[`play.html`](https://github.com/nschneir/Project64) fetches at runtime from

```
https://nschneir.github.io/vc64web.github.io/roms/
```

They are **free/libre re-implementations** from the MEGA65 **open-roms** project. No
Commodore ROM is present in this repository, and none is needed: all four Project64
demos boot and play on this set, loaded from `.prg`.

**There is deliberately no 1541 drive ROM here.** The demos are loaded as `.prg`, which
vc64web flashes directly into RAM and auto-`RUN`s — the drive is never involved. The
`.d64` load path *does* require a drive ROM, and no free 1541 DOS exists, so that path
is simply not used by the play page.

## Provenance

Upstream project: **MEGA65 open-roms** — <https://github.com/MEGA65/open-roms>

Downloaded 2026-08-13 from `https://raw.githubusercontent.com/MEGA65/open-roms/master/bin/`,
with `master` at commit `ad178dbe4d48cd6a317737a8e0e7e662f7e33d32` (2026-05-16).
The binaries themselves are older than that tip — the per-file commit below is the last
commit that modified each one.

| file here | upstream file | size | sha256 | last upstream commit to touch it |
|---|---|---|---|---|
| `kernal.rom` | `bin/kernal_generic.rom` | 8192 | `88e86ed3d0c710edab8f90ad146faa8de1ead11f43494b176c7b54724ca721c6` | `5192c683a098204e835b8c5529ecd24c808eba92` (2021-08-23) |
| `basic.rom` | `bin/basic_generic.rom` | 8192 | `54a1464b4b27c9dc61bbd62a818fdd12ec99af9089111005a5add0ad0e6bd5ec` | `5192c683a098204e835b8c5529ecd24c808eba92` (2021-08-23) |
| `chargen.rom` | `bin/chargen_pxlfont_2.3.rom` | 4096 | `bc5ed24e8e694543f0229800d050acff86d9674aec6dbd95055a26e824d8a395` | `b9661811579418f52d201d26fc4eca3e7f0e9c4f` (2020-03-06) |

Note the rename: the files are stored under the plain names `kernal.rom` / `basic.rom` /
`chargen.rom` because that is what the play page's `ROM_BASE` URLs address. The
`kernal.rom` and `basic.rom` here are the **generic** (plain C64) builds, not the
Ultimate64, MEGA65 or `_crt` variants.

**Build identifier.** The KERNAL and BASIC binaries self-identify at boot as
`OPEN ROMS GENERIC BUILD / RELEASE DEV.210823.FC.1`; the string `DEV.210823.FC.1` is
present verbatim in `kernal.rom`. That is the version this deployment is pinned to.

**CHARGEN choice.** `chargen_pxlfont_2.3.rom` was chosen over the alternative
`chargen_openroms.rom` because it is the closer visual match to the demos' committed
reference screenshots. This matters: snake, invaders and ms-muncher each bank in the
character ROM at `$D000`, copy all 2 KB into RAM and then patch only their own glyphs,
so every letter, digit and punctuation mark on their screens comes from whichever
CHARGEN is installed. (la-galaxia builds its whole set in RAM and is CHARGEN-independent.)
Upstream's `bin/README.md` explicitly permits mixing CHARGEN across builds — it is the
one documented exception to its "don't mix ROMs from different builds" rule. BASIC and
KERNAL here are from the same build, as that rule requires.

## Licence

All three files: **GNU LGPL-3.0-or-later**. The full upstream licence text is vendored
alongside them as [`LICENSE-open-roms.txt`](./LICENSE-open-roms.txt).

> Copyright Paul Gardner-Stephen, 2019.
> Copyright Roman Standzikowski (FeralChild64), 2019-2021.
>
> This program is free software: you can redistribute it and/or modify it under the
> terms of the GNU Lesser General Public License as published by the Free Software
> Foundation, either version 3 of the License, or (at your option) any later version.

Additional terms that apply to parts of this set:

- **`basic.rom`** — some of the BASIC source files it is built from are additionally
  under the **MIT licence, © Microsoft Corporation** (from Microsoft's 2020 MS-BASIC
  release). Upstream's `LICENSE` states: *"Some BASIC files (noted in the specific
  files) are subject to this license and copyright: MIT License — Copyright (c)
  Microsoft Corporation."* The MIT text is included in `LICENSE-open-roms.txt`.
- **`chargen.rom`** — the PXL font is the work of **Retrofan**, included in open-roms
  with permission under the LGPL-3.0. Upstream's `bin/README.md`: *"PXL font was created
  by Retrofan, we got a permission to include it with Open ROMs under GNU Lesser General
  Public License 3.0."*

**Corresponding source.** The complete source for these binaries is the open-roms
repository at the commits pinned in the table above:
<https://github.com/MEGA65/open-roms>.

## Known risk in this pin

`bin/` is an **untagged 2021 development build**. The open-roms repository has no
releases and no tags; the binaries self-report `DEV.210823.FC.1`, i.e. 2021-08-23. The
project is still active (last push 2026-05-16), but there is **no release cadence to
track**, and `master`'s `bin/` is a moving target rather than a versioned artefact.

The mitigation is what this directory does: the bytes are vendored here and pinned by
sha256, so an upstream rebuild of `bin/` cannot silently change what the play page runs.
Anyone refreshing these files should re-verify the hashes above and re-check the demos —
in particular the glyph rendering, given three of the four demos take their text font
from CHARGEN.

Also worth knowing: the demos' own test suites run under VICE against the real Commodore
ROMs, so **CI does not exercise the ROM set this page actually ships on**. A future demo
change that started calling a KERNAL routine could pass CI and still break here. The
current demos do not call the KERNAL at all — they take the machine over with their own
IRQ, charset and direct VIC writes, and depend on the ROMs only for the boot path
(banner → auto-typed `RUN` → the BASIC `10 SYS 2061` stub).
