# Квиз: App Signing

**Тема:** 16.1 — Code Signing & Certificates  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое keystore в Android и зачем он нужен?

- A) Хранилище для SharedPreferences
- B) Файл, содержащий приватный ключ для подписи APK/AAB — Google Play требует одинаковую подпись для обновлений приложения
- C) Сертификат HTTPS соединения
- D) Ключ для шифрования данных пользователя

<details>
<summary>Ответ</summary>

**B) Приватный ключ для подписи APK/AAB**

```bash
# Создать keystore:
keytool -genkey -v \
  -keystore ~/upload-keystore.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias upload

# Параметры:
# -validity 10000 — дней действия (рекомендуется 10000+)
# -alias — псевдоним ключа внутри keystore
# Пароль keystore + пароль ключа — РАЗНЫЕ, оба нужно хранить

# КРИТИЧНО:
# ❌ Никогда не добавлять keystore в Git!
# ✅ Хранить в надёжном месте (password manager, secrets vault)
# ✅ Сделать резервную копию — потеря = невозможность обновить приложение
```

</details>

---

### Вопрос 2 🟢

Как настроить подпись Android приложения в Flutter?

- A) Редактировать `AndroidManifest.xml`
- B) Создать `android/key.properties` с путём/паролями; настроить `android/app/build.gradle` для чтения этих свойств
- C) `flutter sign --android keystore.jks`
- D) Только через Android Studio GUI

<details>
<summary>Ответ</summary>

**B) `key.properties` + `build.gradle`**

```properties
# android/key.properties (НЕ коммитить в Git!):
storePassword=myStorePass
keyPassword=myKeyPass
keyAlias=upload
storeFile=/Users/ivan/upload-keystore.jks
```

```groovy
// android/app/build.gradle:
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ?
                file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
        }
    }
}
```

</details>

---

### Вопрос 3 🟢

В чём разница между Upload Key и App Signing Key в Google Play?

- A) Это одно и то же
- B) Upload Key — ключ разработчика для загрузки в Play Console; App Signing Key — ключ Google для финальной подписи, показываемой пользователям
- C) App Signing Key — только для платных приложений
- D) Upload Key — для debug, App Signing Key — для release

<details>
<summary>Ответ</summary>

**B) Upload Key — твой; App Signing Key — Google's**

```
Google Play App Signing (обязательно с Aug 2021):

Разработчик → подписывает Upload Key → загружает в Play Console
       ↓
Google Play → переподписывает своим App Signing Key → раздаёт пользователям

Преимущества:
✅ Если потерял Upload Key — можно сгенерировать новый через Play Console
✅ Google защищает App Signing Key в Hardware Security Module
✅ Пользователи получают подпись Google

App Signing Key fingerprint:
- Можно найти в Play Console → Setup → App signing
- Нужен для Firebase, Google Maps API, некоторых SDK
- SHA-1: в Play Console → Release → Setup → App signing
```

</details>

---

### Вопрос 4 🟡

Как настроить iOS подпись через Provisioning Profile?

- A) `flutter build ios --sign certificate.p12`
- B) В Apple Developer Portal создать App ID + Certificate + Provisioning Profile; в Xcode → Signing & Capabilities; или автоматически через "Automatically manage signing"
- C) Только через Fastlane Match
- D) iOS не требует подписи для TestFlight

<details>
<summary>Ответ</summary>

**B) Apple Developer Portal + Xcode Signing**

```
Типы сертификатов:
1. Development Certificate — для разработки/тестирования на устройстве
2. Distribution Certificate — для App Store / Ad Hoc

Типы Provisioning Profiles:
1. Development — Debug сборки, конкретные устройства
2. Ad Hoc — Тестирование на <= 100 устройств
3. App Store — Загрузка в App Store Connect
4. Enterprise — Корпоративное распространение

В Xcode (Runner.xcworkspace):
- Signing & Capabilities → Team → выбрать
- Bundle Identifier должен совпадать с App ID
- "Automatically manage signing" — Xcode создаёт всё сам
```

