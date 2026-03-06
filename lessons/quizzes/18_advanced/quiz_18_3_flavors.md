# Квиз: Flutter Flavors

**Тема:** 18.3 — Flutter Flavors & Environments  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое flavors в Flutter и зачем они нужны?

- A) Вкусы мороженого в UI
- B) Разные конфигурации одного приложения (dev/staging/prod) с разными app ID, именами, иконками, URL API
- C) Только для Android приложений
- D) Система управления зависимостями

<details>
<summary>Ответ</summary>

**B) Разные конфигурации одного кодовая база — несколько приложений**

```
Типичные flavors:
- dev:     com.example.app.dev, "MyApp Dev", api.dev.example.com
- staging: com.example.app.staging, "MyApp Staging", api.staging.example.com
- prod:    com.example.app, "MyApp", api.example.com

Что различается:
✅ App ID (Bundle ID / Package Name)
✅ Название приложения
✅ Иконка приложения
✅ API URL
✅ Firebase конфигурация (google-services.json / GoogleService-Info.plist)
✅ Feature flags
✅ Logging level
✅ Crashlytics (вкл/выкл)
```

</details>

---

### Вопрос 2 🟢

Как настроить flavors через `--dart-define`?

- A) `flutter build --flavor dev`
- B) `flutter run --dart-define=FLAVOR=dev --dart-define=API_URL=https://api.dev.example.com`; в Dart: `const apiUrl = String.fromEnvironment('API_URL')`
- C) `flutter run --env=dev`
- D) `dart-define` только для Web

<details>
<summary>Ответ</summary>

**B) `--dart-define` + `fromEnvironment()`**

```bash
# Запуск с настройками окружения:
flutter run \
  --dart-define=FLAVOR=dev \
  --dart-define=API_URL=https://api.dev.example.com \
  --dart-define=ENABLE_LOGGING=true

# Production сборка:
flutter build apk --release \
  --dart-define=FLAVOR=prod \
  --dart-define=API_URL=https://api.example.com \
  --dart-define=ENABLE_LOGGING=false
```

```dart
// lib/config.dart:
class AppConfig {
  static const flavor = String.fromEnvironment('FLAVOR', defaultValue: 'dev');
  static const apiUrl = String.fromEnvironment(
    'API_URL',
    defaultValue: 'https://api.dev.example.com',
  );
  static const enableLogging = bool.fromEnvironment(
    'ENABLE_LOGGING',
    defaultValue: true,
  );

  static bool get isDev => flavor == 'dev';
  static bool get isProd => flavor == 'prod';
}
```

</details>

---

### Вопрос 3 🟢

Как использовать `flutter_flavorizr` для автоматической настройки flavors?

- A) `flutter_flavorizr` — только для iOS
- B) Установить пакет, описать flavors в `pubspec.yaml`, запустить `dart run flutter_flavorizr` — автоматически настроит Android/iOS
- C) `dart run flutter_flavorizr --help`
- D) `flutter_flavorizr` создаёт отдельные проекты

<details>
<summary>Ответ</summary>

**B) Автоматическая настройка через `pubspec.yaml`**

```yaml
# pubspec.yaml:
dev_dependencies:
  flutter_flavorizr: ^2.2.3

flavorizr:
  app:
    android:
      flavorDimensions: "environment"
    ios:
      xcconfig: {}

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
    prod:
      app:
        name: "MyApp"
      android:
        applicationId: "com.example.myapp"
        icon: "assets/icons/icon_prod.png"
      ios:
        bundleId: "com.example.myapp"
        icon: "assets/icons/icon_prod.png"
```

```bash
dart run flutter_flavorizr
```

</details>

---

### Вопрос 4 🟡

Как настроить разные `google-services.json` для каждого flavor в Android?

- A) Один `google-services.json` для всех flavors
- B) Разместить файлы в `android/app/src/{flavor}/google-services.json`; Gradle автоматически выберет нужный
- C) `flutter build --google-services=dev.json`
- D) Только через environment variables

<details>
<summary>Ответ</summary>

**B) Файлы в директории flavor**

```
android/
  app/
    src/
      dev/
        google-services.json   ← Firebase dev проект
      staging/
        google-services.json   ← Firebase staging проект
      prod/
        google-services.json   ← Firebase prod проект
    build.gradle
```

```groovy
// android/app/build.gradle:
android {
    flavorDimensions "environment"
    productFlavors {
        dev {
            applicationId "com.example.myapp.dev"
            resValue "string", "app_name", "MyApp Dev"
        }
        staging {
            applicationId "com.example.myapp.staging"
            resValue "string", "app_name", "MyApp Staging"
        }
        prod {
            applicationId "com.example.myapp"
            resValue "string", "app_name", "MyApp"
        }
    }
}

// Gradle автоматически использует google-services.json
// из android/app/src/{flavor}/
```

