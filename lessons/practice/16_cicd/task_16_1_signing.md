# 16.1: Подписание приложения — Keystore и сертификаты

> Project: FitMenu | Глава 16 — CI/CD

### 16.1: Настройка подписания для Android Keystore и iOS

🎯 **Цель шага:** Настроить подписание релизной сборки FitMenu для Android (Keystore) и iOS (Distribution Certificate), вынести чувствительные данные в переменные окружения.

---

📝 **Техническое задание:**

**Android — генерация Keystore:**
```bash
keytool -genkey -v \
  -keystore ~/fitmenu-release.keystore \
  -alias fitmenu \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
```

**`android/key.properties` (НЕ коммитить в git!):**
```properties
storePassword=<пароль keystore>
keyPassword=<пароль ключа>
keyAlias=fitmenu
storeFile=/path/to/fitmenu-release.keystore
```

**`android/app/build.gradle`:**
```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    signingConfigs {
        release {
            storeFile     file(keystoreProperties['storeFile'])
            storePassword keystoreProperties['storePassword']
            keyAlias      keystoreProperties['keyAlias']
            keyPassword   keystoreProperties['keyPassword']
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

**`.gitignore`:**
```
android/key.properties
*.keystore
*.jks
ios/Runner.xcarchive
```

**iOS (в Xcode):**
- Product → Archive → Distribute App → App Store Connect
- Автоматическое управление подписью: `AUTOMATICALLY_MANAGE_SIGNING = YES`

---

✅ **Критерии приёмки:**
- [ ] `key.properties` добавлен в `.gitignore`
- [ ] `flutter build apk --release` успешно создаёт подписанный APK
- [ ] `jarsigner -verify -verbose -certs app-release.apk` подтверждает подпись
- [ ] `storeFile` и пароли не хардкодятся в `build.gradle`
- [ ] Для CI: ключи хранятся в secrets (GitHub Actions / Fastlane)

---

💡 **Подсказка:** Никогда не коммить `.keystore` в репозиторий! Для утраченного ключа Play Store невозможно обновить приложение. Храни Keystore в зашифрованном хранилище (1Password, AWS Secrets, GitHub Secrets base64-encoded). `keytool -list -v -keystore file.keystore` проверяет содержимое.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```groovy
// android/app/build.gradle

def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

android {
    compileSdkVersion flutter.compileSdkVersion

    signingConfigs {
        debug {
            storeFile file('debug.keystore')
            storePassword 'android'
            keyAlias 'androiddebugkey'
            keyPassword 'android'
        }
        release {
            // Читаем из key.properties или переменных окружения (для CI)
            if (System.getenv('KEYSTORE_BASE64') != null) {
                // CI: декодируем base64 → временный файл
                def keystoreBytes = System.getenv('KEYSTORE_BASE64').decodeBase64()
                def keystoreFile = file("${buildDir}/ci.keystore")
                keystoreFile.bytes = keystoreBytes
                storeFile keystoreFile
            } else {
                storeFile file(keystoreProperties['storeFile'])
            }
            storePassword System.getenv('KEYSTORE_PASSWORD') ?: keystoreProperties['storePassword']
            keyAlias      System.getenv('KEY_ALIAS')         ?: keystoreProperties['keyAlias']
            keyPassword   System.getenv('KEY_PASSWORD')      ?: keystoreProperties['keyPassword']
        }
    }

    buildTypes {
        release {
            signingConfig    signingConfigs.release
            minifyEnabled    true
            shrinkResources  true
            proguardFiles    getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }
}
```

```yaml
# .github/secrets (в репозитории → Settings → Secrets)
# KEYSTORE_BASE64: base64-encoded keystore file
# KEYSTORE_PASSWORD: пароль keystore
# KEY_ALIAS: alias ключа
# KEY_PASSWORD: пароль ключа

# Encode keystore to base64:
# base64 -i fitmenu-release.keystore | pbcopy
```

</details>
