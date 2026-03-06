# 2.2: Flutter CLI

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 2 — Настройка окружения Flutter

---

### 2.2: Flutter CLI

🎯 **Цель шага:** Освоить ключевые команды Flutter CLI для ежедневной разработки FitMenu — запуск, анализ, сборка релизного APK и управление зависимостями.

📝 **Техническое задание:**

Выполни следующие команды и задокументируй их в разделе `README.md`:

1. `flutter devices` — вывести список устройств, найти `device_id` эмулятора
2. `flutter run -d <device_id>` — запустить FitMenu на конкретном устройстве
3. `flutter pub add dio flutter_riverpod` — добавить зависимости через CLI
4. `flutter analyze --fatal-infos` — статический анализ; исправь все предупреждения до нуля
5. `flutter format lib/ test/` — форматирование по Dart стандарту
6. `flutter build apk --release --obfuscate --split-debug-info=build/debug-info/`
7. `flutter test --coverage` — запустить тесты с отчётом покрытия

Добавь в `README.md` таблицу с командами и их описанием.

✅ **Критерии приёмки:**
- [ ] `flutter analyze` выдаёт `No issues found!`
- [ ] `flutter format --set-exit-if-changed lib/` завершается с кодом 0 (код уже отформатирован)
- [ ] Release APK собирается без ошибок (`flutter build apk --release`)
- [ ] `flutter test` проходит (`widget_test.dart` из шаблона)
- [ ] `README.md` содержит раздел "Development Commands"

💡 **Подсказка:** `--obfuscate` скрывает имена классов при декомпиляции APK. Всегда сохраняй папку `build/debug-info/` — без неё невозможно расшифровать стектрейсы из Crashlytics.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```bash
# Список устройств
flutter devices
# Пример вывода:
# emulator-5554 • sdk gphone64 x86 64 • android-x64 • Android 14 (API 34)
# iPhone 15 Pro • mobile • ios • iOS 17.0
# Chrome (web)  • web-javascript • Google Chrome 121

# Запуск на конкретном устройстве
flutter run -d emulator-5554
flutter run -d chrome --web-port 8080

# Управление зависимостями
flutter pub add dio flutter_riverpod google_fonts
flutter pub add --dev mockito build_runner mocktail

# Анализ (--fatal-infos завершается с кодом 1 при info-предупреждениях — полезно в CI)
flutter analyze --fatal-infos

# Форматирование
flutter format lib/ test/
flutter format --set-exit-if-changed lib/  # для CI-проверки

# Сборка release APK с обфускацией
flutter build apk --release \
  --obfuscate \
  --split-debug-info=build/debug-info/ \
  --dart-define=APP_ENV=production

# App Bundle для Google Play (предпочтительнее APK)
flutter build appbundle --release

# Тесты с покрытием
flutter test --coverage
# lcov → HTML (требует установки lcov)
# genhtml coverage/lcov.info -o coverage/html

# Проверка устаревших зависимостей
flutter pub outdated
flutter pub upgrade --major-versions  # осторожно: breaking changes
```

```markdown
<!-- README.md -->
# FitMenu 🥗

Трекер питания и рецептов для iOS и Android.

## Быстрый старт
```bash
git clone https://github.com/your-org/fitfood.git
cd fitfood
flutter pub get
flutter run
```

## Development Commands

| Команда | Описание |
|---------|----------|
| `flutter run` | Debug-запуск |
| `flutter run --release` | Release-запуск |
| `flutter analyze` | Статический анализ |
| `flutter format lib/` | Форматирование |
| `flutter test` | Все тесты |
| `flutter test --coverage` | Тесты + покрытие |
| `flutter build apk --release` | Release APK |
| `flutter build appbundle` | App Bundle (Play Store) |
| `flutter pub get` | Установить зависимости |
| `flutter pub outdated` | Устаревшие пакеты |
```

</details>