</details>

---

### Вопрос 5 🟡

Как настроить разные иконки для каждого flavor?

- A) Одна иконка для всех flavors
- B) `flutter_launcher_icons` с конфигурацией под каждый flavor в `pubspec.yaml`; нативные ресурсы в flavor-директориях
- C) Иконки нельзя менять по flavor
- D) Только через `flutter_flavorizr`

<details>
<summary>Ответ</summary>

**B) `flutter_launcher_icons` с flavor конфигурацией**

```yaml
# pubspec.yaml:
dev_dependencies:
  flutter_launcher_icons: ^0.13.0

# Несколько конфигураций:
flutter_launcher_icons:
  image_path_android: "assets/icons/icon_prod.png"
  image_path_ios: "assets/icons/icon_prod.png"

flutter_launcher_icons_dev:
  image_path_android: "assets/icons/icon_dev.png"
  image_path_ios: "assets/icons/icon_dev.png"
  android: true
  ios: true
  flavor: dev

flutter_launcher_icons_staging:
  image_path_android: "assets/icons/icon_staging.png"
  image_path_ios: "assets/icons/icon_staging.png"
  android: true
  ios: true
  flavor: staging
```

```bash
# Генерировать иконки для всех flavors:
dart run flutter_launcher_icons:main -f flutter_launcher_icons*

# Результат:
# android/app/src/dev/res/mipmap-*/ic_launcher.png
# android/app/src/staging/res/mipmap-*/ic_launcher.png
```

</details>

---

### Вопрос 6 🟡

Как запустить приложение с конкретным flavor?

- A) `flutter run --environment=dev`
- B) `flutter run --flavor dev -t lib/main_dev.dart` или `flutter run --flavor prod -t lib/main_prod.dart`
- C) `flutter run --config=dev.yaml`
- D) Flavor указывается только при сборке, не при запуске

<details>
<summary>Ответ</summary>

**B) `--flavor` + `-t` (target entry point)**

```bash
# Запуск dev:
flutter run --flavor dev -t lib/main_dev.dart

# Запуск prod:
flutter run --flavor prod -t lib/main_prod.dart

# Сборка APK для конкретного flavor:
flutter build apk --flavor dev --release -t lib/main_dev.dart
flutter build apk --flavor prod --release -t lib/main_prod.dart

# Сборка AAB:
flutter build appbundle --flavor prod --release -t lib/main_prod.dart

# iOS:
flutter build ios --flavor dev --release -t lib/main_dev.dart
```

```dart
// lib/main_dev.dart:
import 'package:my_app/config.dart';
import 'package:my_app/main.dart' as app;

void main() {
  AppConfig.initialize(
    flavor: Flavor.dev,
    apiUrl: 'https://api.dev.example.com',
  );
  app.main();
}
```

</details>

---

### Вопрос 7 🟡

Как настроить iOS `GoogleService-Info.plist` для разных flavors?

- A) Один файл для всех flavors
- B) Создать `ios/Runner/` подпапки для каждого flavor или использовать скрипт в Xcode Build Phase для копирования нужного файла
- C) `flutter build ios --google-services=dev.plist`
- D) iOS не поддерживает несколько Firebase конфигураций

<details>
<summary>Ответ</summary>

**B) Xcode Build Phase скрипт для копирования**

```bash
# Структура файлов:
ios/
  Runner/
    dev/
      GoogleService-Info.plist
    staging/
      GoogleService-Info.plist
    prod/
      GoogleService-Info.plist
```

```bash
# Xcode Build Phase → + → New Run Script Phase:
# Скрипт (запускается при сборке):

RESOURCES_PATH="$PROJECT_DIR/Runner"
PLIST_FILE="$RESOURCES_PATH/$FLUTTER_BUILD_MODE/GoogleService-Info.plist"

# Если файл есть для режима сборки:
if [ -f "$PLIST_FILE" ]; then
  cp -f "$PLIST_FILE" "$BUILT_PRODUCTS_DIR/$PRODUCT_NAME.app/GoogleService-Info.plist"
fi

# Или через Flutter flavor:
FLAVOR="${PRODUCT_NAME##*_}"  # dev, staging, prod
cp -f "$RESOURCES_PATH/$FLAVOR/GoogleService-Info.plist" \
  "$BUILT_PRODUCTS_DIR/$PRODUCT_NAME.app/GoogleService-Info.plist"
```

</details>

---

### Вопрос 8 🔴

Как реализовать Feature Flags через flavors и remote config?

