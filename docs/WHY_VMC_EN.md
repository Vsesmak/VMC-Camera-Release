# Why Mobile Cameras Hit a Wall and How I Rebuilt Google Camera in 97 Engineering Iterations

### The Architecture of VMC 8.8: Bypassing Manual Hex-Patching, Tapping Hexagon HVX & Sabre MTK, and Controlling Sensor Physics Live from the Viewfinder

**By Vseslav Makarov (VSEMAK)**  
*Creator and Lead Developer of VMC (Vseslav Makarov Camera)*

---

## Prologue: The Crisis of Flagship Mobile Imaging

Modern smartphones ship with remarkable camera hardware: 1-inch sensors (such as Sony LYT-900), variable mechanical apertures, optical periscope telephotos, and dedicated DSP/NPU accelerators.

Yet examine photos from most stock OEM camera apps and you witness the same disconnect between optical capability and computational output:

1. **Destructive Spatial Denoising:** Multi-pass spatial smoothing wipes out foliage, asphalt microtextures, fabric weave, and natural skin pores, leaving plastic watercolor output.
2. **Aggressive Unsharp Masking (Halo Artifacts):** To fake lost detail, stock ISPs apply wide sharpening halos around contrast boundaries.
3. **Flattened Atmosphere & Clipped Signage:** Stock tone mapping either crushes shadow atmosphere or leaves neon signs and highlights blown out, while hiding genuine optics controls inside clumsy Pro modes.

---

## The Impasse of Traditional GCam Modding

Google Camera ports brought HDR+ multi-frame RAW merging to non-Pixel devices, but configuring them turned into friction:
* **Menu Labyrinths:** Adjusting sharpness or denoise sliders in LibPatcher meant leaving the viewfinder, diving through nested submenus, tweaking blindly, and restarting repeatedly.
* **Fragile Manual Hex-Patching:** Advanced fixes inside `libgcastartup.so` (highlight behavior, Sabre MTK initialization on MediaTek, tone tables) historically required manually patching machine-code offsets in binary editors—where a one-byte mistake caused an instant crash.

Photography happens in a split second. You cannot compose cleanly when a wall of menus and binary patches stands between your eye and the sensor.

---

## The Genesis of VMC: 97 Engineering Iterations

The **VMC (Vseslav Makarov Camera)** project began from the **LMC 8.8 base (Google Camera 8.8.224 fix by Hasli)** with one clear goal:
> **Eliminate manual hex-patching and buried XML guesswork.** Expose genuine sensor physics, hardware-matched calibration, and live NEON post-processing directly in a clean Dual-Corner Viewfinder HUD.

Reaching build **v97** took months of low-level engineering across Java, native ARM64 C (`makcam_turbojpeg.c`, `makcam_hvx_probe.c`), and Smali bytecode, verified across crash dumps and live device testing from Android 10 through Android 15.

Every photograph captured with VMC embeds an authoritative EXIF Software tag: `Camera by Vseslav Makarov`.

---

# 1️⃣ THE DUAL-CORNER VIEWFINDER HUD

VMC transforms the viewfinder into an ergonomic **Dual-Corner Control Deck**, automatically localized across all APK system languages via **`HudI18n` (192 keys)**:

```
┌────────────────────────────────────────────────────────────────────────┐
│ [● HDR++ · ready] [AI Dynamic ▾] [9.x HDR | P3 ▾]  [HEX / Mediatek]    │ ➔ Top-Right HUD:
│                                                                        │   Platform, Tonemap,
│                                                                        │   sRGB/P3 + Vibrance,
│                            LIVE VIEWFINDER                             │   AWB, Bracketing,
│                                                                        │   Denoise, Configs &
│ [EV 0.0] [AF Auto] [f/1.63]                                            │   Per-Lens AutoCalibrator
│         (0.6x)   (1.0x)   (2.0x / ISZ)                                 │ ➔ Bottom-Left HUD:
└────────────────────────────────────────────────────────────────────────┘   EV, Focus & Aperture
```

