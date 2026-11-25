# Release Notes — VMC 8.8 v93

**Release Date:** November 23, 2026  
**Application Version:** `VMC 8.8 v93`  
**Package Identifier (Package ID):** `com.google.android.vmcamera`  
**Android versionCode:** `66263635`  
**Minimum Android Version:** Android 10 (API 29)  
**Target Android Version:** Android 12–15 (API 31–35)  
**SHA-256 Checksum:** `985BBCEA42B73233E629DBE4EDD1783375FE1C5EBBCCBD5C8F6826A9A1730D2D`  
**File Size:** `338,598,550 bytes (322.9 MB)`  
**Modification Author:** Vseslav Makarov (VSEMAK)  

📖 **Author's Manifesto & Architecture Background:** [Why Stock OEM Cameras Hit a Dead End and How VMC Was Built](WHY_VMC_EN.md)  
🌐 **Language / Язык:** **[English](RELEASE_NOTES_v93_EN.md)** | **[Русская версия](RELEASE_NOTES_v93.md)**

---

### 🌟 Key Changes and Innovations in Release v93

#### 1. Microdetail and Fine Texture Restoration (PolySharp Medium Radius Fix)
*   **The Issue:** In versions v91–v92, after accumulating over 35 consecutive captures under overcast lighting, the auto-calibrator pushed the sharpness offset to its extreme floor (`-0.40`). This caused the medium sharpness radius filter to be disabled entirely (`lib_polysharp_rad_med_key = 0`), resulting in the complete loss of fine microtextures across asphalt, gravel, leaves, bark, and fabrics ("watercolor" effect).
*   **The Solution:**
    *   Enforced a safe adaptive lower sharpness floor (`Math.max(-0.12, sharpnessBias - 0.04)` instead of `-0.40`).
    *   Implemented automatic return to neutral sharpness in the absence of edge halo artifacts (`emaHalo < 1.4`).
    *   Hard-locked `lib_polysharp_rad_med_key` to `"1"`, guaranteeing uninterrupted texture filtering across all shooting conditions.

#### 2. Elimination of Parasitic Green Shadow Cast on Sony IMX686 (`lmipro`)
*   **The Issue:** Under shade and overcast illumination ($CCT \approx 9500K$), the hardware sensor matrix `calibration1` excessively amplified the green channel ($G=1.0$ vs $R=0.59$ and $B=0.62$), causing dark road asphalt and shadow transitions to exhibit a noticeable green hue.
*   **The Solution:**
    *   Integrated dynamic color balance protection into `clampAwb()`: in `overcast_shade` conditions, red channel gain is clamped to a physically neutral ratio ($R \ge 1.82$), eliminating green shifts and preserving natural asphalt and shadow tones.

#### 3. Automatic Highlight DRC Floor Clamping
*   **The Issue:** On devices with the Sony IMX686 sensor, retaining zero exposure offset (`offsetEv = 0`) in overcast skies led to specular clipping on clouds and bright building facades.
*   **The Solution:**
    *   Enforced a protective negative exposure floor for the IMX686: `tierOffset = Math.min(tierOffset, -0.333)`, effectively protecting cloud textures and preventing highlight blowout.

#### 4. Physical Variable Aperture for Xiaomi 14 Ultra (Aperture Switcher)
*   Integrated a third direct sensor control button in the bottom-left corner of the live viewfinder.
*   Cyclic single-tap switching: `f/Auto` ➔ `f/1.63` ➔ `f/2.0`.
*   Long-tap reveals a modal dialog with a continuous 9-step physical aperture slider (`f/1.63, 1.8, 2.0, 2.2, 2.5, 2.8, 3.2, 3.5, 4.0`).
*   Intelligently hides on non-variable camera modules (ultrawide and telephoto lenses).

#### 5. In-Sensor Zoom (ISZ) Layout Alignment
*   On Xiaomi 14 Ultra (`aurora`), 4 dedicated In-Sensor Zoom buttons are active on the primary lens: `1x`, `1.2x`, `1.5x`, `2x`.
*   Secondary lenses and Redmi K30 Pro retain the battle-tested `1x` and `2x` layout.

#### 6. Binary Hardening and Debug Metadata Stripping
*   All native C/C++ libraries (`libmakcam_turbojpeg.so`, `libmakcam_hvx_probe.so`) processed via `llvm-strip --strip-all`, removing all local symbols, labels, and debug tables.
*   Java bytecode compiled with `d8 --release` and `javac -g:none`, completely eliminating line numbers and local variable tables.

#### 7. Dual-Corner HUD: Professional In-Viewfinder Control Suite
*   **Dynamic Hardware Acceleration Indicator (`HEX` vs `CPU`):** Real-time monitoring of FastRPC offload status: Qualcomm Hexagon DSP HVX (`HEX`) delivers maximal battery efficiency and zero thermal throttling, versus software fallback to CPU cores (`CPU`).
*   **HDR+ Enhanced Readiness Indicator:** Live tracking of the Zero Shutter Lag (ZSL) ring buffer depth and memory allocation, signaling readiness for 15–36 RAW frame bursts without shutter lag or queue drop risk.
*   **Live Denoise Monitoring & In-Flight Adjustment:** Real-time readout of spatial ($G_{spatial}$) and temporal ($G_{temporal}$) noise reduction gains with interactive Luma and Chroma sliders in the viewfinder.
*   **Hot-Swappable Tone-Mapping Models:** Seamless in-flight switching between `HDRnet Neural`, `Filmic Tone-Curve`, `ACES / Rec.709`, and `Reinhard Extended` without camera restarts.
*   **Hot-Swappable Color Space:** Instant selection between DCI-P3 Wide Gamut, Display P3, sRGB, and Rec.2020 with hardware CAT02 chromatic adaptation.
*   **Hot-Swappable AWB Engine:** In-flight switching between Google Neural AWB, Qualcomm Spectra Hardware ISP AWB, and Hybrid CCT Matrix with live Kelvin output.
*   **Real-Time Color Correction & CCM Tuning:** Interactive $3\times 3$ CCM calibration and dynamic `clampAwb` neutral point protection.
*   **Hot-Swappable Exposure Bracketing:** Auto adaptive mode, curated presets (HDR $\pm 2.0$ EV, Shadow Boost, Highlight Preservation), and parametric manual mode (series count, frame depth $N \in [3 \dots 48]$, and exposure step $\Delta EV$).
*   **Integrated In-Viewfinder Config Selector:** Immediate switching between embedded APK profiles (`AI Dynamic v93`, `Leica Classic`, `FilmStyle`), hardware auto-detection, and external XML files without double-tapping black viewports.

---

### 📦 Upgrade Instructions
1. Install `VMC8.8_v93.apk` over your previous version (no need to uninstall; all settings and captures are preserved).
2. Launch the application.
3. Verify the telemetry badge in the top-right corner: Hexagon DSP status should indicate active hardware processing (`kHexagon`).
4. To reset legacy auto-tuner biases (if a greenish cast was previously observed), tap the profile badge ➔ "Reset Auto-Calibration".
