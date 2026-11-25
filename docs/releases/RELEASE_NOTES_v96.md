# VMC 8.8 v96 — Release Notes (RU)

**Версия:** `VMC 8.8 v96` (`versionCode: 66263638`)  
**Дата:** 25 ноября 2026  

---

## Что нового в v96

### 1. Аппаратный автокалибратор без сценарных искажений (Hardware-First AutoCalibrator)
Полностью удалена устаревшая логика обратной связи по JPEG и «обучения от условий сцены» (`observePhoto`, `photoCalibration`, `autotuner_database.json`, `isIndoorOrWarmScene`, постобработка JPEG через ONNX/HTP в фоне).  
Автокалибратор теперь выполняет ровно **5 аппаратных задач**:
1. **Автоматическая идентификация всех камер устройства:** определение роли (Main, Tele, Ultrawide, ID4, Front), мегапикселей, размера пикселя (µm), OIS, RAW и CFA, отсев заглушек `< 4 Мп`, точный маппинг суффиксов слотов (`_2.._5`, `_main`, `_tele`, `_wide`, `_id4`, `_front`).
2. **Автоматический подбор цветовой модели для всех камер:** сопоставление каждого физического модуля с базой смартфонных цветовых матриц + perceptual System Color Vibrance (`rowSum = 1.0`) без двойного перенасыщения на дополнительных объективах (AUX).
3. **Автоматический подбор модели шума для всех камер:** точный подбор профиля шума по сенсору и бренду с аппаратным фоллбэком на Camera2 `SENSOR_NOISE_PROFILE`.
4. **Автоматическая калибровка детализации под оптику:** апгрейд нулевых значений резкости до безопасного базового уровня (`"1"`) при **строгом сохранении более высоких значений** (`lib_sabre_sharp_contrast_key* > 1` и `lib_polysharp_rad_med_key* > 1` не понижаются!). `lib_shasta_force_key*` (`28008052`) сохраняется всегда.
5. **Корректный выбор Camera2 Session OpMode:** автоматический подбор и активация `OpMode` для каждого объектива, автовосстановление при сбоях конфигурации (`onConfigureFailed` quarantine) и защита не-Xiaomi устройств от ошибочного `fix_resolution_key=37`.

### 2. База моделей шума и цвета — только смартфоны (Smartphone-Only Catalogs)
- Удалены все профили фотоаппаратов и беззеркалок (Canon, Nikon, Fujifilm, Pentax, Olympus, Panasonic, Leica, Hasselblad, Sony ILCE/DSC/NEX, Raspberry Pi) из встроенных баз `noise_models.json.gz`, `color_models.json.gz` и настроек GCam.
- В релиз включены **429** проверенных моделей шума мобильных сенсоров (индексы `119..547`, всего `549` пунктов в меню) и **45** смартфонных цветовых матриц (индексы `21..65`, всего `66` пунктов в меню), включая актуальные флагманские матрицы 2020–2026 гг. (Xiaomi, Samsung, Google Pixel, Sony Xperia, vivo, OPPO, OnePlus, Honor).

---

## Сборка и пакеты (7 OEM Package IDs)

Все 7 пакетов подписаны единым сертификатом (`SHA-256: 1e08a903aef9c3a721510b64ec764d01d3d094eb954161b62544ea8f187b5953`), выровнены под 16 КБ страницы (`zipalign -P 16 4`) и готовы к установке:

| Файл APK | Package ID | Назначение | SHA-256 |
| :--- | :--- | :--- | :--- |
| `VMC8.8_v96_vmcamera_obf.apk` | `com.google.android.vmcamera` | Основной пакет (обновление с прошлых версий VMC) | `2CE54FD445A63906D1358AD3848D64D6310ECA5B6707448D05A736231ABAD631` |
| `VMC8.8_v96_snapcam_obf.apk` | `org.codeaurora.snapcam` | Xiaomi, POCO, Redmi, ASUS, Motorola | `AB0A0A15F3718B933A997728A42FE10622A36A27B91C40B20C34B81A60A76933` |
| `VMC8.8_v96_aweme_obf.apk` | `com.ss.android.ugc.aweme` | Realme, OPPO, OnePlus (ColorOS / RealmeUI AUX) | `130F021947C995F1BB72B2B04BCDE36A2C1FFB892D368120A3AA416307BEB8A5` |
| `VMC8.8_v96_qcamera3_obf.apk` | `org.codeaurora.qcamera3` | Snapdragon-устройства с белым списком `qcamera3` | `5E8552805C68A9D5C0965E1DAD185A15774335CAD700A9EA81D31E4845B11477` |
| `VMC8.8_v96_ruler_obf.apk` | `com.samsung.android.ruler` | Samsung Galaxy (OneUI AUX) | `7387A46640376494333366A958CBC4B7216E4110F8CFDA9DFEB9DC0B14376AEB` |
| `VMC8.8_v96_scan3d_obf.apk` | `com.samsung.android.scan3d` | Samsung Galaxy (альтернативный AUX ID) | `81547A3D8669AEB76808917FEE0132BAA8BACD3DEC761C4A9BBA22DB7615E876` |
| `VMC8.8_v96_glens_obf.apk` | `com.google.android.GoogleCameraEng` | Google Pixel и чистый AOSP | `C5A2952E1082B97E7DB4242FFBABEF522F4CAE2581969AA2338A7430412B3A90` |
