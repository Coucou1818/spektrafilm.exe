# Spektrafilm.exe

A portable, standalone Windows build of Andrea Volpato's **spektrafilm**, packaged as a double-clickable application with no Python installation required. This repository exists to make spektrafilm's spectral film-simulation GUI accessible to Windows users who do not want to set up a Python environment, `uv`, or build the application themselves.

This began as a packaging project. All of the simulation science and the photographic model are Andrea Volpato's work. This repository contributes the Windows freezing, the launcher, the fixes required to make the application run correctly as a self-contained executable, and — as of v.b.0.1 / v.b.0.2 — **GPU acceleration** of the pipeline (via wgpu, with automatic CPU fallback), a dedicated **export panel**, **batch editing** (roll session, film strip, batch export, roll normalization), and import-stage conveniences. The simulation model itself is never modified: every GPU kernel is validated against the CPU reference within documented tolerances (122 automated tests), so the physics stays Andrea's, just faster.

---

## Relation to Andrea Volpato's spektrafilm

spektrafilm is a research project by **Andrea Volpato**: an end-to-end, physically based spectral simulation of the analog photographic pipeline. It models how film-stock data, dye couplers, enlarger settings, grain, halation, and scanning shape a final image, grounded in published spectral measurements rather than a generic "film look."

- Upstream project: https://github.com/andreavolpato/spektrafilm
- Development discussion: https://discuss.pixls.us/t/spectral-film-simulations-from-scratch/48209

This build tracks upstream **v0.3.2**. It does not modify the simulation model: the source-level changes are the freezing fixes (documented in the Log below) plus a GPU compute backend and GUI additions that compute the same results faster — each GPU kernel is parity-tested against the CPU path it replaces. For the meaning of every control, the methodology, and the model itself, refer to Andrea Volpato's repository and the pixls.us thread. If you find this useful, please consider supporting Andrea's work directly.

---

## Download and Releases

Builds are published under the **Releases** tab of this repository.

- Download the latest `Spektrafilm-win64.zip` from Releases.
- Extract the entire folder anywhere you like (Desktop, Documents, a USB drive).
- Run `Spektrafilm.exe` from inside the extracted folder.

The application is distributed as a one-folder build: a folder containing `Spektrafilm.exe` alongside an `_internal` directory holding the bundled Python runtime and dependencies. **Keep the folder intact.** Do not move `Spektrafilm.exe` out of the folder on its own; it needs the `_internal` directory beside it. You may create a desktop shortcut to the exe.

### Releases Log

