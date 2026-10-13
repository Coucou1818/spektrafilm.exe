# NOTICE: provenance, copyright, and licences

**The Dyeworks** is a modified version of **spektrafilm** by Andrea Volpato.

This file exists because `LICENSE` is the verbatim GNU GPL v3 text and says
nothing about *this* program: who wrote which part, what was changed, and
which of the bundled pieces carry terms of their own. GPL-3 §5(a) requires a
modified work to carry prominent notices stating that it was modified and the
date; that is section 2 below.

> Not legal advice. This file records what the repository and the installed
> packages actually say, so that a reader, or a lawyer, can check it.

---

## 1. Upstream work

| | |
|---|---|
| **Original program** | spektrafilm |
| **Author** | Andrea Volpato |
| **Source** | https://github.com/andreavolpato/spektrafilm |
| **Licence** | GNU General Public License v3 (see `LICENSE`) |
| **Fork point** | `3bb2c2d`, "fix: vlog midgray exposure", 7 June 2026 |

The spectral film model is Andrea Volpato's work and remains his. That
includes the physical pipeline (spectral upsampling, the density-curve
family, the coupler and halation models, the print and scan chains) and the
`spektrafilm`, `spektrafilm_gui` and `spektrafilm_lut_creator` package names,
which are kept deliberately so that future merges from upstream remain
possible (`src/spektrafilm_gui/branding.py` records that decision).

Upstream's copyright notices are preserved wherever they appear. Nothing in
this fork removes or alters them.

`docs/PROVENANCE.md` is the machine-checked, per-component provenance table
behind this attribution. It is kept current by `tools/gen_provenance.py`, not
by hand. It is the gate-5 licence item this file's section 7 flags as pending.

---

## 2. Modification notice (GPL-3 §5(a))

This program is a **modified version** of spektrafilm.

| | |
|---|---|
| **Modified by** | Celian Le Bolloch (Natcelian), celian.lebolloch@proton.me |
| **Modifications from** | 7 June 2026 |
| **Most recent modification** | 7 September 2026 |
| **Product name of the modified work** | The Dyeworks |

Measured against the fork point, in the repository this notice ships with:

* **193 files added** under `src/` that do not exist upstream;
* **101 upstream files modified**;
* **+75,928 / −3,447 lines of Python** under `src/`;
* upstream had 181 `.py` files at the fork point; this tree has 578.

The engine core remains recognisably Andrea Volpato's and is modified rather
than replaced. The GPU compute layer (`src/spektrafilm/gpu/`), the Qt
interface work, the tooling under `tools/`, and the test suite are
substantially this fork's additions.

Twenty-three files that both projects edit carry a **fork contract** header
naming, line by line, which side owns what and how a conflict resolves. The
convention is described in `docs/reports/2026.08.20_fork_contracts.md`, and it
is the most precise per-file record of provenance in the project.

---

## 3. Copyright

```
Copyright (C) 2026 Andrea Volpato
    the original spektrafilm program and the spectral film model.

Copyright (C) 2026 Celian Le Bolloch (Natcelian)
    modifications and additions made in this fork ("The Dyeworks").
```

Copyright in a derivative work covers the contributions of its author, not
the underlying work. Both notices apply together; neither replaces the other.

---

## 4. The three tiers

This project is **not** under a single licence. Three distinct bodies of work
travel together, and each keeps its own terms.

### 4.1 Program code: GPL-3

All source under `src/`, `tools/` and `tests/`, both upstream's and this
fork's, is licensed under the **GNU General Public License, version 3**. The
full text is in `LICENSE`.

What that obliges anyone distributing a binary of this program to do:

1. keep the copyright notices in section 3 intact;
2. state that the work is modified and when, per section 2;
3. license the whole work under GPL-3;
4. provide the **Corresponding Source for the exact binary shipped**, which
   for a PyInstaller build includes `Spektrafilm.spec`, since it is the script
   that controls the build;
5. include a copy of `LICENSE`;
6. not impose further restrictions.

### 4.2 Film profiles and LUTs: CC BY-SA 4.0

The spectral profiles under `src/spektrafilm/data/profiles/`, and LUTs derived
from them, are Andrea Volpato's and are licensed **CC BY-SA 4.0**, not GPL-3,
by his deliberate choice. His reasoning is in `SPEKTRAFILM_LICENSE.txt`
(shipped in full, unmodified, and also bundled at
`src/spektrafilm/data/license/`): the GPL's software-shaped notion of a
derivative work fits a LUT poorly.

Custom stocks created in this program are recipes referencing a base profile,
and carry an auto-stamped CC BY-SA derived-metadata field so that a `.sfstock`
file states its own descent.

### 4.3 Custom LUT terms