### Bottom-Left Corner: Direct Optics & Exposure Control
* **EV Switcher + EVDialView:** One-tap exposure offset (`-0.7` to `+0.7`) plus circular dial precision control.
* **Focus Switcher:** Quick toggle between `AF Auto`, `Macro`, `Infinity / Hyperfocal`, and `Manual Focus (MF)` with diopter readout.
* **Aperture Switcher (`f/1.63 – f/4.0`):** Hardware control for variable-aperture flagships (Xiaomi 14 Ultra / 13 Ultra) with 9-step SeekBar precision.
* **In-Sensor Zoom (`1.0x / 1.2x / 1.5x / 2.0x`):** High-resolution sensor crop presets on supported flagships without phantom duplicate lens buttons.

### Top-Right Corner: Real-Time Pipeline & Calibration Deck
1. **Dynamic Platform Indicator (`HEX` / `CPU` / Purple `Mediatek`):**
   * **`HEX`:** Confirms Qualcomm Hexagon HVX DSP acceleration via `libmakcam_hvx_probe.so`.
   * **`CPU`:** Transparent fallback readout when DSP offloading is unavailable.
   * **`Mediatek`:** Dedicated purple badge shown when **DeviceSetup** detects a MediaTek SoC and **AutoHighlightPatch** activates **Sabre MTK** safely (`0x1F757F8` -> `28008052`, `0x1F75DB0` -> `08028052`, byte-verified against `680a40b9`).
2. **HDR+ Enhanced Readiness & Gyroscope/OIS Shutter Guard:** Tracks HDR++ readiness and dynamically balances handheld vs. tripod exposure duration while unconditionally preserving **Shasta** (`28008052`).
3. **Adaptive ISO Denoise Control:** Quick presets and adaptive ISO tiering (`<400`, `400–1600`, `1600+`) adjusting spatial and temporal smoothing.
4. **Honest Tonemap Switching + `Cupertino SmartTone`:**
   * Selecting native GCam tonemaps writes `_StartTone` (`0xdbc290`) and `_StartGamma` (`0xdbc318`) directly and cleanly restarts the camera session so LUT changes truly affect rendered frames.
   * **`8. Cupertino SmartTone`:** Executes instantly in ARM64 NEON at save time with zero session restart.
5. **Color Gamut (`sRGB` / `DCI-P3`) & Live `Chroma Vibrance`:**
   * Switches standard `sRGB` vs. wide-gamut `Display P3` (writing EXIF Tag `0xA001 = 0xFFFF` and Display P3 ICC chunks preserved by TurboJPEG).
   * Pairs directly with the live **Chroma Vibrance** toggle (`Off` / `VMC Natural +22%` / `Cupertino Rich +38%`).
6. **AWB & Deep Color Calibration:**
   * Instant switching between `Auto`, `Google Pixel AWB`, `Sony IMX686`, Kelvin presets, `Config XML`, and `Native Camera2 ISP` relying genuinely on hardware `SENSOR_NEUTRAL_COLOR_POINT` (with artificial R-channel `clampAwb()` floors completely removed).
   * Paired with **Hardware Black Level** (`64 / 32 / 0 / XML`) and **Perceptual System Color Vibrance** (`rowSum = 1.0`, eliminating AUX double-saturation).
7. **Parametric Exposure Bracketing & Config Switcher:** Built-in XML styles (`AI Dynamic`, `ClassicStyle`, `FilmStyle 5.1`, `AI Dynamic · GCam 5.1`), custom `/sdcard/LMC8.8/` XMLs, and Auto / preset / manual bracketing series.

---

# 2️⃣ LOW-LEVEL NATIVE ENGINE: `libmakcam_turbojpeg.so` (NEON SIMD)

Instead of letting legacy baseline JPEG encoding clip highlights or wash out colors—and instead of touching fragile closed C++ RAW buffers blindly—VMC routes baseline JPEG output through **`libmakcam_turbojpeg.so` (libjpeg-turbo 3.2.1 ARM64 NEON SIMD)** in `TurboJpegCodec`:

