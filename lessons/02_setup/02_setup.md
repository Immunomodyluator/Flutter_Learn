# 2. Настройка окружения Flutter

## 1. Суть концепции

Flutter SDK — набор инструментов: компилятор Dart, фреймворк виджетов, движок рендеринга (Skia/Impeller), CLI и DevTools. Устанавливается один раз, поддерживает сборку под Android, iOS, Web, Desktop из одной кодовой базы.

---

## 2. Установка Flutter SDK

### Windows

```powershell
# 1. Скачать Flutter SDK с flutter.dev/docs/get-started/install
# 2. Распаковать в C:\src\flutter (НЕ в Program Files — проблемы с правами)
# 3. Добавить в PATH:
$env:PATH += ";C:\src\flutter\bin"
# Или через System Properties > Environment Variables (постоянно)

# 4. Проверить установку
flutter doctor
```

### macOS / Linux

```bash
# Через git (рекомендуется для обновлений)
git clone https://github.com/flutter/flutter.git -b stable ~/flutter
export PATH="$PATH:$HOME/flutter/bin"

# Или через FVM (Flutter Version Manager) — лучший вариант для команд
dart pub global activate fvm
fvm install stable
fvm use stable
```

---

## 3. flutter doctor

`flutter doctor` — диагностика окружения. Запускать после каждого изменения.

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x.x)
[✓] Android toolchain - develop for Android devices
[✓] Xcode - develop for iOS and macOS          ← только macOS
[✓] Chrome - develop for the web
[✓] Android Studio (version 2024.x)
[✓] VS Code (version 1.x)
[✓] Connected device (2 available)
[✓] Network resources
```

Каждый `[✗]` содержит инструкцию что нужно установить.

---

## 4. Настройка IDE

### VS Code (рекомендуется для начала)

Расширения обязательные:

- **Flutter** (от Dart Code team) — включает Dart extension
- **Dart** — устанавливается автоматически с Flutter

Полезные:

- **Flutter Widget Snippets** — сниппеты для быстрого создания виджетов
- **Pubspec Assist** — добавление зависимостей без редактирования вручную
- **Error Lens** — ошибки прямо в коде

### Android Studio

- Установить плагин Flutter (File → Settings → Plugins)
- Android Studio включает встроенный эмулятор — удобнее для Android-разработки

---

## 5. Создание первого проекта

```bash
# Создать проект
flutter create my_app

# С явным указанием org (важно для App Store / Google Play)
flutter create --org com.yourcompany my_app

# Указать только нужные платформы
flutter create --platforms android,ios my_app

# Перейти в папку и запустить
cd my_app
flutter run
```

---

## 6. Типичные проблемы

| Проблема                      | Решение                                                      |
| ----------------------------- | ------------------------------------------------------------ |
| `flutter: command not found`  | Не добавлен PATH                                             |
| Android licenses not accepted | `flutter doctor --android-licenses`                          |
| No connected devices          | Запустить эмулятор или подключить устройство с USB debugging |
| CocoaPods не установлен (iOS) | `sudo gem install cocoapods` или `brew install cocoapods`    |
| Xcode недостаточной версии    | Обновить Xcode через App Store                               |

---

## 7. Практические рекомендации

1. **FVM (Flutter Version Manager)** — если работаешь над несколькими проектами или в команде, используй FVM для изоляции версий Flutter по проектам.
2. **VS Code > Android Studio** для начинающих — быстрее запускается, меньше потребляет RAM.
3. **Держи Flutter обновлённым** — `flutter upgrade` регулярно.
4. **Эмулятор vs реальное устройство** — на эмуляторе удобно, но тестируй на реальном устройстве перед релизом.
5. **`flutter clean`** если что-то сломалось после обновления зависимостей — чистит build папку.