`SPEKTRAFILM_LICENSE.txt` also carries terms specific to LUTs. It is shipped
verbatim and must not be modified; changes to profiles or LUTs belong in a
separate `CHANGELOG.txt` beside them. That file ships at the root of the
application folder next to `SPEKTRAFILM_LICENSE.txt`, and lists which profiles
are Andrea Volpato's and untouched, and which this fork added.

---

## 4.4 Other projects this fork learned from or adapted

Distinct from section 5, which lists code *shipped inside the binary*. The
projects below are **not bundled**; they were read, and in some cases adapted
from, while building this fork. Each is verified as absent at the fork point,
so each is this fork's attribution to make rather than something inherited
from upstream.

The distinction that matters legally is **code** versus **method**, and it is
stated per project rather than blurred.

### NegPy: MIT, and the one with a real obligation

```
Copyright (c) 2023 NRG Systems, Inc.
```
Licensed MIT, whose terms require the copyright notice AND the permission
notice to travel with copies or substantial portions. Citing it is therefore
not enough, so the full text ships with this program at
**`licenses/NegPy-MIT.txt`** and is bundled into the binary.

What this fork took, adapted rather than copied verbatim:

* `src/spektrafilm/gpu/device.py`, the wgpu adapter/device manager, its own
  docstring: *"Adapted from `references/NegPy/.../infrastructure/gpu/device.py`
  (PyQt6 app), stripped of any Qt coupling"*. The tiling ladder's shape and
  the persistent-readback-staging pattern follow it too.
* `src/spektrafilm_gui/roll_session.py`, *"adapted from NegPy's
  WorkspaceSession"*; the roll/batch model and its export-task payload.
* `src/spektrafilm/model/scan_finishing.py:49`, NegPy's **default
  dye-crosstalk unmixing matrix**, which is data taken directly.
* Smaller borrowings named in place: the batch-analysis EV semantics, the
  flip-H toggle, the crop rubber-band interaction, two canvas swatches.

### spektrafilm.rs: GPL

