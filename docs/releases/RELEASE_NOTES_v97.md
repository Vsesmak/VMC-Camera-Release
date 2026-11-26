# VMC 8.8 v97 — Release Notes (RU)

**Версия:** `VMC 8.8 v97` (`versionCode: 66263639`)  
**Дата:** 26 ноября 2026  

---

## Что нового в v97

### 1. Умное спасение пересветов без потери ночной атмосферы (All-Device Highlights + NEON Knee)
- Активирован нативный фикс светов (`0x263f2c8`, `0x1dd1074`, `0x1dd0890`) и деликатная компенсация пиковых светов `lib_highlight_comp_key = 08008152` (`2048`) для **всех смартфонов** в режиме `Auto` и встроенных профилях.
- В `libmakcam_turbojpeg.so` добавлен селективный **ARM64 NEON Soft-Highlight Knee** (`luma > 212`): плавно возвращает детали неоновых вывесок, фар и неба **без затемнения теней и атмосферы кадра**.

### 2. Реальный пайплайн TurboJPEG 3.2.1 NEON: Chroma-Gated Vibrance + Cupertino SmartTone
- **Chroma-Gated NEON Vibrance** подключён непосредственно в поток сохранения JPEG: точечно оживляет цвета при нулевом влиянии на нейтральный серый асфальт и тени (`chroma ≤ 12`) и бережной защите оттенков кожи. Доступен быстрый переключатель `Chroma Vibrance` рядом с выбором гаммы `sRGB / DCI-P3`.
- **Честная горячая смена Tonemap-моделей:** выбор модели в HUD теперь записывает нативные таблицы `_StartTone` (`0xdbc290`) и `_StartGamma` (`0xdbc318`) и автоматически перезапускает сессию камеры для реального применения кривых.
- Добавлен профиль **`8. Cupertino SmartTone`** — вдохновлённый мобильной цветопередачей и кривыми контраста с мгновенным NEON P3-обработкой без перезапуска.

### 3. Полная поддержка MediaTek (Sabre MTK + фиолетовый бейдж `Mediatek`)
- Автоматическое определение чипсетов MediaTek (Dimensity / Helio / MT6xxx) и безопасная инициализация **Sabre MTK** в `libgcastartup.so` (`0x1F757F8` -> `28008052`, `0x1F75DB0` -> `08028052` после верификации исходных байтов).
- На месте индикатора `HEX / CPU` на поддерживаемых SoC отображается фирменный фиолетовый бейдж **`Mediatek`**.

### 4. Автокалибратор v97: мгновенный старт, точность `0.00 µm` и пообъективный контроль
- **Устранена задержка холодного запуска:** кэшированный аппаратный профиль (`Fast-loaded cached Hardware-First profile`) теперь сохраняется безусловно — на 2-м и последующих стартах тяжелое сканирование камер и декомпрессия баз пропускаются.
- Удалены сбивающие с толку погодные ярлыки (`облачно/пасмурно`), размер пикселя форматируется как `1.12 µm` (2 знака после точки).
- Добавлена понятная сводка **`✔ Применено` vs `• Сохранено`** (включая гарантию `Shasta=ON` и `Sharp >1 kept`) и **индивидуальный переключатель Автокалибратора для каждого сенсора** (`Main`, `Wide`, `Tele`, `Front`).

### 5. Исправление фантомной кнопки `3.0` на Redmi K30 Pro Zoom, редизайн и локализация HUD
- Удалена ошибочная 4-я кнопка зума `3.0` на `lmipro` без нарушения работы Samsung (`aurora`) и Realme/OPPO AUX.
- Обновлена иконка приложения, сплэш-экран и экран `О приложении` в едином стиле VMC с выгравированным объективом.
- Добавлена полная система локализации **`HudI18n`** (192 ключа) — весь Dual-Corner HUD и модальные диалоги автоматически переводятся вслед за системным языком Android и языком всех модулей APK; исправлена подстановка `packageName` в тексте приватности.
- Исправлены вылеты из присланных краш-логов на **Android 10 (`whyred` / MIUI 12, `WindowInsets$Type.navigationBars()`)** и **Android 11 (`davinci` / Mi 9T, `Build.SOC_MODEL`)**.

---

## Сборка и пакеты (7 OEM Package IDs)

Все 7 пакетов подписаны единым сертификатом (`SHA-256: 1e08a903aef9c3a721510b64ec764d01d3d094eb954161b62544ea8f187b5953`), выровнены под 16 КБ страницы (`zipalign -P 16 4`) и проверены:

| Файл APK | Package ID | Назначение | SHA-256 |
| :--- | :--- | :--- | :--- |
| `VMC8.8_v97_vmcamera_obf.apk` | `com.google.android.vmcamera` | Основной пакет (обновление с прошлых версий VMC) | `875983B07CDECEF591FB3DD7FD875A2E33AD3440785A0EF56BA1A6712B27BE04` |
| `VMC8.8_v97_snapcam_obf.apk` | `org.codeaurora.snapcam` | Xiaomi, POCO, Redmi, ASUS, Motorola | `FECD0A552B22F0558986AABD20487B4FCEC269066D5CF630D1B03BF8C3A9CA9E` |
| `VMC8.8_v97_aweme_obf.apk` | `com.ss.android.ugc.aweme` | Realme, OPPO, OnePlus (ColorOS / RealmeUI AUX) | `326BD87178DD0B666361695E181CDA1E3303513BB3227067077DE2BF4297A709` |
| `VMC8.8_v97_qcamera3_obf.apk` | `org.codeaurora.qcamera3` | Snapdragon-устройства с белым списком `qcamera3` | `DB2DE55080D52B03301A24E3B6D420DE81943A1E7760B441CC3B4F86538EFD67` |
| `VMC8.8_v97_ruler_obf.apk` | `com.samsung.android.ruler` | Samsung Galaxy (OneUI AUX) | `E6F643551A42119FA3ABF3E9DEBAFF334AF5E2B4DB635326160128442360738F` |
| `VMC8.8_v97_scan3d_obf.apk` | `com.samsung.android.scan3d` | Samsung Galaxy (альтернативный AUX ID) | `FAA8902583D2702005916421AF08A47220F667DEC4EAA8AAB4A5B3B31A945ADA` |
| `VMC8.8_v97_glens_obf.apk` | `com.google.android.GoogleCameraEng` | Google Pixel и чистый AOSP | `5F25259834C78E490A491A8C8188A07198F0A3E489D83FF6BF5116C970FA552D` |
