# VMC 8.8 v98 — Release Notes (EN)

🌐 **English** | **[Русский](RELEASE_NOTES_v98.md)**

**Version:** `VMC 8.8 v98` (`versionCode: 66263640`)  
**Date:** November 27, 2026  

---

## What's New in v98

### 1. MediaTek Sabre RAW16 Fix, Exynos Support (`Exynos` Orange Badge) & Cross-Vendor GPU Offload
- **MediaTek Sabre MTK (`corot` / Dimensity / Helio):** Fixed the native C++ `SIGABRT` on RAW16 captures by patching `sabre_processor.cc` (`0x1f6f77c` bit-depth abort NOP + `0x1F757F8` -> `28008052` + `0x1F75DB0` -> `08028052`), clearing `pref_hexagon_key=0` on MediaTek, and enabling a universal `1.0x / 1.2x / 1.5x / 2.0x` zoom toggle across Main and `1.0x / 2.0x` on AUX lenses.
- **Samsung Exynos Initialization & Sabre Activation:** Fixed the inverted `pref_exynosfix_key` (`"-1"` = Fix On instead of `"1"` = Fix Off) across all lens slots (`"", _2.._5, _front`), cleared `pref_hexagon_key=0`, enabled Sabre RAW16/Mali-Xclipse GL Bayer fixes on Exynos sensors, and added the orange **`Exynos`** HUD indicator in the hardware backend badge slot.
- **GPU Offload (All Vendors — Adreno, Mali, PowerVR, Xclipse):**
  - Enabled `ShotParams_gpu_power_boost_set(true)`, `PortraitRequest_enable_gpu_boost_set(true)`, `PortraitRequest_use_gpu_resample_set(true)`, and `PortraitRequest_use_opencl_depth_set(true)` directly in the GCam pipeline.
  - Upgraded `libmakcam_turbojpeg.so` with a runtime **OpenGL ES 3.0 Pbuffer tiled GPU shader** (`libEGL.so` / `libGLESv3.so` with seamless ARM64 NEON SIMD fallback) running **Cupertino SmartTone**, **Soft-Highlight Knee**, **Chroma-Gated Vibrance**, and **Lateral Chromatic Aberration** on GPU.

### 2. Audit Fixes: `-0.3 EV` Removal, Shasta Daylight Shadow Noise Clean-up & Anti-Blur Guard
- **Removed built-in `-0.333 EV` offset:** Zeroed legacy `pref_exposure_option_available_key` and `auto_exp_key` in all 4 built-in XML profiles (`ai`, `classic`, `film`, `ai_gc51`) and migrated upgrading prefs — all EV adjustment now comes strictly from the manual `EVSwitcher`.
- **Bright-Light Shadow Denoise & Crisp Sabre Merging:** Unblocked `ShotParams_shasta_enabled` (`ShotParams_shasta_force_set(true)` + unlocked `ShotParams.f(Z)`), ported key `LibPatcher` sharpening/Sabre/denoising overrides directly into `AutoHighlightPatch` (`_SmoothingTheSabre = 0.75f`, `_SharpBig = 0.50f`, `_SabreDetail = 1.5`, boosted `_ChromaSmoothing = 2.0`, dynamic `_DenoiseSmoothing` per HUD Denoise mode with automatic session reload), and tightened the **Handheld Gyro Guard** (`33–50 ms` under handheld movement) to eliminate multi-frame Sabre blur and bright-light shadow chroma noise.

