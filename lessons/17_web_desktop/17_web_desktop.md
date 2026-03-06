# 17. Flutter Web и Desktop

## 1. Суть

Flutter умеет собирать приложения для 6 платформ из одной кодовой базы:

```
         Flutter
    ┌──────────────────┐
    ├─ iOS             │
    ├─ Android         │
    ├─ Web             │
    ├─ macOS           │
    ├─ Windows         │
    └─ Linux           │
    └──────────────────┘
```

---

## 2. Поддержка платформ (2024)

| Платформа | Статус | Renderer         |
| --------- | ------ | ---------------- |
| Android   | Stable | Impeller / Skia  |
| iOS       | Stable | Impeller         |
| Web       | Stable | CanvasKit / HTML |
| macOS     | Stable | Impeller         |
| Windows   | Stable | Impeller         |
| Linux     | Stable | Impeller         |

---

## 3. Включение платформ

```bash
# Добавить поддержку Web
flutter create --platforms=web .

# Добавить Desktop
flutter create --platforms=macos,windows,linux .

# Проверить поддерживаемые платформы
flutter devices

# Запустить на Web
flutter run -d chrome

# Запустить на macOS
flutter run -d macos
```

---

## 4. Условный код под платформу

```dart
import 'package:flutter/foundation.dart';

// Проверка платформы
if (kIsWeb) {
  print('Web');
} else if (defaultTargetPlatform == TargetPlatform.android) {
  print('Android');
} else if (defaultTargetPlatform == TargetPlatform.iOS) {
  print('iOS');
} else if (defaultTargetPlatform == TargetPlatform.macOS) {
  print('macOS');
} else if (defaultTargetPlatform == TargetPlatform.windows) {
  print('Windows');
}
```

---

## 5. Обзор тем раздела

| Файл                  | Тема                                                               |
| --------------------- | ------------------------------------------------------------------ |
| `17_1_web.md`         | Flutter Web: рендереры, SEO, responsive, деплой                    |
| `17_2_desktop.md`     | Flutter Desktop: меню, окна, drag & drop, Squirrel/MSIX            |
| `17_3_adaptive_ui.md` | Adaptive/Responsive UI: LayoutBuilder, NavigationRail, breakpoints |

---

## 6. Рекомендации

1. **Адаптивный UI** — обязателен для многоплатформенного приложения.
2. **Platform-specific код** изолируй через `PlatformService` интерфейс.
3. **Flutter Web** — хорош для инструментов и дашбордов, не для SEO-контента.
4. **Desktop** — отличная история для внутренних инструментов компании.
5. **Тестируй на каждой платформе** — одинаковый код ≠ одинаковое поведение.
