# 18.3: Flavors — dev/staging/prod конфигурации

> Project: FitMenu | Глава 18 — Advanced

### 18.3: Flutter Flavors — разделение dev, staging и production

🎯 **Цель шага:** Настроить три flavor-конфигурации FitMenu (dev/staging/prod) с отдельными Firebase проектами, App ID, иконками и базовыми URL для каждого окружения.

---

📝 **Техническое задание:**

**1. Создать конфигурационный класс:**
```dart
// lib/core/config/app_config.dart

enum Flavor { dev, staging, prod }

class AppConfig {
  static late final AppConfig instance;

  AppConfig._({
    required this.flavor,
    required this.apiBaseUrl,
    required this.appName,
  });

  final Flavor flavor;
  final String apiBaseUrl;
  final String appName;

  static void initialize(Flavor flavor) {
    instance = switch (flavor) {
      Flavor.dev     => AppConfig._(
          flavor:     Flavor.dev,
          apiBaseUrl: 'https://dev-api.fitmenu.app',
          appName:    'FitMenu DEV',
        ),
      Flavor.staging => AppConfig._(
          flavor:     Flavor.staging,
          apiBaseUrl: 'https://staging-api.fitmenu.app',
          appName:    'FitMenu Staging',
        ),
      Flavor.prod    => AppConfig._(
          flavor:     Flavor.prod,
          apiBaseUrl: 'https://api.fitmenu.app',
          appName:    'FitMenu',
        ),
    };
  }

  bool get isDev     => flavor == Flavor.dev;
  bool get isStaging => flavor == Flavor.staging;
  bool get isProd    => flavor == Flavor.prod;
}
```

**2. Точки входа для каждого flavor:**
```dart
// lib/main_dev.dart
void main() {
  AppConfig.initialize(Flavor.dev);
  runApp(const FitMenuApp());
}

// lib/main_staging.dart
void main() {
  AppConfig.initialize(Flavor.staging);
  runApp(const FitMenuApp());
}

// lib/main_prod.dart
void main() {
  AppConfig.initialize(Flavor.prod);
  runApp(const FitMenuApp());
}
```

**3. Запуск нужного flavor:**
```bash
flutter run   --flavor dev     --target lib/main_dev.dart
flutter run   --flavor staging --target lib/main_staging.dart
flutter build apk --flavor prod --target lib/main_prod.dart --release
```

**4. Android `build.gradle` — Product Flavors:**
```groovy
android {
  flavorDimensions "environment"
  productFlavors {
    dev {
      dimension          "environment"
      applicationIdSuffix ".dev"
      versionNameSuffix  "-dev"
      resValue "string", "app_name", "FitMenu DEV"
    }
    staging {
      dimension          "environment"
      applicationIdSuffix ".staging"
      versionNameSuffix  "-staging"
      resValue "string", "app_name", "FitMenu Staging"
    }
    prod {
      dimension "environment"
      resValue "string", "app_name", "FitMenu"
    }
  }
}
```

---

✅ **Критерии приёмки:**
- [ ] `flutter run --flavor dev` запускает приложение с суффиксом `.dev`
- [ ] `AppConfig.instance.apiBaseUrl` возвращает правильный URL для каждого flavor
- [ ] Dev и Prod версии устанавливаются одновременно на одно устройство
- [ ] Debug banner показывается только в `dev` flavor
- [ ] Firebase проекты разделены по flavor (разные `google-services.json`)

---

💡 **Подсказка:** `--dart-define=FLAVOR=dev` — альтернатива flavors через Dart. `const String flavor = String.fromEnvironment('FLAVOR', defaultValue: 'dev')` читает это значение. `flutter_flavor` пакет добавляет визуальный бейдж на экран в non-prod flavors.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/config/app_config.dart

enum Flavor { dev, staging, prod }

/// Конфигурация приложения для текущего flavor-окружения.
///
/// Инициализируй через [AppConfig.initialize] в main.dart
/// до вызова [runApp].
class AppConfig {
  AppConfig._({
    required this.flavor,
    required this.apiBaseUrl,
    required this.appName,
    required this.showDebugBanner,
  });

  static late final AppConfig _instance;
  static AppConfig get instance => _instance;

  final Flavor flavor;
  final String apiBaseUrl;
  final String appName;
  final bool   showDebugBanner;

  bool get isDev     => flavor == Flavor.dev;
  bool get isStaging => flavor == Flavor.staging;
  bool get isProd    => flavor == Flavor.prod;

  static void initialize(Flavor flavor) {
    _instance = switch (flavor) {
      Flavor.dev     => AppConfig._(
          flavor:          Flavor.dev,
          apiBaseUrl:      'https://dev-api.fitmenu.app/v1',
          appName:         'FitMenu DEV',
          showDebugBanner: true,
        ),
      Flavor.staging => AppConfig._(
          flavor:          Flavor.staging,
          apiBaseUrl:      'https://staging-api.fitmenu.app/v1',
          appName:         'FitMenu Staging',
          showDebugBanner: false,
        ),
      Flavor.prod    => AppConfig._(
          flavor:          Flavor.prod,
          apiBaseUrl:      'https://api.fitmenu.app/v1',
          appName:         'FitMenu',
          showDebugBanner: false,
        ),
    };
  }
}
```

```dart
// lib/main_dev.dart
import 'package:fitmenu/core/config/app_config.dart';
import 'package:fitmenu/app.dart';
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'firebase_options_dev.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  AppConfig.initialize(Flavor.dev);
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(const FitMenuApp());
}
```

```dart
// lib/app.dart — использует AppConfig

class FitMenuApp extends StatelessWidget {
  const FitMenuApp({super.key});

  @override
  Widget build(BuildContext context) {
    final config = AppConfig.instance;
    return MaterialApp.router(
      title:                    config.appName,
      debugShowCheckedModeBanner: config.showDebugBanner,
      theme:        AppTheme.lightTheme,
      darkTheme:    AppTheme.darkTheme,
      routerConfig: AppRouter.router,
    );
  }
}
```

</details>