### 3. `>=108 MP` Buffer Protection, `4-in-1 / 9-in-1 / Auto` Binning & One-Tap Chromatic Aberration
- **`>=108 MP` RAW Ring Buffer Guard:** Dynamically caps `frameBudget` (`4–5` frames on 108 MP–200 MP RAW) and `max_imagereader_image_count` at ImageReader creation to prevent ION/ashmem out-of-memory exhaustion.
- **Dynamic Lower-Left `BinningSwitcher`:** Automatically appears on detected `>=108 MP` sensors (taking `ApertureSwitcher`'s slot when variable aperture is absent, or sitting immediately right of it when variable aperture is present). Cycles `Def` → `Auto` → `4→1` (forces GCam linear `downsample_by_2_before_merge`) → `9→1` (3×3 high-SNR superpixel binning in `libmakcam_turbojpeg.so`).
- **One-Button Chromatic Aberration (`CA: Off / On`):** Added `ChromaticSwitcher` to the bottom-left control bar — enables Camera2 `COLOR_CORRECTION_ABERRATION_MODE_HIGH_QUALITY` plus GPU/NEON Adobe-style radial R/B alignment and purple/green edge defringing.

### 4. Android 10 (SDK 29) Native Crash Fix, Dynamic Camera2 Resolutions & 12-Language UI
- **Android 10 (`whyred` / SDK 29) `dlopen` Fix:** Cleared stale `.rela.plt[206]` symbol binding (`ANativeWindow_tryAllocateBuffers`) in `libgcastartup.so` + safe `RET` at `0x18a3130` (`SurfaceNative_tryAllocateBuffers`, 0 callers) and added SDK < 30 conditional Hexagon host relocation cleanup (`LegacyDsp.loadHexagonHost`) without impacting Android 11–16.
- **Dynamic Camera2 Picture Sizes & Clean Universal Device Setup:** Replaced hardcoded Redmi K30 Pro Zoom picture sizes (`4624x3472`, `3264x2448`, `4208x3120`, `2592x1940`) with Camera2 `4:3` JPEG auto-resolution while gently isolating `lmipro`-specific calibration solely to `"lmipro"`; upgraded `ApertureSwitcher` visibility to use `LENS_INFO_AVAILABLE_APERTURES.length > 1`.
- **Internal Engraved-Lens VMC Icon:** Restored original external launcher icon styling while applying `camera_icon_v1_lens_engraved.png` (`hsl_lmc.png`, `vmc_brand_logo.png`) across internal settings and `AboutPage`.
- **Complete 12-Language HUD & Settings Menu Translation:** Expanded `hud_i18n.json` (263 keys) and added `menu_i18n.json` (346 preference keys) across all 12 `AboutPage` locales (`ru`, `uk`, `be`, `en`, `zh`, `hi`, `fr`, `de`, `ja`, `es`, `it`, `pt`).

---

## Builds & Packages (7 OEM Package IDs)

All 7 APKs are signed with a single certificate (`SHA-256: 1e08a903aef9c3a721510b64ec764d01d3d094eb954161b62544ea8f187b5953`), 16 KB page-aligned (`zipalign -P 16 4`), and verified (`versionCode: 66263640`):

| APK File | Package ID | Target Devices | SHA-256 |
| :--- | :--- | :--- | :--- |
| `VMC8.8_v98_vmcamera_obf.apk` | `com.google.android.vmcamera` | Primary package (updates existing VMC installs) | `C1E10DDB7DD66F38A8900C797FB6538DEAFF0026A192A992087A68BF7AC276A8` |
| `VMC8.8_v98_snapcam_obf.apk` | `org.codeaurora.snapcam` | Xiaomi, POCO, Redmi, ASUS, Motorola | `CDC2392ACB966D13D7069BF59C1C8A527F881DF93C534BEF582EE4A5CB3E1937` |
| `VMC8.8_v98_aweme_obf.apk` | `com.ss.android.ugc.aweme` | Realme, OPPO, OnePlus (ColorOS / RealmeUI AUX) | `46581CDF1934771A5D23325E003A411E6E9E98211D2FA9692DF38D73E6EC792D` |
| `VMC8.8_v98_qcamera3_obf.apk` | `org.codeaurora.qcamera3` | Snapdragon devices whitelisting `qcamera3` | `784EF5DD0B048886979C1FD25AD19C9F525CC13D994E9539626704CDBA8FFDD6` |
| `VMC8.8_v98_ruler_obf.apk` | `com.samsung.android.ruler` | Samsung Galaxy (OneUI AUX) | `9475DFF10A7C3872317CD4977FDB349F0FA9DA1B84DBB8610C040D5470348C55` |
| `VMC8.8_v98_scan3d_obf.apk` | `com.samsung.android.scan3d` | Samsung Galaxy (alternative AUX ID) | `DE702441C8F151CF38303A25904E80BE53AF317B4AB7208850A2E06AD3E4B223` |
| `VMC8.8_v98_glens_obf.apk` | `com.google.android.GoogleCameraEng` | Google Pixel & clean AOSP | `A54F289D9D545B37E07A41D685FE12B670EBA826D46E966DDF58369D390FDFD4` |
