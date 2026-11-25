# Полный Changelog VMC 8.8: от v93 до v95

---

## v95 (25 ноября 2026) — `versionCode 66263637`

### Опубликованные сборки (7 APK, единая подпись `1e08a903...`, `323.2 МБ`)
- `VMC8.8_v95_obf.apk` (`com.google.android.vmcamera`) — `SHA-256: CEBD07C2F90B02B720E70E8DDE34EE168F015650B48839F016347405225C0566`
- `VMC8.8_v95_snapcam_obf.apk` (`org.codeaurora.snapcam`) — `SHA-256: B22D1D95436C0FF54DAF31AAA5280A233BEBDFB33209E7A711CC4691FB8EF290`
- `VMC8.8_v95_aweme_obf.apk` (`com.ss.android.ugc.aweme`) — `SHA-256: D571380EC5223BA98D7E18041F87E8499AFADDBFFE9B64D9F5FA76E019E4EBA3`
- `VMC8.8_v95_qcamera3_obf.apk` (`org.codeaurora.qcamera3`) — `SHA-256: CA43A42F41888C9835406CDC17BC80E9616475FB38958A2BCABE5623A034A8AA`
- `VMC8.8_v95_ruler_obf.apk` (`com.samsung.android.ruler`) — `SHA-256: 9A9DF1E00731C4A0E05C5B962443C0C151AE47A5B881ABA9F0EE79605B3EF9DA`
- `VMC8.8_v95_scan3d_obf.apk` (`com.samsung.android.scan3d`) — `SHA-256: 077A3FFDC02D92FABA03AA0FADDB88AD01057AF21CE0C35F8E78C3B923ACDA77`
- `VMC8.8_v95_glens_obf.apk` (`com.google.android.GoogleCameraEng`) — `SHA-256: 7FA9E3662E67F59F0318D77978BA6D578883D3C653B233B320D123EEDCACCD0F`

### Основные изменения v95
- **+1 534 модели шумоподавления (`119..1652`, 1 654 всего)** и **+1 237 моделей цвета (`21..1257`, 1 258 всего)** в компактных сжатых каталогах `assets/models/` (`160.9 КБ` суммарно) с сохранением всех старых индексов конфигов (`-1..118` и `0..20`).
- **Perceptual System Color Vibrance (`ModelDatabase.applyPerceptualSystemVibrance`)**: корректная насыщенность для системной матрицы без линейного умножения `satCCT * 1.3` — нормировка строк `rowSum = 1.0`, защита телесных тонов в помещении и автоматическая адаптация под агрессивность аппаратной CCM.
- **Устранение ночного смаза при полном сохранении `lib_shasta_force_key = 28008052`**: включена выборочная отбраковка смазанных кадров (`pref_zsl_burstfix_key = 0`) во всех 4 профилях; добавлен гироскопический (`TYPE_GYROSCOPE`) + OIS ограничитель выдержки с рук с компенсацией числом кадров при сохранении до 5 с на штативе.
- **Переработка Автокалибратора (Hardware-First) + мастер-выключатель**:
  - Убрано снижение резкости (`sharpnessBias >= 0`, `lib_sabre_sharp_contrast_key` не отключается) и исправлен обход настроек OIS в `Shamim.GetC2apiPref()`.
  - Добавлен отдельный переключатель в HUD **`«⚡ Влияние автокалибратора на фото: ВКЛ / ВЫКЛ»`** (`makcam_autocalibrator_photo_override`).
  - Первичное профилирование в `LMC8.8/CurrentSetup/device_hardware_profile.json`: определение сенсоров, фильтрация `<4 МП` non-RAW заглушек, автоподбор моделей цвета/шума и `Session OpMode` (`61448` на K30 Pro Zoom OIS tele, профили Realme/OPPO/OnePlus) с автокарантином сбоев `onConfigureFailed`.
- **Разблокировка AUX на Realme / OPPO / OnePlus / Samsung**: универсальный опрос скрытых ID (`2..7`, `20..23`), запасной `CONTROL_ZOOM_RATIO` (`0.6×` / `2×` / `3×`) в `LensControl` и многопакетная сборка `-AllPackages` в `Build-HtpRelease.ps1` / `package_v31.py`.

---

## v94 (23 ноября 2026) — `versionCode 66263636`
*Файл: `VMC8.8_v94.apk` (`SHA-256: DE37B57BC401FBF2BA96D83B738B1AC2DA9C683B713E942A56AC7D5962A63407`)*

- **Устранение паразитного красного/пурпурного оттенка:** убран искусственный нижний порог `r >= 1.82f` в `clampAwb()`, AWB переведён на честный аппаратный вектор нейтрали `SENSOR_NEUTRAL_COLOR_POINT` (`AsShotNeutral`).
- **CCT-классификация уличных сцен:** исправлена логика `isIndoorOrWarmScene()` — тир `overcast_shade` при `CCT >= 6000K` больше не принимается за тёплое помещение.
- **Восстановление резкости Sabre:** добавлена санация `autotuner_database.json` (`sanitizePhotoCalibration()`) на запуске с исправлением дрейфа `sharpnessBias` до `-0.40` и ускоренным возвратом резкости.
- **Защита от ореольной ловушки Lanczos:** в `updateDetailTrial()` ресемплер Lanczos бракуется в пользу стока, если даёт более чем на 20% больше ореолов.

---

## v93 (23 ноября 2026) — `versionCode 66263635`
*Файл: `VMC8.8_v93.apk` (`SHA-256: 985BBCEA42B73233E629DBE4EDD1783375FE1C5EBBCCBD5C8F6826A9A1730D2D`)*

- **Фикс микродетализации (PolySharp Medium Radius):** ограничен порог снижения `sharpnessBias`, зафиксирован `lib_polysharp_rad_med_key = 1`.
- **Фикс зелёного оттенка асфальта на Sony IMX686 (`lmipro`):** внедрена коррекция `clampAwb()` для `overcast_shade` и защитный минимум `-0.333 EV` для защиты светов (`Highlight DRC Floor`).
- **Переменная диафрагма на Xiaomi 14 Ultra (`Aperture Switcher`):** кнопка `f/Auto → f/1.63 → f/2.0` + 9-ступенчатый слайдер (`f/1.63..4.0`) по долгому нажатию.
- **Разметка ISZ-зума и Dual-Corner HUD:** кнопки `1x / 1.2x / 1.5x / 2x` на `aurora`, индикатор `HEX / CPU`, управление денойзом, тонмапами (`HDRnet Neural`, `Filmic`, и др.), цветовым пространством (`DCI-P3 / sRGB`), AWB, брекетингом и селектором профилей прямо в видоискателе.
- **Стриппинг и оптимизация сборки:** `llvm-strip --strip-all` на нативных `.so` и `javac -g:none` + `d8 --release`.
