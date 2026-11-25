# VMC 8.8 Changelog (v93 → v96)

☑️ **Hardware-First AutoCalibrator with Zero Scene Bias (v96):** Completely removed legacy JPEG feedback and scene-based parameter overrides (`observePhoto`, `photoCalibration`, `isIndoorOrWarmScene`, background JPEG analysis). AutoCalibrator now performs strictly 5 hardware tasks: camera discovery, color & noise matching for all lenses, optics-aware detail calibration, and per-camera `Session OpMode` selection
☑️ **Preserved High Micro-Contrast (v96):** Optics-aware detail calibration upgrades disabled `"0"` values of `lib_sabre_sharp_contrast_key*` and `lib_polysharp_rad_med_key*` to `"1"`, while strictly preserving any higher user/profile values (`> 1`)
☑️ **Smartphone-Only Noise & Color Catalogs (v96):** Removed all DSLR/mirrorless camera entries (Canon, Nikon, Fujifilm, Sony Alpha, etc.) and curated 429 smartphone noise models + 45 mobile color matrices (2020–2026 flagships)
☑️ **Multi-Brand AUX & Non-Xiaomi Guard (v95–v96):** 7 OEM Package IDs (`vmcamera`, `snapcam`, `aweme`, `qcamera3`, `ruler`, `scan3d`, `glens`) under one signing certificate, universal AUX probing, and non-Xiaomi `fix_resolution_key=37` safety guard
☑️ **Night Motion Blur Fix & Unconditional Shasta (v95):** Selective ZSL blurry-frame rejection and gyroscope-aware handheld vs. tripod shutter guard while keeping `lib_shasta_force_key` (`28008052`) active across all lenses
☑️ **Perceptual System Color Vibrance (v95–v96):** Luminance-preserving System CCM color across all lenses (`rowSum = 1.0`) with zero AUX double-saturation
☑️ **Red Cast & Texture Mush Elimination (v94):** Removed artificial R-gain floor in `clampAwb()` so AWB relies purely on hardware `SENSOR_NEUTRAL_COLOR_POINT`
