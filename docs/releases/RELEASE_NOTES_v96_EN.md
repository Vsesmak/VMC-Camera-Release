# VMC 8.8 v96 — Release Notes (EN)

**Version:** `VMC 8.8 v96` (`versionCode: 66263638`)  
**Date:** November 25, 2026  

---

## What's New in v96

### 1. Hardware-First AutoCalibrator (Zero Scene Bias)
Completely removed legacy runtime JPEG scene feedback and scene-based parameter overrides (`observePhoto`, `photoCalibration`, `autotuner_database.json`, `isIndoorOrWarmScene`, and background JPEG analysis via ONNX/HTP).  
The AutoCalibrator now focuses strictly on **5 hardware-driven tasks**:
1. **Automatic Camera Discovery:** Identifies all physical lenses (Main, Tele, Ultrawide, ID4, Front), megapixels, pixel pitch (µm), OIS, RAW, and CFA; filters `< 4 MP` stub cameras; and maps exact lens slot suffixes (`_2.._5`, `_main`, `_tele`, `_wide`, `_id4`, `_front`).
2. **Automatic Color Model Matching for All Cameras:** Matches every detected module to the smartphone color matrix catalog with perceptual System Color Vibrance (`rowSum = 1.0`), eliminating double-saturation on AUX lenses.
3. **Automatic Noise Model Matching for All Cameras:** Selects sensor-accurate noise models per lens with Camera2 `SENSOR_NOISE_PROFILE` fallback.
4. **Optics-Aware Detail Calibration:** Upgrades disabled `"0"` sharpness keys to `"1"` while **preserving any higher values** (`lib_sabre_sharp_contrast_key* > 1` and `lib_polysharp_rad_med_key* > 1` are never lowered). `lib_shasta_force_key*` (`28008052`) is always preserved.
5. **Correct Session OpMode Selection:** Selects optimal Camera2 `OpMode` per camera, recovers from `onConfigureFailed` via camera quarantine, and guards non-Xiaomi devices from invalid `fix_resolution_key=37`.

### 2. Smartphone-Only Noise & Color Catalogs
- Stripped all DSLR, mirrorless, compact, and Raspberry Pi entries (Canon, Nikon, Fujifilm, Pentax, Olympus, Panasonic, Leica, Hasselblad, Sony ILCE/DSC/NEX) from `noise_models.json.gz`, `color_models.json.gz`, and GCam settings arrays.
- Bundled **429** validated smartphone noise models (indices `119..547`, `549` total entries) and **45** curated smartphone color matrices (indices `21..65`, `66` total entries), including 2020–2026 flagship profiles (Xiaomi, Samsung, Google Pixel, Sony Xperia, vivo, OPPO, OnePlus, Honor).

---

## Builds & Packages (7 OEM Package IDs)

All 7 APKs are signed with a single certificate (`SHA-256: 1e08a903aef9c3a721510b64ec764d01d3d094eb954161b62544ea8f187b5953`), 16 KB page-aligned (`zipalign -P 16 4`), and verified:

| APK File | Package ID | Target Devices | SHA-256 |
| :--- | :--- | :--- | :--- |
| `VMC8.8_v96_vmcamera_obf.apk` | `com.google.android.vmcamera` | Primary package (updates existing VMC installs) | `2CE54FD445A63906D1358AD3848D64D6310ECA5B6707448D05A736231ABAD631` |
| `VMC8.8_v96_snapcam_obf.apk` | `org.codeaurora.snapcam` | Xiaomi, POCO, Redmi, ASUS, Motorola | `AB0A0A15F3718B933A997728A42FE10622A36A27B91C40B20C34B81A60A76933` |
| `VMC8.8_v96_aweme_obf.apk` | `com.ss.android.ugc.aweme` | Realme, OPPO, OnePlus (ColorOS / RealmeUI AUX) | `130F021947C995F1BB72B2B04BCDE36A2C1FFB892D368120A3AA416307BEB8A5` |
| `VMC8.8_v96_qcamera3_obf.apk` | `org.codeaurora.qcamera3` | Snapdragon devices whitelisting `qcamera3` | `5E8552805C68A9D5C0965E1DAD185A15774335CAD700A9EA81D31E4845B11477` |
| `VMC8.8_v96_ruler_obf.apk` | `com.samsung.android.ruler` | Samsung Galaxy (OneUI AUX) | `7387A46640376494333366A958CBC4B7216E4110F8CFDA9DFEB9DC0B14376AEB` |
| `VMC8.8_v96_scan3d_obf.apk` | `com.samsung.android.scan3d` | Samsung Galaxy (alternative AUX ID) | `81547A3D8669AEB76808917FEE0132BAA8BACD3DEC761C4A9BBA22DB7615E876` |
| `VMC8.8_v96_glens_obf.apk` | `com.google.android.GoogleCameraEng` | Google Pixel & clean AOSP | `C5A2952E1082B97E7DB4242FFBABEF522F4CAE2581969AA2338A7430412B3A90` |
