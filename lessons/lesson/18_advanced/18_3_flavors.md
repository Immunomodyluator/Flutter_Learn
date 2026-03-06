# 18.3 App Flavors (Варианты сборки)

## 1. Суть

**Flavors** — механизм создания нескольких конфигураций одного приложения (dev / staging / prod) с разными:

- названиями и иконками приложения
- bundle ID / applicationId
- API-ключами и URL-адресами
- Firebase-проектами
- флагами функциональности

---

## 2. Базовый подход — `--dart-define`

```bash
# Передать переменные при сборке
flutter run --dart-define=APP_ENV=dev
flutter build apk --dart-define=APP_ENV=prod --dart-define=API_URL=https://api.example.com
```

```dart
// lib/config/app_config.dart
const _env = String.fromEnvironment('APP_ENV', defaultValue: 'dev');
const _apiUrl = String.fromEnvironment('API_URL', defaultValue: 'https://dev.api.example.com');

class AppConfig {
  static const env = _env;
  static const apiUrl = _apiUrl;
  static const isDev = _env == 'dev';
  static const isProd = _env == 'prod';
}
```

---

## 3. Flutter Flavorizr — автоматизация

```bash
# Установить пакет
flutter pub add --dev flutter_flavorizr
```

```yaml
# pubspec.yaml
flavorizr:
  app:
    android:
      flavorDimensions: "environment"
    ios: {}
  flavors:
    dev:
      app:
        name: "MyApp Dev"
      android:
        applicationId: "com.example.myapp.dev"
        icon: "assets/icons/icon_dev.png"
      ios:
        bundleId: "com.example.myapp.dev"
        icon: "assets/icons/icon_dev.png"
    staging:
      app:
        name: "MyApp Staging"
      android:
        applicationId: "com.example.myapp.staging"
        icon: "assets/icons/icon_staging.png"
      ios:
        bundleId: "com.example.myapp.staging"
        icon: "assets/icons/icon_staging.png"
    prod:
      app:
        name: "MyApp"
      android:
        applicationId: "com.example.myapp"
        icon: "assets/icons/icon.png"
      ios:
        bundleId: "com.example.myapp"
        icon: "assets/icons/icon.png"
```

```bash
# Запустить генерацию
dart run flutter_flavorizr
```

---

## 4. Конфигурация вручную (Android)

```groovy
// android/app/build.gradle
android {
    flavorDimensions "environment"
    productFlavors {
        dev {
            dimension "environment"
            applicationIdSuffix ".dev"
            resValue "string", "app_name", "MyApp Dev"
        }
        staging {
            dimension "environment"
            applicationIdSuffix ".staging"
            resValue "string", "app_name", "MyApp Staging"
        }
        prod {
            dimension "environment"
            resValue "string", "app_name", "MyApp"
        }
    }
}
```

---

## 5. Точки входа — main файлы

```
lib/
  main.dart           ← для IDE (опционально)
  main_dev.dart
  main_staging.dart
  main_prod.dart
  config/
    app_config.dart
```

```dart
// lib/main_dev.dart
import 'package:flutter/material.dart';
import 'app.dart';
import 'config/app_config.dart';

void main() {
  AppConfig.init(
    env: Env.dev,
    apiUrl: 'https://dev.api.example.com',
    enableLogging: true,
  );
  runApp(const App());
}

// lib/main_prod.dart
void main() {
  AppConfig.init(
    env: Env.prod,
    apiUrl: 'https://api.example.com',
    enableLogging: false,
  );
  runApp(const App());
}
```

---

## 6. AppConfig — класс конфигурации

```dart
// lib/config/app_config.dart
enum Env { dev, staging, prod }

class AppConfig {
  static late final AppConfig _instance;

  final Env env;
  final String apiUrl;
  final String supabaseUrl;
  final String supabaseAnonKey;
  final bool enableLogging;

  AppConfig._({
    required this.env,
    required this.apiUrl,
    required this.supabaseUrl,
    required this.supabaseAnonKey,
    required this.enableLogging,
  });

  static void init({
    required Env env,
    required String apiUrl,
    required String supabaseUrl,
    required String supabaseAnonKey,
    required bool enableLogging,
  }) {
    _instance = AppConfig._(
      env: env,
      apiUrl: apiUrl,
      supabaseUrl: supabaseUrl,
      supabaseAnonKey: supabaseAnonKey,
      enableLogging: enableLogging,
    );
  }

  static AppConfig get instance => _instance;

  bool get isDev => env == Env.dev;
  bool get isProd => env == Env.prod;
}
```

---

## 7. Запуск по flavor

```bash
# По flavor
flutter run --flavor dev -t lib/main_dev.dart
flutter run --flavor staging -t lib/main_staging.dart
flutter build appbundle --flavor prod -t lib/main_prod.dart

# Запуск в iOS симуляторе
flutter run --flavor dev -t lib/main_dev.dart -d "iPhone 15"
```

---

## 8. Firebase per-flavor

```bash
# Структура google-services.json для Android
android/app/
  src/
    dev/google-services.json
    staging/google-services.json
    prod/google-services.json   # ← или в app/ для дефолта

# iOS GoogleService-Info.plist через скрипт в Xcode
```

```bash
# iOS Xcode Run Script — копировать нужный plist
cp "${PROJECT_DIR}/firebase/${FLUTTER_BUILD_MODE}/GoogleService-Info.plist" \
   "${BUILT_PRODUCTS_DIR}/${PRODUCT_NAME}.app/GoogleService-Info.plist"
```

---

## 9. VS Code launch.json

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Dev",
      "request": "launch",
      "type": "dart",
      "args": ["--flavor", "dev", "-t", "lib/main_dev.dart"]
    },
    {
      "name": "Staging",
      "request": "launch",
      "type": "dart",
      "args": ["--flavor", "staging", "-t", "lib/main_staging.dart"]
    },
    {
      "name": "Production",
      "request": "launch",
      "type": "dart",
      "args": ["--flavor", "prod", "-t", "lib/main_prod.dart"]
    }
  ]
}
```

---

## 10. Иконки по flavor — flutter_launcher_icons

```yaml
# pubspec.yaml
flutter_icons:
  image_path: "assets/icons/icon.png"
  android: true
  ios: true

flutter_icons_dev:
  image_path: "assets/icons/icon_dev.png"
  android: true
  ios: true
```

```bash
flutter pub run flutter_launcher_icons -f pubspec.yaml
```

---

## 11. Типичные ошибки

| Ошибка                       | Причина                                    | Решение                             |
| ---------------------------- | ------------------------------------------ | ----------------------------------- |
| `Flavor not found`           | Не настроен build.gradle/Xcode             | Добавить flavor в нативный проект   |
| Всегда запускается prod      | Не указан `-t lib/main_dev.dart`           | Явно указывать entry point          |
| Firebase крашится            | Неверный google-services.json              | Проверить структуру `src/{flavor}/` |
| AppConfig не инициализирован | `main.dart` не вызывает `AppConfig.init()` | Вызвать до `runApp()`               |

---

## 12. Рекомендации

1. **3 среды минимум**: dev (разработка) → staging (QA/UAT) → prod (релиз).
2. **Разные appId** для установки нескольких вариантов на одном устройстве.
3. **Разные иконки** — быстро заметить, какая версия запущена.
4. **Dev лого** с баннером "DEV" поверх иконки — визуальный индикатор.
5. **Не хардкодить** конфиденциальные ключи даже в dev — использовать `.env` файлы (gitignored).
