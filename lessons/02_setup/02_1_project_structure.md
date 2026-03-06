# 2.1 Структура Flutter-проекта

## 1. Суть концепции

Стандартная структура Flutter-проекта, создаваемая командой `flutter create`. Понимание каждой папки и файла — обязательная база для работы с реальными проектами.

---

## 2. Дерево проекта

```
my_app/
├── android/          ← нативный Android проект (Kotlin/Java)
├── ios/              ← нативный iOS проект (Swift/ObjC)
├── linux/            ← Desktop Linux
├── macos/            ← Desktop macOS
├── windows/          ← Desktop Windows
├── web/              ← Web приложение
├── lib/              ← ВСЯ твоя Dart/Flutter логика здесь
│   └── main.dart     ← точка входа
├── test/             ← тесты
├── assets/           ← изображения, шрифты, json (создаётся вручную)
├── pubspec.yaml      ← манифест проекта (зависимости, assets)
├── pubspec.lock      ← зафиксированные версии зависимостей
├── analysis_options.yaml  ← правила линтера
└── README.md
```

---

## 3. pubspec.yaml — главный конфиг

```yaml
name: my_app
description: My Flutter application
version: 1.0.0+1 # версия: major.minor.patch+build_number

environment:
  sdk: ">=3.0.0 <4.0.0" # ограничение версии Dart SDK

dependencies:
  flutter:
    sdk: flutter # Flutter framework

  # Сторонние пакеты с pub.dev
  http: ^1.2.0 # ^ = "1.2.0 и выше, но < 2.0.0"
  provider: ^6.1.0
  go_router: ^13.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0 # правила линтера
  mockito: ^5.4.4

flutter:
  uses-material-design: true # подключает иконки Material

  # Подключение assets
  assets:
    - assets/images/
    - assets/data/config.json

  # Подключение шрифтов
  fonts:
    - family: Roboto
      fonts:
        - asset: assets/fonts/Roboto-Regular.ttf
        - asset: assets/fonts/Roboto-Bold.ttf
          weight: 700
```

**Версионирование**: `version: 1.0.0+1`

- `1.0.0` — отображается пользователю (CFBundleShortVersionString / versionName)
- `+1` — build number (CFBundleVersion / versionCode), инкрементируется при каждом релизе

---

## 4. main.dart — точка входа

```dart
import 'package:flutter/material.dart';

void main() {
  // Инициализация Flutter engine
  WidgetsFlutterBinding.ensureInitialized();

  // Дополнительная инициализация (Firebase, DI и т.д.)
  // await Firebase.initializeApp(...);

  runApp(const MyApp()); // запускает дерево виджетов
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'My App',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const HomeScreen(),
    );
  }
}
```

`WidgetsFlutterBinding.ensureInitialized()` — обязателен если делаешь `await` в `main()` до `runApp()`.

---

## 5. Организация папки lib/

Для небольшого проекта:

```
lib/
├── main.dart
├── screens/          ← экраны (страницы приложения)
│   ├── home_screen.dart
│   └── details_screen.dart
├── widgets/          ← переиспользуемые виджеты
│   ├── product_card.dart
│   └── loading_indicator.dart
├── models/           ← data классы
│   └── product.dart
├── services/         ← API, база данных
│   └── api_service.dart
└── utils/            ← вспомогательные функции
    └── formatters.dart
```

Для больших проектов — Feature-first структура (см. урок 13.4).

---

## 6. analysis_options.yaml

```yaml
include: package:flutter_lints/flutter.yaml

linter:
  rules:
    # Дополнительные правила
    prefer_const_constructors: true
    prefer_const_literals_to_create_immutables: true
    avoid_print: true # использовать debugPrint
    always_use_package_imports: true
```

Линтер помогает поддерживать единый стиль кода в команде.

---

## 7. Практические рекомендации

1. **Не коммить `pubspec.lock` игнорировать** — наоборот, коммить его. `pubspec.lock` гарантирует воспроизводимые сборки в команде.
2. **`android/` и `ios/` в .gitignore** — только сгенерированные файлы. Нативный код плагинов туда не входит — его коммитить нужно.
3. **Версию приложения автоматизировать** через CI: `flutter build apk --build-number=$BUILD_NUMBER`.
4. **Assets с конечным `/`** в pubspec.yaml подключают всю папку: `assets/images/` — все файлы в папке.
5. **`flutter pub get`** после клонирования — всегда первая команда перед запуском проекта.
