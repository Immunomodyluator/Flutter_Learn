# 16.1 Подпись приложения

## 1. Android — Keystore

Перед публикацией в Google Play каждый APK/AAB должен быть подписан.

### Создание ключа

```bash
keytool -genkey -v \
  -keystore upload-keystore.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias upload \
  -storepass MyStorePass \
  -keypass MyKeyPass \
  -dname "CN=Ivan, OU=Mobile, O=MyCompany, L=Moscow, ST=Moscow, C=RU"
```

**Хранить `upload-keystore.jks` в безопасном месте — если потерять, нельзя обновить приложение!**

### Настройка подписи в Flutter

```properties
# android/key.properties (не коммитить в git!)
storePassword=MyStorePass
keyPassword=MyKeyPass
keyAlias=upload
storeFile=../upload-keystore.jks
```

```groovy
// android/app/build.gradle
def keyProperties = new Properties()
def keyPropertiesFile = rootProject.file('key.properties')
if (keyPropertiesFile.exists()) {
    keyProperties.load(new FileInputStream(keyPropertiesFile))
}

android {
    signingConfigs {
        release {
            keyAlias keyProperties['keyAlias']
            keyPassword keyProperties['keyPassword']
            storeFile keyProperties['storeFile'] ? file(keyProperties['storeFile']) : null
            storePassword keyProperties['storePassword']
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

```gitignore
# .gitignore — ОБЯЗАТЕЛЬНО исключить
android/key.properties
*.jks
*.keystore
```

### Сборка подписанного релиза

```bash
# AAB (для Google Play — рекомендуется)
flutter build appbundle --release

# APK
flutter build apk --release
```

---

## 2. Android — подпись в CI (GitHub Actions)

```yaml
# .github/workflows/release.yml
- name: Decode keystore
  run: |
    echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/upload-keystore.jks

- name: Create key.properties
  run: |
    echo "storePassword=${{ secrets.KEYSTORE_PASSWORD }}" > android/key.properties
    echo "keyPassword=${{ secrets.KEY_PASSWORD }}" >> android/key.properties
    echo "keyAlias=${{ secrets.KEY_ALIAS }}" >> android/key.properties
    echo "storeFile=../upload-keystore.jks" >> android/key.properties

- name: Build AAB
  run: flutter build appbundle --release

- name: Upload artifact
  uses: actions/upload-artifact@v4
  with:
    name: app-release.aab
    path: build/app/outputs/bundle/release/app-release.aab
```

```bash
# Закодировать keystore для хранения в GitHub Secrets
base64 -w 0 upload-keystore.jks | pbcopy  # macOS
base64 -w 0 upload-keystore.jks           # Linux
```

---

## 3. iOS — Certificates и Provisioning Profiles

iOS требует:

1. **Development Certificate** — для сборки на устройство в процессе разработки.
2. **Distribution Certificate** — для публикации в App Store.
3. **Provisioning Profile** — связывает приложение, сертификат и устройства.

### Типы профилей

| Тип         | Использование                            |
| ----------- | ---------------------------------------- |
| Development | Тестирование на устройстве               |
| Ad Hoc      | Распространение на конкретные устройства |
| App Store   | Публикация в App Store                   |
| Enterprise  | Корпоративное распространение            |

### Настройка вручную

```bash
# Через Xcode: Signing & Capabilities → Automatically manage signing
# Для release-сборки нужен macOS с Xcode

# Сборка IPA
flutter build ipa --release
```

### App Store Connect API Key

```bash
# В App Store Connect: Users and Access → Keys
# Скачать .p8 файл → добавить в CI как secret
```

---

## 4. iOS — подпись в CI с match (Fastlane)

```ruby
# Fastfile
lane :setup_ios_signing do
  match(
    type: 'appstore',
    app_identifier: 'com.mycompany.myapp',
    git_url: 'https://github.com/myorg/certificates',  # приватный репозиторий
    readonly: true,
  )
end
```

`match` хранит сертификаты и профили в зашифрованном git-репозитории.

---

## 5. Version и Build Number

```yaml
# pubspec.yaml
version: 1.2.3+456
#         ^   ^
#         |   build number (versionCode Android, CFBundleVersion iOS)
#         semantic version (versionName Android, CFBundleShortVersionString iOS)
```

```bash
# Сборка с явной версией
flutter build appbundle --build-name=1.2.3 --build-number=456

# Автоматически из git tag в CI:
BUILD_NUMBER=$(git rev-list --count HEAD)
BUILD_NAME=$(git describe --tags --abbrev=0)
flutter build appbundle --build-name=$BUILD_NAME --build-number=$BUILD_NUMBER
```

---

## 6. Типичные ошибки

| Ошибка                 | Причина                        | Решение                                       |
| ---------------------- | ------------------------------ | --------------------------------------------- |
| `keystore not found`   | Неверный путь в key.properties | Проверить относительный путь от android/      |
| Подписан debug-ключом  | `signingConfig` не подключён   | Указать `signingConfigs.release` в buildTypes |
| iOS signing error в CI | Нет сертификата на агенте      | Использовать Fastlane match                   |
| Разные подписи версий  | Новый keystore вместо старого  | Хранить keystore надёжно, не терять           |

---

## 7. Рекомендации

1. **Google Play Upload Key** ≠ App Signing Key — Google Play хранит финальный ключ подписи.
2. **Бэкап keystore** в нескольких местах — потеря = невозможно обновить приложение.
3. **CI/CD**: keystore как base64 в GitHub Secrets, создавать key.properties динамически.
4. **iOS**: Fastlane match — единственный разумный способ управлять сертификатами в команде.
5. **Build number** = монотонно растущий (git commit count или CI run number).