The Rust implementation, by **turbasvin**
(<https://github.com/turbasvin/spektrafilm-rs>).
`src/spektrafilm/runtime/params_builder.py:288`
records Phase 12A's B&W branch as **"line-for-line from `spektrafilm.rs`
`pipeline.rs:68-93`"**, and `params_schema.py:188` mirrors its `params.rs`.
The B&W profiles were converted from its `data/profiles`.

This is code derivation, not method borrowing. Its GPL licence is compatible
with this program's GPL-3; the obligation is attribution and share-alike,
both satisfied here.

### darktable: GPL-3, approach only

`src/spektrafilm_gui/scopes.py:7` is explicit: **"PORTED FROM darktable's
approach, not its code."** The waveform and vectorscope follow how darktable
presents those instruments, and the harmony-overlay offsets follow its
`harmony_type`. No darktable source is copied or shipped.

### vkdt: architectural reference

Named in `CLAUDE.md` as the secondary reference for the GPU node graph, and in
`src/spektrafilm/gpu/resident.py` as one of the "fast references" whose
architecture the resident path proves out. **No code and no data taken**: a
search of `src/` finds no vkdt-derived material.

### The spektrafilm OFX plugin: behavioural reference

Phases 11 and 14 mirror the OFX plugin's *behaviour*: the grain model tiers,
the `grainAmount`/`grainSaturation` trim (manual §4.7), the preset browser's
shape, worked out from `references/manual.pdf` and the plugin's observable
output. Its `.spkpreset` payloads are obfuscated and were **not** decoded; the
finding is recorded in `presets/output/CONVERSION_FINDINGS.md`.

### Hanatos 2025: upstream's, not this fork's

The Hanatos 2025 spectral-upsampling adaptation appears in 38 files **at the
fork point**. It is Andrea Volpato's use of that method and is covered by
section 1. Named here only so that its absence from this list is not mistaken
for an omission.

---

## 5. Bundled third-party components

Read from the installed distribution metadata of the environment that produces
the release build, not from memory. Versions are those of the 2026-10-12
build.

| Component | Version | Licence | Notes |
|---|---|---|---|
| PySide6 (+ Essentials, Addons) | 6.11.2 | LGPL-3.0-only OR GPL-2.0-only OR GPL-3.0-only | See §5.1 |
| shiboken6 | 6.11.2 | LGPL-3.0-only OR GPL-2.0-only OR GPL-3.0-only | PySide binding layer |
| qtpy | 2.4.3 | MIT | Qt abstraction shim |
| wgpu | 0.31.1 | BSD-2-Clause | Bundles `wgpu_native-release.dll` |
| rendercanvas | 2.7.2 | BSD-2-Clause | |
| numba | 0.67.0 | BSD-2-Clause | |
| llvmlite | 0.49.0 | BSD-2-Clause AND Apache-2.0 WITH LLVM-exception | |
| numpy | 2.5.3 | BSD-3-Clause AND 0BSD AND MIT AND Zlib AND CC0-1.0 | |
| scipy | 1.18.1 | BSD-3-Clause | |
| scikit-image | 0.26.0 | BSD-3-Clause | |
| colour-science | 0.4.7 | BSD-3-Clause | |
| Pillow | 12.3.0 | MIT-CMU | |
| imageio | 2.37.4 | BSD-2-Clause | |
| OpenImageIO | 3.1.17.0 | Apache-2.0 | |
| rawpy | 0.26.1 | MIT | **Bundles LibRaw, which is LGPL-2.1**, see §5.2 |
| python-exiv2 | 0.18.1 | GPL-3.0-or-later | Compatible; reinforces the GPL-3 obligation |
| psutil | 7.2.2 | BSD-3-Clause | |
| PyInstaller | 6.20.0 | GPL-2.0 with a bootloader exception | Build tool; its exception permits shipping the bootloader with any licence |
| Bricolage Grotesque, Geist, Geist Mono, Inter, Chivo Mono, Archivo, Azeret Mono, Arimo, Noto Sans Mono | n/a | SIL OFL 1.1 | Bundled typefaces, see §5.3 |

### 5.1 PySide6 is LGPL: the obligation that is easy to miss

PySide6 is used under the **LGPL-3**. LGPL requires that a recipient be able
to replace the LGPL component with a modified version and relink the
application. A PyInstaller one-folder build satisfies this far more readily
than a one-file build, because the Qt libraries remain replaceable files in
`_internal/`. **The release build is one-folder** (`dist/TheDyeworks/`), which
is the right shape for this and should stay that way.

### 5.2 LibRaw

`rawpy` is MIT, but the LibRaw library it wraps and ships is **LGPL-2.1** (a
copy travels in the wheel as `LICENSE.LibRaw`). The same relinking
consideration as §5.1 applies to it.

### 5.3 Fonts: OFL, bundled, and settled

| File | Family | Copyright |
|---|---|---|
| `Archivo-400/500/600.ttf` | Archivo | 2020 The Archivo Project Authors |
| `AzeretMono-400/500.ttf` | Azeret Mono | 2021 The Azeret Project Authors |
| `Arimo-400/500.ttf` | Arimo | 2020 The Arimo Project Authors |
| `NotoSansMono-400.ttf` | Noto Sans Mono | 2022 The Noto Project Authors |
| `BricolageGrotesque-400/500/600.ttf` | Bricolage Grotesque | 2022 The Bricolage Grotesque Project Authors |
| `Geist-400/500/600.ttf`, `GeistMono-400/500.ttf` | Geist, Geist Mono | 2024 The Geist Project Authors |
| `ChivoMono-400/500.ttf` | Chivo Mono | 2018 The Chivo Project Authors |
| `Inter-400/500/600.ttf` | Inter | 2016 The Inter Project Authors |

All of them are under the **SIL Open Font License 1.1**, which permits bundling
and redistribution inside an application and requires its notice to travel
with the fonts. It does:
`src/spektrafilm_gui/assets/fonts/OFL-NOTICE.md` sits beside the `.ttf`s and
is collected into the build with them. That file carries the per-family
detail and the reasoning for the choice; this table exists so a reader of
the licence notice does not have to already know it is there.

**GT America and FK Grotesk SemiMono are NOT bundled and never were.** They
were the design's intended faces and are trial licences; they were removed
outright on 2026-09-03 (`design/DECISIONS.md`) rather than shipped
unlicensed, and a test enforces that the trial files cannot re-enter the
build. Nothing here is waiting on a font purchase.

---

## 6. Reproducing the release binary

Corresponding Source for a distributed binary means the source *and* the
scripts that control its build:

* this repository at the tagged commit;
* `Spektrafilm.spec`, the PyInstaller spec;
* `pyproject.toml`, the dependency set;
* the build command: `python -m PyInstaller --clean --noconfirm Spektrafilm.spec`.

`tools/frozen_smoke.py` verifies that a build carries what it must.

---

## 7. Known gaps

Recorded rather than left implicit.

* **Two earlier gaps are closed** (checked against the repository on
  2026-09-25). `README.md` was upstream's until commit `eef2b668`; it is now
  this fork's own. It says the model is Andrea Volpato's, links the upstream
  repository, and its donation link is his and is labelled as funding
  spektrafilm, not this fork. `pyproject.toml` lists Andrea Volpato first and
  Celian Le Bolloch (Natcelian) second as authors. Its package version `0.3.4`
  is the ENGINE's, Andrea's, and stays so on purpose: the product version,
  `2.1.0`, lives in `spektrafilm_gui/branding.py` (`APP_VERSION`), and the
  interface shows both.
* The bundled-component table covers **distributions**, not every transitive
  native library inside them. LibRaw is called out because it is the one that
  materially changes an obligation; a deeper sweep is warranted before a
  public release.
