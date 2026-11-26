# Release Notes — VMC 8.8 v94

🌐 **English** | **[Русский](RELEASE_NOTES_v94.md)**
**Release Date:** November 23, 2026  
**Application Version:** `VMC 8.8 v94`  
**Package ID:** `com.google.android.vmcamera`  
**Android versionCode:** `66263636`  
**Minimum Android Version:** Android 10 (API 29)  
**Target Android Version:** Android 12–15 (API 31–35)  
**SHA-256 Checksum:** `DE37B57BC401FBF2BA96D83B738B1AC2DA9C683B713E942A56AC7D5962A63407`  
**File Size:** `338,598,550 bytes (322.9 MB)`  
**Author / Lead Developer:** Vseslav Makarov (Camera by Vseslav Makarov)  

📖 **Engineering Manifesto:** [Why Stock Mobile Cameras Hit a Wall and How I Rebuilt Google Camera](WHY_VMC_EN.md)

---

### 🌟 Key Engineering Fixes and Optimizations in Release v94

Release **v94** completely resolves two interconnected systemic image processing defects that persisted across all shooting environments: an unwanted parasitic red/magenta color cast and loss of micro-texture detail ("watercolor mush").

---

#### 1. Complete Elimination of the Parasitic Red/Magenta Cast (Fix clampAwb)
* **Root Cause:** In `clampAwb()`, an artificial red-gain floor `r = Math.max(r, 1.82f)` was originally hardcoded for the Sony IMX686 sensor (`lmipro`) under overcast/shaded conditions (`overcast_shade`). Once the AutoTuner calibrated a cold outdoor scene (CCT ~9500K), `overcast_shade` locked in as the persistent `activeTier`. This forced excessive +34% red channel multiplication into all subsequent captures regardless of lighting (R/G ratio surged to 1.343, with shadow red levels exceeding green by over 2.5×).
* **Engineering Solution:**
  * Completely eliminated the artificial `r >= 1.82f` floor in `clampAwb()`.
  * Auto White Balance now relies 100% on the genuine hardware `SENSOR_NEUTRAL_COLOR_POINT` (`AsShotNeutral` vector in DNG) from the IMX686 sensor without artificial shifts.
  * Color balance is restored to physical neutrality: asphalt, road surfaces, neutral tones, and portraits are rendered accurately with zero magenta/red color contamination.

---

#### 2. CCT-Aware Outdoor Scene Classification (Fix isIndoorOrWarmScene)
* **Root Cause:** `isIndoorOrWarmScene()` erroneously classified `overcast_shade` as warm indoor lighting because it fell outside direct sunlight flags. Consequently, cold outdoor shade and overcast skies (CCT 6000–9500K) were subjected to warm indoor tone handling, disengaging the correct Dynamic v92 color matrix.
* **Engineering Solution:**
  * Enhanced `isIndoorOrWarmScene()` with correlated color temperature evaluation: `overcast_shade` now inspects calibrated CCT ($CCT \ge 6000\text{ K}$) and correctly identifies as cold outdoor skylight.
  * Prevents incorrect warm matrix overrides during outdoor photography.

---

#### 3. Restoration of Sabre Micro-Contrast & AutoTuner Sanitize (Fix Texture Blur)
* **Root Cause:** Legacy test builds allowed `sharpnessBias` to drift to `-0.40`. Whenever `sharpnessBias` dropped below `-0.06`, the engine automatically disabled Sabre multi-frame edge contrast (`lib_sabre_sharp_contrast = 0`). Images suffered severe loss of high-frequency detail across foliage, fur, bark, and fabric, turning fine textures into smeared mush.
* **Engineering Solution:**
  * Introduced `sanitizePhotoCalibration()`, executing on every database load and reconcile cycle. All stored values are strictly clamped to valid operational boundaries: `sharpnessBias` within `[-0.12, +0.35]` and `denoiseBias` within `[-0.30, +0.50]`. Legacy `-0.40` entries are corrected immediately in memory upon launch.
  * Added accelerated sharpness recovery: when edge halos are within normal limits (`emaHalo < 1.6`), `sharpnessBias` rapidly steps back toward neutral at `+0.05` per frame.
  * `lib_sabre_sharp_contrast` remains reliably active, delivering crisp pixel-level micro-contrast without watercolor smoothing.

---

#### 4. Protection Against Lanczos Halo Feedback Traps (Hardened Halo Threshold)
* **Root Cause:** `updateDetailTrial()` accepted Lanczos resampling (`lanczos_retained`) even when halo artifacts reached 40.8% (compared to 28.7% for stock resampling). Elevated edge halos subsequently forced the adaptive sharpness controller to push `sharpnessBias` deeper into negative territory, trapping the camera in a cycle of image degradation.
* **Engineering Solution:**
  * Established a strict relative threshold: Lanczos is rejected in favor of stock resampling (`stock_resampling_restored`) whenever its halo rate exceeds stock by more than 20%.
  * The feedback loop of "halos ➔ sharpness reduction ➔ blur" is permanently broken.

---

#### 5. Release Obfuscation & Bytecode Optimization
* Java source code is compiled with `javac -g:none` (completely stripping debug line numbers, source files, and local variable tables) and optimized with `d8 --release`.
* Native binaries (`libmakcam_turbojpeg.so`, `libmakcam_hvx_probe.so`) are compiled with `-fvisibility=hidden` and `-O3` optimizations.
* The APK package is 16-byte aligned via `zipalign -P 16 4` and signed using an Android v3 cryptographic certificate.

---

### 📥 Installation & Upgrade
**VMC 8.8 v94** installs directly over v93. Existing calibration databases (`autotuner_database.json`) are automatically sanitized upon the first launch—manual data wiping or cache clearing is no longer required.