```bash
# Сборка для App Store из терминала:
flutter build ios --release
cd ios
xcodebuild -workspace Runner.xcworkspace \
  -scheme Runner -configuration Release \
  -archivePath build/Runner.xcarchive archive
```

</details>

---

### Вопрос 5 🟡

Как хранить signing credentials в CI/CD безопасно?

- A) Добавить keystore в репозиторий — закрытый репозиторий безопасен
- B) Хранить в GitHub Secrets / переменных CI; передавать через env variables; keystore кодировать в Base64
- C) Создавать новый keystore при каждой сборке
- D) Использовать debug keystore для release

<details>
<summary>Ответ</summary>

**B) CI Secrets + Base64 encoding**

```yaml
# GitHub Actions — хранение keystore:

# 1. Закодировать keystore в Base64:
# base64 upload-keystore.jks > keystore_base64.txt
# Скопировать содержимое в GitHub Secret: KEYSTORE_BASE64

# 2. В workflow:
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Decode keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode \
            > android/app/upload-keystore.jks

      - name: Create key.properties
        run: |
          echo "storePassword=${{ secrets.STORE_PASSWORD }}" > android/key.properties
          echo "keyPassword=${{ secrets.KEY_PASSWORD }}" >> android/key.properties
          echo "keyAlias=${{ secrets.KEY_ALIAS }}" >> android/key.properties
          echo "storeFile=upload-keystore.jks" >> android/key.properties

      - name: Build release
        run: flutter build appbundle --release
```

</details>

---

### Вопрос 6 🟡

Как настроить iOS подпись в CI без GUI Xcode?

- A) Нельзя подписать iOS без GUI
- B) Импортировать certificate (.p12) и provisioning profile через `security` команду macOS; или использовать Fastlane Match
- C) `xcodebuild --sign-manually`
- D) iOS подпись требует физический Mac

<details>
<summary>Ответ</summary>

**B) `security` команда + provisioning profile**

```bash
# Установить сертификат:
security create-keychain -p "" build.keychain
security import certificate.p12 -k build.keychain \
  -P "$CERT_PASSWORD" -A
security list-keychains -s build.keychain
security default-keychain -s build.keychain
security unlock-keychain -p "" build.keychain
security set-key-partition-list -S apple-tool:,apple: \
  -s -k "" build.keychain

# Установить Provisioning Profile:
mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
cp profile.mobileprovision \
  ~/Library/MobileDevice/Provisioning\ Profiles/$(uuidgen).mobileprovision

# Сборка:
flutter build ios --release --no-codesign
cd ios
xcodebuild -workspace Runner.xcworkspace \
  -scheme Runner -configuration Release \
  CODE_SIGN_IDENTITY="Apple Distribution: ..." \
  PROVISIONING_PROFILE_SPECIFIER="MyApp Distribution"
```

</details>

---

### Вопрос 7 🟡

Что такое `.aab` (Android App Bundle) и чем он отличается от `.apk`?

- A) `.aab` — это зашифрованный `.apk`
- B) AAB — формат для Google Play, содержит все конфигурации; Play собирает оптимальный APK для каждого устройства (split APK)
- C) AAB — только для Android 12+
- D) APK и AAB — одинаковые форматы

<details>
<summary>Ответ</summary>

**B) AAB → Play собирает оптимизированные APK для каждого устройства**

```bash
# Собрать AAB (требуется Google Play с Aug 2021):
flutter build appbundle --release

# Собрать APK (для прямой установки / других сторов):
flutter build apk --release --split-per-abi

# AAB преимущества:
# - Размер скачиваемого приложения на 15-20% меньше
# - Dynamic delivery: дополнительные модули по запросу
# - Автоматически исключает неиспользуемые ресурсы

# Тестирование AAB локально через bundletool:
bundletool build-apks \
  --bundle=build/app/outputs/bundle/release/app-release.aab \
  --output=app.apks \
  --ks=upload-keystore.jks \
  --ks-pass=pass:password \
  --ks-key-alias=upload \
  --key-pass=pass:password
bundletool install-apks --apks=app.apks
```