- A) Feature flags — только для production
- B) Локальные flags через `dart-define`; remote flags через Firebase Remote Config; комбинировать для гибкости
- C) Feature flags — только платная функция
- D) `FeatureFlag.enable('feature_name')`

<details>
<summary>Ответ</summary>

**B) Локальные + remote flags**

```dart
// Конфигурация флагов:
class FeatureFlags {
  // Статические флаги через dart-define:
  static const newCheckoutEnabled = bool.fromEnvironment(
    'NEW_CHECKOUT_ENABLED',
    defaultValue: false,
  );

  // Remote Config флаги (Firebase):
  static bool get darkModeV2Enabled =>
      FirebaseRemoteConfig.instance.getBool('dark_mode_v2');

  static bool get analyticsEnabled =>
      // В dev — отключить аналитику:
      AppConfig.isProd &&
      FirebaseRemoteConfig.instance.getBool('analytics_enabled');
}

// Использование:
if (FeatureFlags.newCheckoutEnabled) {
  return const NewCheckoutScreen();
} else {
  return const LegacyCheckoutScreen();
}

// Firebase Remote Config инициализация:
final remoteConfig = FirebaseRemoteConfig.instance;
await remoteConfig.setConfigSettings(RemoteConfigSettings(
  fetchTimeout: const Duration(seconds: 10),
  minimumFetchInterval: Duration.zero, // dev: сразу
));
await remoteConfig.setDefaults({'dark_mode_v2': false});
await remoteConfig.fetchAndActivate();
```

</details>

---

### Вопрос 9 🔴

Как обеспечить безопасное хранение конфигурации, не коммитя секреты в Git?

- A) Хранить секреты в pubspec.yaml
- B) `--dart-define-from-file=config/dev.env` (Flutter 3.7+); CI secrets; `.gitignore` для config файлов; `.env.example` для документации
- C) Шифровать конфигурацию в коде
- D) Секреты в отдельном закрытом репозитории

<details>
<summary>Ответ</summary>

**B) `--dart-define-from-file` + CI secrets + `.gitignore`**

```json
// config/dev.json (НЕ коммитить, добавить в .gitignore):
{
  "FLAVOR": "dev",
  "API_URL": "https://api.dev.example.com",
  "GOOGLE_MAPS_KEY": "AIzaSy...",
  "STRIPE_KEY": "pk_test_..."
}
```

```bash
# Flutter 3.7+ — загружать конфиг из файла:
flutter run --dart-define-from-file=config/dev.json

# .gitignore:
# config/dev.json
# config/staging.json
# config/prod.json

# config/dev.json.example (коммитить — шаблон без значений):
# {
#   "API_URL": "YOUR_API_URL",
#   "GOOGLE_MAPS_KEY": "YOUR_KEY"
# }
```

```yaml
# GitHub Actions — создавать файл из секрета:
- name: Create prod config
  run: echo '${{ secrets.PROD_CONFIG_JSON }}' > config/prod.json

- name: Build prod
  run: flutter build appbundle --flavor prod \
    --dart-define-from-file=config/prod.json
```

</details>

---

### Вопрос 10 🔴

Как настроить разные Signing конфигурации для разных flavors Android?

- A) Одна подпись для всех flavors
- B) Отдельный `signingConfig` для каждого flavor в `build.gradle`; разные keystores для dev/prod
- C) `flutter build apk --flavor dev --keystore dev.jks`
- D) Flavor не влияет на подпись APK

<details>
<summary>Ответ</summary>

**B) Отдельные `signingConfigs` по flavor**

```groovy
// android/app/build.gradle:
def devProps = new Properties()
def prodProps = new Properties()
if (file('../key_dev.properties').exists()) {
    devProps.load(new FileInputStream(file('../key_dev.properties')))
}
if (file('../key_prod.properties').exists()) {
    prodProps.load(new FileInputStream(file('../key_prod.properties')))
}

android {
    signingConfigs {
        devRelease {
            keyAlias devProps['keyAlias']
            keyPassword devProps['keyPassword']
            storeFile file(devProps['storeFile'])
            storePassword devProps['storePassword']
        }
        prodRelease {
            keyAlias prodProps['keyAlias']
            keyPassword prodProps['keyPassword']
            storeFile file(prodProps['storeFile'])
            storePassword prodProps['storePassword']
        }
    }

    flavorDimensions "environment"
    productFlavors {
        dev {
            applicationId "com.example.myapp.dev"
            signingConfig signingConfigs.devRelease
        }
        prod {
            applicationId "com.example.myapp"
            signingConfig signingConfigs.prodRelease
        }
    }

    buildTypes {
        release {
            // signingConfig переопределён в productFlavors
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt')
        }
    }
}
```

</details>
