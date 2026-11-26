# VMC 8.8 Changelog (v95 → v97)

🌐 **English** | **[Русский](CHANGELOG_v95_to_v97.md)**
☑️ **Smart Highlight Recovery & ARM64 NEON Soft-Highlight Knee (v97):** Enabled all-device native highlight protection and balanced highlight headroom (`2048`); inline NEON tone knee rescues signs and specular lights while keeping night ambiance intact
☑️ **TurboJPEG 3.2.1 NEON Vibrance & Cupertino SmartTone (v97):** Wired Chroma-Gated NEON Vibrance (zero grey/asphalt boost, skin protection) into JPEG output, added `Chroma Vibrance` beside `sRGB / DCI-P3`, and introduced `Cupertino SmartTone`
☑️ **Genuine Tonemap Curve Application (v97):** Native `_StartTone` / `_StartGamma` LUT writes with automatic camera session reload so selected tonemaps truly transform output frames
☑️ **MediaTek Sabre MTK & Purple `Mediatek` Badge (v97):** Automatic SoC detection and native Sabre MTK activation in `libgcastartup.so` paired with a purple `Mediatek` HUD badge
☑️ **Instant Cold Start, Per-Lens AutoCalibrator & Unified VMC UI (v97):** Unconditional cold-start cache fast path, per-lens AutoCalibrator toggles, `0.00 µm` pitch format, `Applied / Kept` summary, full multi-language HUD translation (`HudI18n`), updated engraved-lens VMC icon, and removal of phantom `3.0` button on Redmi K30 Pro Zoom
☑️ **Android 10 & 11 Crash Fixes (v97):** Added `WindowInsets` API 29 fallback (`whyred`) and reflection-guarded `Build.SOC_MODEL` lookup (`davinci`, API 30)
☑️ **Hardware-First AutoCalibrator & Smartphone-Only Catalogs (v96):** Removed legacy scene-learning JPEG overrides and shipped 429 mobile noise models + 45 mobile color matrices
☑️ **Multi-Brand AUX & Night Stabilization (v95–v96):** 7 OEM Package IDs under one keystore, gyroscope motion-blur protection, and strict preservation of high sharpness (`> 1`)
