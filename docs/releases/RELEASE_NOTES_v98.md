# VMC 8.8 v98 — Release Notes (RU)

🌐 **[English](RELEASE_NOTES_v98_EN.md)** | **Русский**

**Версия:** `VMC 8.8 v98` (`versionCode: 66263640`)  
**Дата:** 27 ноября 2026  

---

## Что нового в v98

### 1. Исправление MediaTek Sabre RAW16, поддержка Exynos (оранжевый бейдж `Exynos`) и разгрузка CPU на GPU
- **MediaTek Sabre MTK (`corot` / Dimensity / Helio):** устранено нативное падение C++ (`SIGABRT`) при съёмке в RAW16 патчем `sabre_processor.cc` (`0x1f6f77c` NOP проверки битности + `0x1F757F8` -> `28008052` + `0x1F75DB0` -> `08028052`), очисткой `pref_hexagon_key=0` на MediaTek и универсальным переключателем зума `1.0× / 1.2× / 1.5× / 2.0×` на основном модуле и `1.0× / 2.0×` на AUX.
- **Инициализация Samsung Exynos и активация Sabre:** исправлена инверсия ключа `pref_exynosfix_key` (`"-1"` = `Fix On` вместо прежнего `"1"` = `Fix Off`) по всем слотам объективов (`"", _2.._5, _front`), очищен `pref_hexagon_key=0`, активированы GL/Bayer-исправления Sabre для Exynos-сенсоров и добавлен оранжевый статус **`Exynos`** в верхнем HUD.
- **Перенос вычислений на графическое ядро (GPU всех вендоров — Adreno, Mali, PowerVR, Xclipse):**
  - Включены `ShotParams_gpu_power_boost_set(true)`, `PortraitRequest_enable_gpu_boost_set(true)`, `PortraitRequest_use_gpu_resample_set(true)` и `PortraitRequest_use_opencl_depth_set(true)` в пайплайне GCam.
  - В `libmakcam_turbojpeg.so` добавлен потоковый **OpenGL ES 3.0 Pbuffer GPU-шейдер** (`libEGL.so` / `libGLESv3.so` с бесшовным откатом на ARM64 NEON SIMD), выполняющий **Cupertino SmartTone**, **Soft-Highlight Knee**, **Chroma-Gated Vibrance** и **коррекцию хроматических аберраций** на GPU.

### 2. Исправления по аудиту снимков: удаление встроенных `-0.3 EV`, чистые тени с Shasta в ярких сценах и защита от смаза
- **Отвязка встроенного сдвига `-0.333 EV`:** обнулены `pref_exposure_option_available_key` и `auto_exp_key` во всех 4 встроенных XML-профилях (`ai`, `classic`, `film`, `ai_gc51`) и при обновлении настроек — коррекция экспозиции теперь задаётся исключительно пользователем через `EVSwitcher`.
- **Усреднение шума теней при ярком свете и резкость Sabre:** снята жёсткая заглушка `ShotParams.f(Z)` (`ech.smali`) и активирован `ShotParams_shasta_force_set(true)` для подъёма и слияния теней даже при низком AE ISO перед фонарями/мониторами/небом; в `AutoHighlightPatch` перенесены ключевые настройки резкости и шумоподавления из `LibPatcher` (`_SmoothingTheSabre = 0.75f`, `_SharpBig = 0.50f`, `_SabreDetail = 1.5`, усиленное `_ChromaSmoothing = 2.0` и переключение `_DenoiseSmoothing` из HUD Denoise с перезапуском сессии).
- **Усиленный Handheld Gyro Guard:** предел выдержки с рук сужен до `33–50 мс` при активном микродрожании для исключения мыльных кадров без ущерба для ночных серий со штатива.

### 3. Защита буфера `>=108 Мп`, биннинг `4 в 1 / 9 в 1 / Авто` и кнопка коррекции ХА
- **Защита буфера сенсоров `>=108 Мп`:** динамическое ограничение `frameBudget` (`4–5` кадров при полном RAW 108–200 Мп) и `max_imagereader_image_count` при создании `ImageReader` предотвращает переполнение ION/памяти.
- **Кнопка `BinningSwitcher` (слева снизу видоискателя):** появляется автоматически при детекции сенсора `>=108 Мп` (занимая позицию кнопки диафрагмы, если переменной диафрагмы нет, либо сразу правее неё с тем же отступом `8dp` и высотой `32dp` при наличии диафрагмы). Режимы: `Bin: Def` → `Bin: Auto` → `4→1` (линейный RAW 2×2 `downsample_by_2_before_merge` перед Sabre) → `9→1` (ночной 3×3 суперпиксель в `libmakcam_turbojpeg.so`).
- **Коррекция хроматических аберраций одной кнопкой (`CA: Off / On`):** кнопка `ChromaticSwitcher` в нижнем ряду включает аппаратный Camera2 `COLOR_CORRECTION_ABERRATION_MODE_HIGH_QUALITY` и GPU/NEON радиальное сведение R/B-каналов с подавлением фиолетовых и зелёных ореолов по краям кадра.