- **v.b.0.2** (2026-07-07) — **Batch editing release.** Work in rolls, not frames: a **Roll session** (multi-select import of mixed RAW/RGB files, per-frame looks that swap with the frame, position counter and navigation, "Apply look to all" reference-look propagation), a **film strip** of unedited source thumbnails under the image (embedded camera JPEG previews — instant; click to switch frames; hidden for single-image work), **batch export** ("Export roll": every frame rendered full-precision with its own look into a chosen folder, per-frame progress, cancel between frames, no-overwrite naming, sync-export-settings toggle, failing frames reported while the batch continues), and **roll normalization** ("Analyze roll" measures every frame's auto-exposure EV with outliers discarded; "Use roll exposure" stamps one shared EV on the roll so frames keep their relative brightness — re-applying replaces rather than compounds, and manual tweaks on top are preserved). All on the v.b.0.1 GPU pipeline; 122 automated tests. Based on upstream spektrafilm v0.3.2.
- **v.b.0.1** (2026-07-06) — **GPU acceleration release.** The spectral pipeline now runs on the GPU via [wgpu](https://github.com/pygfx/wgpu-py) (Vulkan/DX12) with automatic CPU fallback when no compatible GPU is present. Full 24 MP scan: **41 s → 9.4 s** on the test machine (RTX 4070 SUPER); full-precision export: 4+ minutes → **10.3 s**. Every GPU kernel is validated against the CPU reference (102 automated parity tests; the simulation math is unchanged). Also new: a dedicated **Export panel** (8-bit JPEG, true 16-bit TIFF/PNG from the float render, selectable color profile with embedded ICC, flat/linear or tonally-normal encoding, non-blocking with progress and cancel); **Import RGB white balance** (as-shot / daylight / tungsten / custom temperature+tint for TIFF/PNG/JPEG input) with auto re-import on setting changes; a live **GPU/CPU toggle** and backend status in the GUI. Based on upstream spektrafilm v0.3.2.
- **v.a.0.1** — First public Windows build. Based on upstream spektrafilm v0.3.2. PyInstaller one-folder build with splash screen and custom icon.

---

## Status

**Working Alpha — Windows x64 only.**

This is an early, experimental build. It runs the full spektrafilm pipeline and GUI on Windows without a Python environment. Expect rough edges. With a compatible GPU, previews are sub-second and a full-resolution scan takes on the order of ten seconds; without one, the application falls back to the CPU path automatically and full-resolution processing is slow by nature of the spectral model (see Performance). There is no installer; it is a portable folder. macOS and Linux are not provided here; users on those platforms should run upstream spektrafilm directly.

---

## Import Settings

These notes follow Andrea Volpato's guidance for the underlying application.

The simulation expects **linear, scene-referred** input, with or without a transfer function.

**RAW files** can be imported directly through the import-raw panel (RAW decoding is handled internally). The recommended manual workflow, per upstream, is to open RAW files in a tool such as darktable, deactivate the non-linear mappings (filmic or sigmoid), adjust exposure to preserve all information while avoiding clipping, then export a 32-bit float TIFF in linear ProPhoto RGB.

**Prepared linear images** can be loaded through the file loader. The loader imports 16-bit and 32-bit image files as new layers using OpenImageIO. PNG, TIFF, and EXR are known to work, and other formats may work too. For best results keep the image scene-referred and linear, ideally a 16-bit or 32-bit float TIFF/EXR in a wide-gamut color space such as linear Rec2020 or linear ProPhoto RGB.

**White balance on import.** Both the RAW importer and the RGB file loader offer white balance modes (`as shot`, `daylight`, `tungsten`, `custom` with temperature and tint). For RGB files the adjustment is a Von Kries chromatic adaptation applied in the linear light of the declared input color space, and changing any import setting automatically re-imports the loaded file. An optional "16-bit recovery" adds a small contrast and color-density punch to compensate the flatter look a display-encoded TIFF/JPEG has versus a RAW.

**Performance note.** With a compatible GPU, adjust values with **PREVIEW** (sub-second) and use **SCAN** for finals (~10 s at 24 MP). On CPU-only machines the old advice stands: the simulation is slow at full resolution, so work with PREVIEW and reserve SCAN for the final image.

For the full list of controls and their behavior, refer to the upstream documentation and the in-GUI tooltips.

---

## Performance

As of v.b.0.1 this build runs the heavy per-pixel spectral math on the **GPU** through [wgpu](https://github.com/pygfx/wgpu-py) (Vulkan/DX12 on Windows). Measured on the build machine (24 MP RAW, RTX 4070 SUPER):

| Render | CPU only | GPU |
| --- | --- | --- |
| Full scan | 36–41 s | **9.4 s** |
| Full-precision export (exact per-pixel spectra, LUT off) | ~4.2 min | **10.3 s** |
| Preview | seconds | sub-second |

How this stays faithful to Andrea's model:

- **Parity over speed.** Every GPU kernel is validated against the CPU reference it replaces, within documented tolerances (102 automated tests: strict per-pixel bounds for deterministic kernels, statistical moment/spectrum tests for the stochastic grain). The simulation math is not altered — it is the same computation on different silicon.
- **The CPU path is never removed.** If no compatible GPU is present, or wgpu fails to initialize, every stage falls back to the original CPU implementation automatically. A status label in the GUI shows the active backend, and a tick box lets you force CPU at runtime to compare.
- The GPU work includes a precision-refined software power function in the shaders (the hardware one is not accurate enough for the parity budgets) and "resident" chains that keep whole stages on the GPU between kernels instead of round-tripping through system memory.

numba's JIT remains active for the CPU path, so CPU-only machines still get the full native speed (reaching that took some trial and error — an earlier packaging route ran 2–4x slower because numba was disabled in the frozen build; see the Log).

spektrafilm remains computationally heavy by design: it is a full spectral simulation, not a LUT. Processing time scales with resolution and hardware. Use PREVIEW for adjustments and SCAN for finals.

---

## Build Requirements and Installation

This section is for anyone who wants to reproduce the build, not for end users (end users only need the Releases download).

The build was produced on Windows x64 with Python 3.13, using `uv` to manage the upstream spektrafilm tool environment, and PyInstaller to freeze it. Required components:

- A working install of upstream spektrafilm via `uv` (the `spektrafilm` and `spektrafilm_gui` packages and all their dependencies).
- A C/C++ toolchain is **not** required for this packaging route (PyInstaller bundles the existing CPython interpreter rather than compiling to C).
- PyInstaller, installed into the same environment as spektrafilm.

Install PyInstaller into the spektrafilm `uv` tool environment:

```powershell
uv pip install --python <path-to>\uv\tools\spektrafilm\Scripts\python.exe pyinstaller
```

Build command (run from the folder containing your `splash.png` and `icon.ico`):

```powershell
<path-to>\uv\tools\spektrafilm\Scripts\python.exe -m PyInstaller `
  --name Spektrafilm `
  --windowed `
  --noconfirm `
  --icon icon.ico `
  --splash splash.png `
  --collect-all spektrafilm `
  --collect-all spektrafilm_gui `
  --collect-all colour `
  --collect-all napari `
  --collect-all napari_builtins `
  --collect-all vispy `
  --collect-all numba `
  --collect-all llvmlite `
  --collect-all scipy `
  --collect-all dask `
  --collect-all rawpy `
  --collect-all OpenImageIO `
  --collect-all exiv2 `
  --collect-all imageio `
  --collect-all magicgui `
  --collect-all superqt `
  --collect-all app_model `
  --collect-all npe2 `
  --collect-all wgpu `
  --copy-metadata imageio `
  --copy-metadata numpy `
  --copy-metadata napari `
  --copy-metadata dask `
  --copy-metadata vispy `
  --copy-metadata scipy `
  --copy-metadata numba `
  --copy-metadata magicgui `
  --copy-metadata superqt `
  --copy-metadata npe2 `
  --copy-metadata app_model `
  --copy-metadata psygnal `
  --collect-submodules napari `
  <path-to>\uv\tools\spektrafilm\Lib\site-packages\spektrafilm_gui\app.py
```

Notes on the flags:

- `--collect-all numba` and `--collect-all llvmlite` are essential. They bundle numba and its LLVM backend (including the binary `llvmlite.dll`) so that numba's JIT compilation works at runtime. Without these, the hot numerical loops fall back to pure Python and the application runs several times slower.
- `--collect-all wgpu` is essential for GPU acceleration (v.b.0.1+). It bundles wgpu's native library (`wgpu/resources/wgpu_native-release.dll`) and package data. Without it the frozen application **silently falls back to CPU** — it still works, just without the speedup — because every GPU dispatch is written to degrade gracefully. The WGSL compute shaders themselves ride along with `--collect-all spektrafilm` (they are package data under `spektrafilm/gpu/shaders/`). Check the GPU status label in the GUI's bottom bar after building.
- `--copy-metadata` entries bundle each package's distribution metadata, which several dependencies (notably `imageio` and napari's plugin system) query at runtime via `importlib.metadata`. Missing metadata causes startup `PackageNotFoundError` failures.
- `--splash` provides the native loading image. The splash is dismissed from within the application code (see Log).

The output is `dist\Spektrafilm\`, a portable one-folder build. Zip that folder for distribution.

---

## Roadmap

- [x] Compilation to an `.exe` file
- [x] GPU-accelerated preview and scan (v.b.0.1 — wgpu, parity-tested, automatic CPU fallback)
- [x] Dedicated export panel with print-ready TIFF and web JPEG targets (v.b.0.1 — true 16-bit TIFF/PNG from the float render, selectable ICC profile, flat/linear or normal encoding, non-blocking with cancel)
- [x] Batch processing: apply a reference look across multiple frames, with propagation and normalization (v.b.0.2 — roll session, film strip, batch export, roll-average exposure normalization)
- [ ] Reworked UI/UX
- [ ] Side-by-side view: negative/positive or before/after comparison
- [ ] Additional basic controls for final post processing
- [ ] Whole-pipeline GPU-resident renderer (single upload/readback for the entire render; targeting a few seconds per full scan)
- [ ] Keeping the build in sync with upstream spektrafilm releases

Roadmap items are aspirational and not committed timelines.

---

## Log

A record of what was done to produce this build, including the dead ends, in case it helps anyone attempting the same.

**Initial attempts with Nuitka.** The first packaging route used Nuitka, compiling the Python sources to C. This ran into several obstacles in sequence:

- Nuitka's MinGW backend is not supported on Python 3.13, so MSVC was used.
- MSVC rejected the colour-science Munsell datasets with `error C2026: string too big`. Nuitka serializes large Python constants (the Munsell renotation tuples) into C string literals that exceed MSVC's literal size limit. As spektrafilm does not use the Munsell notation system, the two dataset modules (`colour/notation/datasets/munsell/all.py` and `real.py`) were replaced with minimal stubs exposing the same names as empty tuples, which let compilation proceed.
- The `--python-flag=no_site` and `no_warnings` flags stripped standard-library modules and were removed.
- `spektrafilm/utils/io.py` loads its CSV data (filters, ICC profiles, etc.) via `importlib.resources`, which resolves data directories as importable packages. This fails under a frozen app whose importer only knows compiled modules. A small shim was added to `io.py` so that `pkg_resources.files(...)` resolves data paths via `__file__` relative to the installed package instead, which works in both normal and frozen contexts.
- Even after the app launched, performance was 2–5x slower than running spektrafilm normally. The cause was numba: Nuitka's own documentation states that numba "is currently not working for standalone." With numba's JIT disabled in the frozen build, spektrafilm's hot loops ran in pure-Python fallback. This is not fixable via Nuitka flags, so the Nuitka route was abandoned for performance reasons.

**Switch to PyInstaller.** PyInstaller bundles the genuine CPython interpreter rather than compiling to C, so numba's JIT works normally and full processing speed is preserved. The `io.py` shim and the Munsell handling were kept (they are harmless under PyInstaller and the stubs are not needed for correctness, since PyInstaller never hits the MSVC string limit). The PyInstaller build required:

- `--collect-all` for the heavy data-driven dependencies (colour, napari, vispy, scipy, dask, rawpy, OpenImageIO, exiv2, imageio, magicgui, superqt, app_model, npe2) so their data files, binaries, and submodules are bundled.
- Explicit `--collect-all numba` and `--collect-all llvmlite` to preserve JIT performance.
- `--copy-metadata` for the packages that read their own distribution metadata at runtime, resolving a chain of startup `PackageNotFoundError` failures (imageio first, then several napari plugin-system dependencies).

**Splash and window behavior.** A loading splash image and a custom icon were added at build time. A small block was added to the application's `main()` to maximize the main window on launch and to dismiss the PyInstaller splash (`pyi_splash.close()`) once the main window is shown, wrapped so it is a no-op when running as ordinary Python.

**Result.** A portable, double-clickable Windows build that launches with a splash screen, opens maximized, imports and displays RAW files correctly, and runs at full numba-accelerated speed matching the native Python application.

**GPU acceleration (v.b.0.1).** The pipeline's per-pixel spectral math was moved to the GPU through wgpu, one stage at a time, each behind a dispatch hook with automatic CPU fallback and each validated by a parity test before the next stage was started. Highlights and dead ends, in case they help anyone doing similar work:

- *Kernels ported:* the 81-wavelength spectral integration (~40x alone), Gaussian and exponential blurs (mirroring the CPU's own FIR/IIR dispatch with bit-identical coefficients), density-curve interpolation, DIR-coupler correction, the stochastic grain (counter-based per-pixel RNG; statistical parity via moments and power spectrum, since a GPU cannot reproduce scipy's random stream), color matrices, the sRGB encode, and fused elementwise `10**x` / scale / `log10` tails.
- *Hardware pow is not accurate enough.* WGSL's `pow()`/`exp2(log2())` measured ~4.4e-4 error on the target GPU — far below the 1e-5 parity budgets. The fix is a software implementation in the shaders: mantissa/exponent split by bit twiddling, an atanh-series log2 carried as a double-single (hi+lo) float pair, the product done with fused multiply-adds, and a polynomial exp2. Measured ~3e-7 — this single piece unblocked every power-law transfer function.
- *Round-trips dominate.* Individually dispatched kernels pay a full image upload + readback each; chaining whole stages on-device ("resident" chains: develop, halation, grain, scan blur+unsharp) with one upload and one readback per stage was worth 4–7x per stage on top of the kernel speedups.
- *Two reference-implementation surprises:* `colour`'s sRGB→sRGB `RGB_to_RGB` is not a pure encode (its rounded spec matrices do not compose to identity, ~1e-4 perturbation — the GPU replicates it exactly for parity), and real film-profile density curves are not strictly monotone (~1e-4 measurement wiggles), which silently disabled an over-defensive dispatch gate until an end-to-end benchmark caught it.
- *Freezing:* `--collect-all wgpu` is required or the frozen build ships without `wgpu_native-release.dll` and silently runs CPU-only.

Net result at 24 MP on an RTX 4070 SUPER: scan 41 s → 9.4 s, full-precision export ~4.2 min → 10.3 s, previews sub-second, 102 parity tests green, CPU path intact.

**Batch editing (v.b.0.2).** With single-frame rendering at ~10 s, the missing piece was working across a roll. The session model is adapted from NegPy (per-file settings keyed in a session store, a selected index, look propagation) with one deliberate change: frames INHERIT the look currently being developed until they get a stamped one, so a roll naturally shares the look you are dialing in. The film strip uses the camera's embedded JPEG thumbnails (no demosaic — a 36-frame roll thumbnails in seconds) loaded on worker threads. Batch export pre-resolves every frame into an immutable job list on the GUI thread (path, import treatment, look, format), then a single worker renders and writes sequentially — cancel applies between frames, a failing frame is recorded and the batch continues. Roll normalization maps NegPy's Batch Analysis onto this pipeline's per-frame automatic: it measures each frame's auto-exposure EV, discards outliers beyond 1 EV from the median, and stamps the average as one shared exposure with per-frame bookkeeping so re-applies replace instead of compounding. In practice: add a roll → develop one frame → Apply look to all → Analyze roll → Use roll exposure → Export roll → a consistent set of 16-bit scans, unattended.

---

## License

This project is licensed under **GPL-3.0**, the same license as upstream spektrafilm, of which it is a derivative work. All credit for the simulation, model, and GUI belongs to **Andrea Volpato**. See the upstream repository for the original work: https://github.com/andreavolpato/spektrafilm
