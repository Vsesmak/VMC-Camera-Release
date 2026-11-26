# VMC 8.8 v97 — Release Notes (EN)

🌐 **English** | **[Русский](RELEASE_NOTES_v97.md)**

**Version:** `VMC 8.8 v97` (`versionCode: 66263639`)  
**Date:** November 26, 2026  

---

## What's New in v97

### 1. Smarter Highlight Recovery Without Breaking Night Atmosphere
- Enabled native highlight fixes (`0x263f2c8`, `0x1dd1074`, `0x1dd0890`) and balanced highlight headroom (`lib_highlight_comp_key = 08008152` / `2048`) across **all supported Android devices** in `Auto` and built-in profiles.
- Wired a selective **ARM64 NEON Soft-Highlight Knee** (`luma > 212`) inside `libmakcam_turbojpeg.so`: rescues neon signs, headlights, and bright skies cleanly while leaving night shadows and ambiance untouched.

### 2. Real TurboJPEG 3.2.1 NEON Pipeline: Chroma-Gated Vibrance + Cupertino SmartTone
- **Chroma-Gated NEON Vibrance** runs directly in the JPEG save path: boosts mid-chroma richness while keeping neutral asphalt/greys untouched (`chroma <= 12`) and protecting skin tones. Switchable in one tap via the **Chroma Vibrance** control next to `sRGB / DCI-P3`.
- **True Tonemap Hot-Swap:** Selecting native tone-mapping modes now writes `_StartTone` (`0xdbc290`) & `_StartGamma` (`0xdbc318`) and triggers a clean session restart so curves genuinely change the rendered image.
- Added **`8. Cupertino SmartTone`** — inspired by modern crisp S-curve contrast and wide-gamut tone reproduction with instant NEON execution.

### 3. MediaTek Sabre MTK Auto-Init & Purple `Mediatek` Badge
- Automatically detects MediaTek SoCs (Dimensity / Helio / MT6xxx) and applies **Sabre MTK** native patches (`0x1F757F8` -> `28008052`, `0x1F75DB0` -> `08028052`, byte-verified before patching).
- Displays a dedicated purple **`Mediatek`** indicator in the Top-Right HUD in place of the `HEX / CPU` badge.

### 4. AutoCalibrator v97: Instant Warm Cold Starts, `0.00 µm` Precision & Per-Lens Control
- **Fixed Startup Lag:** Unconditionally caches the profiled hardware state so subsequent cold starts immediately hit the fast cache path without re-scanning HALs or decompressing model catalogs.
- Removed obsolete weather tier strings, standardized sensor pixel pitch readout to 2 decimal places (`1.12 µm`), added an **Applied vs Kept** summary, and introduced **per-lens AutoCalibrator toggles** (`Main`, `Wide`, `Tele`, `Front`).

### 5. K30 Pro Zoom Button Fix, Unified VMC Branding, HUD Localization & Crash Fixes
- Fixed the phantom 4th `"3.0"` lens button on Redmi K30 Pro Zoom (`lmipro`) while preserving multi-lens routing on Samsung (`aurora`) and Realme/OPPO.
- Rolled out the new engraved-lens VMC app icon across the launcher, splash screen, and About screen.
- Added **`HudI18n`** (192 localized keys) synchronizing the entire Dual-Corner HUD and modal dialogs with the Android system language; fixed dynamic package name display in the Privacy section.
- Fixed reported crashes on **Android 10 (`whyred`, `WindowInsets$Type.navigationBars()`)** and **Android 11 (`davinci`, `Build.SOC_MODEL`)**.

---

## Builds & Packages (7 OEM Package IDs)

All 7 APKs are signed with a single certificate (`SHA-256: 1e08a903aef9c3a721510b64ec764d01d3d094eb954161b62544ea8f187b5953`), 16 KB page-aligned (`zipalign -P 16 4`), and verified:

| APK File | Package ID | Target Devices | SHA-256 |
| :--- | :--- | :--- | :--- |
| `VMC8.8_v97_vmcamera_obf.apk` | `com.google.android.vmcamera` | Primary package (updates existing VMC installs) | `875983B07CDECEF591FB3DD7FD875A2E33AD3440785A0EF56BA1A6712B27BE04` |
| `VMC8.8_v97_snapcam_obf.apk` | `org.codeaurora.snapcam` | Xiaomi, POCO, Redmi, ASUS, Motorola | `FECD0A552B22F0558986AABD20487B4FCEC269066D5CF630D1B03BF8C3A9CA9E` |
| `VMC8.8_v97_aweme_obf.apk` | `com.ss.android.ugc.aweme` | Realme, OPPO, OnePlus (ColorOS / RealmeUI AUX) | `326BD87178DD0B666361695E181CDA1E3303513BB3227067077DE2BF4297A709` |
| `VMC8.8_v97_qcamera3_obf.apk` | `org.codeaurora.qcamera3` | Snapdragon devices whitelisting `qcamera3` | `DB2DE55080D52B03301A24E3B6D420DE81943A1E7760B441CC3B4F86538EFD67` |
| `VMC8.8_v97_ruler_obf.apk` | `com.samsung.android.ruler` | Samsung Galaxy (OneUI AUX) | `E6F643551A42119FA3ABF3E9DEBAFF334AF5E2B4DB635326160128442360738F` |
| `VMC8.8_v97_scan3d_obf.apk` | `com.samsung.android.scan3d` | Samsung Galaxy (alternative AUX ID) | `FAA8902583D2702005916421AF08A47220F667DEC4EAA8AAB4A5B3B31A945ADA` |
| `VMC8.8_v97_glens_obf.apk` | `com.google.android.GoogleCameraEng` | Google Pixel & clean AOSP | `5F25259834C78E490A491A8C8188A07198F0A3E489D83FF6BF5116C970FA552D` |
