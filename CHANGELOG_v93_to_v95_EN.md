# Full Changelog — VMC 8.8 (v93 to v95)

---

## v95 (November 25, 2026) — `versionCode 66263637`

### Published R8-Obfuscated Release APKs (7 APKs, Single Keystore `1e08a903...`, `323.2 MB` each, R8 8.6.2)
- `VMC8.8_v95_obf.apk` (`com.google.android.vmcamera`) — `SHA-256: CEBD07C2F90B02B720E70E8DDE34EE168F015650B48839F016347405225C0566`
- `VMC8.8_v95_snapcam_obf.apk` (`org.codeaurora.snapcam`) — `SHA-256: B22D1D95436C0FF54DAF31AAA5280A233BEBDFB33209E7A711CC4691FB8EF290`
- `VMC8.8_v95_aweme_obf.apk` (`com.ss.android.ugc.aweme`) — `SHA-256: D571380EC5223BA98D7E18041F87E8499AFADDBFFE9B64D9F5FA76E019E4EBA3`
- `VMC8.8_v95_qcamera3_obf.apk` (`org.codeaurora.qcamera3`) — `SHA-256: CA43A42F41888C9835406CDC17BC80E9616475FB38958A2BCABE5623A034A8AA`
- `VMC8.8_v95_ruler_obf.apk` (`com.samsung.android.ruler`) — `SHA-256: 9A9DF1E00731C4A0E05C5B962443C0C151AE47A5B881ABA9F0EE79605B3EF9DA`
- `VMC8.8_v95_scan3d_obf.apk` (`com.samsung.android.scan3d`) — `SHA-256: 077A3FFDC02D92FABA03AA0FADDB88AD01057AF21CE0C35F8E78C3B923ACDA77`
- `VMC8.8_v95_glens_obf.apk` (`com.google.android.GoogleCameraEng`) — `SHA-256: 7FA9E3662E67F59F0318D77978BA6D578883D3C653B233B320D123EEDCACCD0F`

### Key Changes in v95
- **+1,534 Noise Models (`119..1652`, 1,654 total)** & **+1,237 Color Models (`21..1257`, 1,258 total)** bundled in compressed `assets/models/` (`160.9 KB` total), preserving all legacy config indices (`-1..118` and `0..20`).
- **Perceptual System Color Vibrance (`ModelDatabase.applyPerceptualSystemVibrance`)**: replaces naive `satCCT * 1.3` multiplication with Rec.709 luminance-preserving spectral vibrance, intrinsic CCM diagonal attenuation, indoor skin protection, and strict `rowSum == 1.0` neutral preservation.
- **Night Motion Blur Fix Without Disabling `lib_shasta_force_key = 28008052`**: re-enabled selective blurry-frame rejection (`pref_zsl_burstfix_key = 0`) across all 4 XML profiles; added real-time `TYPE_GYROSCOPE` + OIS handheld shutter guard with dark-scene frame count compensation while keeping up to 5s exposures on tripods.
- **Hardware-First AutoCalibrator Redesign & Master Kill Switch**:
  - Removed `sharpnessBias` reduction and prevented `lib_sabre_sharp_contrast_key` disabling; fixed `Shamim.GetC2apiPref()` bypassing OIS preferences.
  - Added independent HUD toggle (`makcam_autocalibrator_photo_override`) to disable all AutoCalibrator photo overrides cleanly without changing profiles.
  - Added first-run hardware profiling to `LMC8.8/CurrentSetup/device_hardware_profile.json` (sensor role/MP/OIS/RAW identification, `<4MP` stub filtering, `ModelDatabase` noise/color matching, and per-sensor `Session OpMode` selection with `onConfigureFailed` quarantine recovery).
- **Universal AUX Unlock & Multi-Package Variants**: probes hidden Camera2 IDs (`2..7`, `20..23`), adds `CONTROL_ZOOM_RATIO` logical zoom fallback (`0.6×` / `2×` / `3×`) on Realme/OPPO, and supports `-AllPackages` in `Build-HtpRelease.ps1`.

---

## v94 (November 23, 2026) — `versionCode 66263636`
*APK: `VMC8.8_v94.apk` (`SHA-256: DE37B57BC401FBF2BA96D83B738B1AC2DA9C683B713E942A56AC7D5962A63407`)*

- **Fixed Parasitic Red/Magenta Cast:** removed the artificial `r >= 1.82f` clamp in `clampAwb()`, relying directly on hardware `SENSOR_NEUTRAL_COLOR_POINT` (`AsShotNeutral`).
- **CCT-Aware Scene Classification:** fixed `isIndoorOrWarmScene()` so `overcast_shade` with `CCT >= 6000K` stays classified as cool outdoor light.
- **Sabre Micro-Contrast Restoration:** added `sanitizePhotoCalibration()` at startup/reconcile to rescue legacy `-0.40` `sharpnessBias` drift and keep `lib_sabre_sharp_contrast` active.
- **Lanczos Halo Trap Fix:** `updateDetailTrial()` rejects Lanczos whenever its halo rate exceeds stock resampling by > 20%.

---

## v93 (November 23, 2026) — `versionCode 66263635`
*APK: `VMC8.8_v93.apk` (`SHA-256: 985BBCEA42B73233E629DBE4EDD1783375FE1C5EBBCCBD5C8F6826A9A1730D2D`)*

- **PolySharp Medium Radius Texture Restoration:** clamped downward sharpness bias drift and locked `lib_polysharp_rad_med_key = 1`.
- **Sony IMX686 Green Cast & Highlight Protection:** added `clampAwb()` green-shift protection and `-0.333 EV` DRC highlight floor.
- **Xiaomi 14 Ultra Variable Aperture Switcher:** `f/Auto → f/1.63 → f/2.0` button + 9-step slider (`f/1.63..4.0`).
- **In-Sensor Zoom Layout & Dual-Corner HUD:** `1x / 1.2x / 1.5x / 2x` controls on `aurora`, `HEX / CPU` DSP badge, live denoise/tonemap/gamut/AWB/bracketing/profile HUD controls.
- **Release Hardening:** `llvm-strip --strip-all` on native `.so` libraries plus `javac -g:none` & `d8 --release`.
