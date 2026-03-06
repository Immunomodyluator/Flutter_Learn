# 18.2 Создание Flutter-пакетов

## 1. Суть

**Flutter-пакет** — переиспользуемый модуль кода, опубликованный на pub.dev или используемый локально. Два типа:

- **Dart package** — только Dart-код.
- **Flutter plugin** — Dart + нативный код (Kotlin/Swift) через Platform Channels.

---

## 2. Создание пакета

```bash
# Dart пакет (только Dart-код)
flutter create --template=package my_package

# Flutter plugin (с нативным кодом)
flutter create --template=plugin --platforms=android,ios,web my_plugin

# Plugin с поддержкой всех платформ
flutter create --template=plugin --platforms=android,ios,web,macos,windows,linux my_plugin
```

### Структура Dart-пакета

```
my_package/
  lib/
    src/
      widget.dart
      helper.dart
    my_package.dart   ← главный barrel-файл
  test/
    my_package_test.dart
  example/
    lib/
      main.dart       ← пример использования
  pubspec.yaml
  README.md
  CHANGELOG.md
  LICENSE
```

---

## 3. pubspec.yaml пакета

```yaml
name: my_awesome_widget
description: A beautiful widget for Flutter apps
version: 1.0.0
homepage: https://github.com/myorg/my_awesome_widget

environment:
  sdk: ">=3.0.0 <4.0.0"
  flutter: ">=3.0.0"

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0

# Для publication на pub.dev
# repository: https://github.com/myorg/my_awesome_widget
# issue_tracker: https://github.com/myorg/my_awesome_widget/issues
```

---

## 4. Barrel-файл — публичное API

```dart
// lib/my_package.dart
library my_package;

// Экспортировать только публичный API
export 'src/my_widget.dart';
export 'src/my_controller.dart';
export 'src/models/config.dart';

// НЕ экспортировать:
// export 'src/internal/helper.dart';  // внутренняя реализация
```

---

## 5. Локальный пакет (path dependency)

```yaml
# В основном приложении
dependencies:
  my_package:
    path: ../packages/my_package

  # Или в монорепозитории
  feature_auth:
    path: packages/feature_auth
```

---

## 6. Flutter Plugin — нативная реализация

```dart
// lib/my_plugin.dart — Dart API
class MyPlugin {
  static const _channel = MethodChannel('com.myorg/my_plugin');

  static Future<String> getPlatformVersion() async {
    final version = await _channel.invokeMethod<String>('getPlatformVersion');
    return version ?? 'Unknown';
  }
}
```

```kotlin
// android/src/main/kotlin/.../MyPlugin.kt
class MyPlugin : FlutterPlugin, MethodCallHandler {
  private lateinit var channel: MethodChannel

  override fun onAttachedToEngine(flutterPluginBinding: FlutterPlugin.FlutterPluginBinding) {
    channel = MethodChannel(flutterPluginBinding.binaryMessenger, "com.myorg/my_plugin")
    channel.setMethodCallHandler(this)
  }

  override fun onMethodCall(call: MethodCall, result: Result) {
    when (call.method) {
      "getPlatformVersion" -> result.success("Android ${android.os.Build.VERSION.RELEASE}")
      else -> result.notImplemented()
    }
  }

  override fun onDetachedFromEngine(binding: FlutterPlugin.FlutterPluginBinding) {
    channel.setMethodCallHandler(null)
  }
}
```

```swift
// ios/Classes/MyPlugin.swift
public class MyPlugin: NSObject, FlutterPlugin {
  public static func register(with registrar: FlutterPluginRegistrar) {
    let channel = FlutterMethodChannel(
      name: "com.myorg/my_plugin",
      binaryMessenger: registrar.messenger()
    )
    let instance = MyPlugin()
    registrar.addMethodCallDelegate(instance, channel: channel)
  }

  public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
    switch call.method {
    case "getPlatformVersion":
      result("iOS \(UIDevice.current.systemVersion)")
    default:
      result(FlutterMethodNotImplemented)
    }
  }
}
```

---

## 7. Тестирование пакета

```dart
// test/my_widget_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:my_package/my_package.dart';

void main() {
  testWidgets('MyWidget renders correctly', (tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: MyWidget(label: 'Test'),
      ),
    );
    expect(find.text('Test'), findsOneWidget);
  });
}
```

```bash
# Запуск тестов пакета
cd my_package
flutter test

# Проверка pub.dev score
dart pub publish --dry-run
```

---

## 8. Публикация на pub.dev

```bash
# Проверить score (должен быть высоким)
dart pub publish --dry-run

# Проверка:
# ✅ README.md
# ✅ CHANGELOG.md
# ✅ LICENSE (MIT, BSD, Apache)
# ✅ Документация (/// комментарии)
# ✅ Пример в example/
# ✅ Тесты

# Опубликовать
dart pub publish

# Обновить версию
# pubspec.yaml: version: 1.1.0
# CHANGELOG.md: добавить запись о изменениях
```

---

## 9. CHANGELOG.md формат

```markdown
## 1.1.0

- Added `MyWidget.onTap` callback
- Fixed overflow on small screens
- Updated dependencies

## 1.0.0

- Initial release
- `MyWidget` with basic customization
- Dark mode support
```

---

## 10. Документация кода

````dart
/// Красивая кнопка с анимацией нажатия.
///
/// Пример использования:
/// ```dart
/// AnimatedButton(
///   onPressed: () => print('Нажата!'),
///   child: Text('Нажми меня'),
/// )
/// ```
class AnimatedButton extends StatefulWidget {
  /// Коллбэк при нажатии.
  final VoidCallback? onPressed;

  /// Содержимое кнопки.
  final Widget child;

  /// Длительность анимации. По умолчанию 150мс.
  final Duration animationDuration;

  const AnimatedButton({
    super.key,
    required this.onPressed,
    required this.child,
    this.animationDuration = const Duration(milliseconds: 150),
  });
  // ...
}
````

---

## 11. Типичные ошибки

| Ошибка                       | Причина                     | Решение                                      |
| ---------------------------- | --------------------------- | -------------------------------------------- |
| `pub publish` отклонён       | Нет LICENSE или README      | Добавить оба файла                           |
| Низкий pub.dev score         | Нет документации            | Добавить `///` комментарии к публичным API   |
| Пакет не работает в Web      | Platform-specific код       | Использовать conditional imports             |
| Конфликт версий зависимостей | Слишком жёсткие constraints | Использовать `^` вместо фиксированных версий |

---

## 12. Рекомендации

1. **Semantic versioning**: `major.minor.patch` — breaking changes = major.
2. **Пример обязателен** в `example/` — это первое, что смотрит разработчик.
3. **Документируй публичный API** через `///` — pub.dev считает это в score.
4. **`dart pub publish --dry-run`** перед публикацией — предотвратит ошибки.
5. **MIT или BSD лицензия** — наиболее совместимы с open-source экосистемой.