### 4. Исправление вылета на Android 10 (SDK 29), динамическое разрешение Camera2 и полный перевод на 12 языков
- **Android 10 (`whyred` / SDK 29) нативный фикс `dlopen`:** очищена привязка символа `ANativeWindow_tryAllocateBuffers` (`.rela.plt[206]`) в `libgcastartup.so` + безопасный `RET` в неиспользуемом `SurfaceNative_tryAllocateBuffers` (`0x18a3130`) и добавлен SDK < 30 загрузчик `LegacyDsp.loadHexagonHost` для `libhalide_hexagon_host.so` без влияния на Android 11–16.
- **Универсальное чтение разрешений и бережная очистка привязок K30 Pro (`lmipro`):** удалены жёсткие размеры фото `4624x3472`, `3264x2448`, `4208x3120`, `2592x1940` на прочих устройствах (автовыбор max `4:3` JPEG из `CameraCharacteristics`), привязки Redmi K30 Pro Zoom аккуратно изолированы за `isLmipro()`, а кнопка `ApertureSwitcher` теперь определяется динамически по `LENS_INFO_AVAILABLE_APERTURES.length > 1`.
- **Иконки бренда:** внешняя иконка лаунчера сохранена в оригинальном виде, а `camera_icon_v1_lens_engraved.png` (`hsl_lmc.png`, `vmc_brand_logo.png`) корректно упакована и отображается во внутренних настройках, сплэше и экране `О модификации`.
- **Полный перевод HUD и меню на все 12 языков:** расширен `hud_i18n.json` (263 ключа) и добавлен `menu_i18n.json` (346 ключей настроек) для всех 12 языков раздела `О модификации`: `ru`, `uk`, `be`, `en`, `zh`, `hi`, `fr`, `de`, `ja`, `es`, `it`, `pt`.

---

## Сборка и пакеты (7 OEM Package IDs)

Все 7 пакетов подписаны единым сертификатом (`SHA-256: 1e08a903aef9c3a721510b64ec764d01d3d094eb954161b62544ea8f187b5953`), выровнены под 16 КБ страницы (`zipalign -P 16 4`) и проверены (`versionCode: 66263640`):

| Файл APK | Package ID | Назначение | SHA-256 |
| :--- | :--- | :--- | :--- |
| `VMC8.8_v98_vmcamera_obf.apk` | `com.google.android.vmcamera` | Основной пакет (обновление с прошлых версий VMC) | `C1E10DDB7DD66F38A8900C797FB6538DEAFF0026A192A992087A68BF7AC276A8` |
| `VMC8.8_v98_snapcam_obf.apk` | `org.codeaurora.snapcam` | Xiaomi, POCO, Redmi, ASUS, Motorola | `CDC2392ACB966D13D7069BF59C1C8A527F881DF93C534BEF582EE4A5CB3E1937` |
| `VMC8.8_v98_aweme_obf.apk` | `com.ss.android.ugc.aweme` | Realme, OPPO, OnePlus (ColorOS / RealmeUI AUX) | `46581CDF1934771A5D23325E003A411E6E9E98211D2FA9692DF38D73E6EC792D` |
| `VMC8.8_v98_qcamera3_obf.apk` | `org.codeaurora.qcamera3` | Snapdragon-устройства с белым списком `qcamera3` | `784EF5DD0B048886979C1FD25AD19C9F525CC13D994E9539626704CDBA8FFDD6` |
| `VMC8.8_v98_ruler_obf.apk` | `com.samsung.android.ruler` | Samsung Galaxy (OneUI AUX) | `9475DFF10A7C3872317CD4977FDB349F0FA9DA1B84DBB8610C040D5470348C55` |
| `VMC8.8_v98_scan3d_obf.apk` | `com.samsung.android.scan3d` | Samsung Galaxy (альтернативный AUX ID) | `DE702441C8F151CF38303A25904E80BE53AF317B4AB7208850A2E06AD3E4B223` |
| `VMC8.8_v98_glens_obf.apk` | `com.google.android.GoogleCameraEng` | Google Pixel и чистый AOSP | `A54F289D9D545B37E07A41D685FE12B670EBA826D46E966DDF58369D390FDFD4` |