</details>

---

### Вопрос 8 🔴

Как настроить разные keystore для разных flavors (dev/staging/prod)?

- A) Один keystore для всех environments
- B) Отдельный keystore для каждого flavor; отдельные `key_dev.properties`, `key_prod.properties`; в `build.gradle` — выбор по flavor
- C) `flutter build apk --flavor prod --keystore prod.jks`
- D) Flavor не влияет на подпись

<details>
<summary>Ответ</summary>

**B) Отдельные keystore и properties по flavor**

```groovy
// android/app/build.gradle:
android {
    flavorDimensions "environment"
    productFlavors {
        dev {
            applicationId "com.example.app.dev"
        }
        prod {
            applicationId "com.example.app"
        }
    }

    signingConfigs {
        devRelease {
            def props = loadProperties('key_dev.properties')
            // ... dev keystore
        }
        prodRelease {
            def props = loadProperties('key_prod.properties')
            // ... prod keystore
        }
    }

    buildTypes {
        release {
            productFlavors.dev.signingConfig = signingConfigs.devRelease
            productFlavors.prod.signingConfig = signingConfigs.prodRelease
        }
    }
}

// Сборка конкретного flavor:
// flutter build appbundle --flavor prod --release
```

</details>

---

### Вопрос 9 🔴

Как проверить подпись APK/AAB и получить fingerprint?

- A) `flutter check-signature app.apk`
- B) `keytool -printcert -jarfile app.apk` или `apksigner verify --verbose --print-certs app.apk`
- C) Только через Play Console
- D) `openssl verify app.aab`

<details>
<summary>Ответ</summary>

**B) `apksigner` или `keytool`**

```bash
# Проверить подпись APK:
apksigner verify --verbose --print-certs build/app/outputs/apk/release/app-release.apk

# Получить SHA-1 и SHA-256 fingerprint:
keytool -printcert -jarfile app.apk

# Получить fingerprint из keystore (нужен для Firebase, Google APIs):
keytool -list -v \
  -keystore upload-keystore.jks \
  -alias upload

# SHA-1 fingerprint для debug (нужен при регистрации в Firebase):
keytool -list -v \
  -keystore ~/.android/debug.keystore \
  -alias androiddebugkey \
  -storepass android \
  -keypass android

# Результат:
# SHA1: AA:BB:CC:DD:...
# SHA256: 11:22:33:44:...
```

</details>

---

### Вопрос 10 🔴

Что делать если потерян upload keystore для Google Play?

- A) Загрузить новое приложение с новым ID
- B) Если включён Google Play App Signing — можно запросить новый upload key в Play Console; если не включён и потерян единственный ключ — только новый app listing
- C) Google восстанавливает ключ по email
- D) Использовать debug keystore как замену

<details>
<summary>Ответ</summary>

**B) Play App Signing → запросить новый upload key в Play Console**

```
Сценарий 1: Google Play App Signing включён (рекомендуется):
1. Play Console → Release → Setup → App signing
2. "Request upload key reset"
3. Google генерирует новый upload key
4. Скачать новый keystore и использовать для загрузки
5. App Signing Key (Google's) остаётся тем же — пользователи не замечают

Сценарий 2: Google Play App Signing НЕ включён (старые приложения):
1. Если есть резервная копия keystore — восстановить
2. Если нет — приложение нельзя обновить с тем же package name
3. Придётся создавать новое приложение с новым ID
   → потеря всех оценок, reviews, install base

Лучшие практики:
✅ Включить Google Play App Signing (обязательно для новых приложений)
✅ Хранить keystore в 3 местах (password manager, encrypted drive, team vault)
✅ Документировать все пароли в 1Password / Bitwarden
✅ Тестировать восстановление раз в год
```

</details>