* **Selective Soft-Highlight Knee (`luma > 212`):** Uses fast NEON max-channel block detection (`vcgt_u8`) so pixels at `<= 212` stay completely untouched, gently compressing only bright neon signs and specular highlights while preserving night shadow atmosphere.
* **Non-Linear Chroma-Gated Vibrance (`chroma > 12`):** Boosts mid-saturation colors while leaving neutral asphalt, concrete, and shadows at **0% boost**, combined with skin-tone protection.
* **Zero Metadata Loss:** Preserves Display P3 / sRGB APP2 ICC profiles and full EXIF blocks while applying lossless Huffman table optimization.

---

# 3️⃣ HARDWARE-FIRST PER-LENS AUTOCALIBRATOR (ZERO SCENE BIAS)

Earlier scene-feedback auto-tuners that learned bias from saved JPEGs were completely retired in v96 in favor of a deterministic **Hardware-First 5-Step AutoCalibrator**:

1. **Universal Physical Camera Profiling (`<4 MP` Filtering):** Probes Camera2 IDs across brands (`Main`, `Ultrawide`, `Tele`, `Periscope`, `Front`), records exact megapixels and two-decimal sensor pixel pitch (`%.2f µm`), and excludes depth/macro stubs `< 4 MP` and virtual duplicates.
2. **Smartphone-Only Compact Model Catalogs:** Matches every lens cleanly to curated mobile-only databases — **429 Noise Models** + **45 Color Matrices** (all DSLR/mirrorless entries stripped).
3. **Optics-Aware Detail Preservation:** Upgrades `"0"` sharpness keys to `"1"`, **never lowers higher user/XML settings** (`lib_sabre_sharp_contrast_key* > 1`, `lib_polysharp_rad_med_key* > 1`), and preserves `lib_shasta_force_key*` (`28008052`).
4. **Validated OpMode & Cross-Brand Quarantine:** Assigns verified `Session OpMode` values (e.g. `61448` on K30 Pro Zoom OIS tele, Realme/OPPO/OnePlus profiles), quarantines incompatible modes on `onConfigureFailed`, and protects non-Xiaomi devices from invalid `fix_resolution_key=37`.
5. **Per-Lens Control & Zero-Lag Cold-Start Cache:** Lets users inspect `Applied` vs. `Kept` calibration details and independently toggle AutoCalibrator **ON/OFF per sensor**, while unconditionally caching `device_hardware_profile.json` so subsequent launches skip discovery completely.

---

# 4️⃣ CROSS-VERSION PLATFORM STABILITY (ANDROID 10 – 15)

* **Android 10 (`whyred` / SDK 29):** Added backward-compatible `WindowInsets` inset handling in `ikp.smali` replacing API 30+ `WindowInsets$Type.navigationBars()`.
* **Android 11 (`davinci` / SDK 30):** Guarded `Build.SOC_MODEL` / `SOC_MANUFACTURER` lookups via `DeviceSetup.safeSocModel()` across all telemetry, TFLite, QNN HTP, and LegacyDSP classes.
* **Legacy Exynos / Samsung & Multi-Brand AUX:** Purged `.getClass()` NPE triggers in `dzp.smali` and shipped **7 OEM-whitelisted Package IDs** (`vmcamera`, `snapcam`, `aweme`, `qcamera3`, `ruler`, `scan3d`, `glens`) under a single signature and 16 KB page alignment (`zipalign -P 16 4`).

---

## Hardware & OS Compatibility

* **OS:** Android 10, 11, 12, 13, 14, 15 (`arm64-v8a`)
* **SoCs:** Qualcomm Snapdragon 865–8 Gen 3 (Hexagon HVX + QNN HTP), MediaTek Dimensity / Helio (Sabre MTK), Samsung Exynos / Google Tensor
* **Latest Release (v97):** [Download VMC 8.8 v97 on GitHub Releases](https://github.com/Vsesmak/VMC-Camera-Release/releases/tag/v97)
* **GitHub Repository:** [Vsesmak/VMC-Camera-Release](https://github.com/Vsesmak/VMC-Camera-Release)

---

*VMC (Vseslav Makarov Camera) is built for photographers and developers who demand transparent hardware control and uncompromised optical depth.*
