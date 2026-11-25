# VMC 8.8 v95 (Vseslav Makarov Camera) — Release Notes

**Version:** `VMC 8.8 v95` (`versionCode: 66263637`)  
**Date:** November 25, 2026  

---

## What's New in v95

### 1. Expanded Catalogs: 1,534 Noise Models & 1,237 Color Matrices (Only 160.9 KB Compressed)
- **Noise Models (`119..1652`):** Added **1,534** validated noise models (flagship mobile sensors including Sony LYT-900/808/600, IMX989/890/858, Samsung HP2/HP3/GN3, OmniVision OV50H/OV64B, Google GNK + 1,114 Darktable profiles with Low/High ISO dual-band calibration).  
  Available under: *Settings → Additional settings → Processing functions → Noise model → Camera module → Select noise model*.
- **Color Models (`21..1257`):** Added **1,237** calibrated 3×3 color correction matrices (curated Leica, Hasselblad, Zeiss, Pixel, Fujifilm, Kodak profiles + Rawspeed, LibRaw & RawTherapee datasets).  
  Available under: *Settings → Additional settings → Processing functions → Color correction → Camera module → Select color model*.
- **Backward Config Compatibility:** Strictly appended **after index 118** (noise) and **after index 20** (color) so existing XML configs remain 100% compatible.

### 2. Perceptual "System Color Vibrance" (No Brute-Force `satCCT × 1.3` Clipping)
- Replaced naive linear saturation multipliers on System CCM (`index 1`) with `ModelDatabase.applyPerceptualSystemVibrance()`.
- Measures intrinsic diagonal aggressiveness of the hardware CCM, gently lifts sky/foliage chroma while restraining warm reds/skin indoors, and enforces strict `rowSum == 1.0` normalization to preserve neutral white balance and clean shadows.

### 3. Night Motion Blur Fix While Keeping `lib_shasta_force_key` Magic
- **`lib_shasta_force_key = 28008052` is permanently preserved** across every lens module and scenario.
- Re-enabled selective blurry-frame rejection (`pref_zsl_burstfix_key = 0`) in all 4 built-in XML profiles (`ai`, `classic`, `film`, `ai_gc51`) while keeping short anchor frames intact.
- Added a real-time **Gyroscope + OIS Handheld vs. Tripod Shutter Guard** in `HdrStatus`: handheld night shots cap overly slow frames and add compensatory HDR++ burst frames to eliminate motion blur, while static/tripod shots still allow exposures up to **5 seconds**.

### 4. Hardware-First AutoCalibrator Redesign & Independent Master Kill Switch
- **Stopped lowering sharpness:** AutoCalibrator no longer decrements `sharpnessBias` or disables `lib_sabre_sharp_contrast_key`. Fixed `Shamim.GetC2apiPref()` bypassing user OIS settings.
- **Independent HUD Toggle ("AutoCalibrator Photo Influence: ON/OFF"):** Completely turns off AutoCalibrator overrides at any time—leaving manual / XML / LibPatcher settings in full control without switching away from your active profile.
- **First-Run Hardware Profiler (`LMC8.8/CurrentSetup/device_hardware_profile.json`):** Automatically identifies sensors, filters out `<4MP` non-RAW stubs, assigns matching Noise & Color models, configures detail settings, and applies validated per-sensor `Session OpMode` values (e.g., `61448` on K30 Pro Zoom, Realme/OPPO/OnePlus profiles) with automatic `onConfigureFailed` quarantine recovery.

### 5. Universal AUX Camera Unlock & Multi-Package OEM Variants
- **Universal Auxiliary Discovery:** Probes hidden Camera2 IDs (`2..7`, `20..23`) and provides a `CONTROL_ZOOM_RATIO` Logical Multi-Camera fallback (`0.6×` / `2×` / `3×` on ID `0`) on Realme/OPPO devices that restrict physical IDs.
- **7 Package ID Variants Signed with a Single Certificate (`1e08a903...`):**
  - `VMC8.8_v95.apk` — `com.google.android.vmcamera` (`6EC857D76741FA9BD414E134F8D022430A660360195CC0B66EFCCA6B074AB6C1`)
  - `VMC8.8_v95_snapcam.apk` — `org.codeaurora.snapcam` (`F4C072464B6C1B3A80C53DC979902A28C2C704CF8D703F96D2CFF37424D0BE74`)
  - `VMC8.8_v95_aweme.apk` — `com.ss.android.ugc.aweme` (`FEA98166A044F3024DA63F12F7B785944DFCC4E001C73A59A4E6BFF1B61F256A`)
  - `VMC8.8_v95_qcamera3.apk` — `org.codeaurora.qcamera3` (`E67F7DCD87FBB779FEFDC0C6080FBAA7B3A281B8D6392731B0CD164DE981DD1F`)
  - `VMC8.8_v95_ruler.apk` — `com.samsung.android.ruler` (`1023A3D1776E146A47D2CCFE8D5AD7E1F8A2ABB241A93F0063ADF5DE8D21CD71`)
  - `VMC8.8_v95_scan3d.apk` — `com.samsung.android.scan3d` (`20BD39EC82D8C3D6AA95F227B4B059FC13BDD7EE2B70C368E4146C74BFE581F1`)
  - `VMC8.8_v95_glens.apk` — `com.google.android.GoogleCameraEng` (`A3A2D067F8C03F7BD4CE02514C9820FEEC18161718FB9026C75550C54B528736`)
- **7 Package Variants (Single Keystore):**
  - `VMC8.8_v95_vmcamera_obf.apk` (`CEBD07C2F90B02B720E70E8DDE34EE168F015650B48839F016347405225C0566`)
  - `VMC8.8_v95_snapcam_obf.apk` (`B22D1D95436C0FF54DAF31AAA5280A233BEBDFB33209E7A711CC4691FB8EF290`)
  - `VMC8.8_v95_aweme_obf.apk` (`D571380EC5223BA98D7E18041F87E8499AFADDBFFE9B64D9F5FA76E019E4EBA3`)
  - `VMC8.8_v95_qcamera3_obf.apk` (`CA43A42F41888C9835406CDC17BC80E9616475FB38958A2BCABE5623A034A8AA`)
  - `VMC8.8_v95_ruler_obf.apk` (`9A9DF1E00731C4A0E05C5B962443C0C151AE47A5B881ABA9F0EE79605B3EF9DA`)
  - `VMC8.8_v95_scan3d_obf.apk` (`077A3FFDC02D92FABA03AA0FADDB88AD01057AF21CE0C35F8E78C3B923ACDA77`)
  - `VMC8.8_v95_glens_obf.apk` (`7FA9E3662E67F59F0318D77978BA6D578883D3C653B233B320D123EEDCACCD0F`)
