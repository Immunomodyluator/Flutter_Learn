# 16.4 Публикация в Google Play и App Store

## 1. Google Play — подготовка

### Требования:

- Аккаунт Google Play Console ($25 разовый взнос)
- Подписанный AAB (App Bundle)
- Скриншоты (минимум 2 для каждого типа устройства)
- Иконка приложения 512×512 PNG
- Feature graphic 1024×500 PNG
- Политика конфиденциальности (URL)

---

## 2. Google Play — треки публикации

```
Internal Testing → Closed Testing (Alpha) → Open Testing (Beta) → Production
```

| Трек               | Описание                            | Максимум тестировщиков |
| ------------------ | ----------------------------------- | ---------------------- |
| **Internal**       | Сразу доступен, только приглашённые | 100                    |
| **Closed (Alpha)** | Тестовая группа                     | Неограниченно          |
| **Open (Beta)**    | Публичное бета-тестирование         | Неограниченно          |
| **Production**     | Все пользователи                    | Неограниченно          |

```bash
# Сборка AAB
flutter build appbundle --release

# Артефакт
build/app/outputs/bundle/release/app-release.aab
```

---

## 3. Google Play — автоматическая публикация

```ruby
# Fastlane или напрямую через API
supply upload \
  --aab build/app/outputs/bundle/release/app-release.aab \
  --track internal \
  --json_key google-play-key.json
```

**Создание JSON-ключа сервисного аккаунта:**

1. Google Play Console → Setup → API Access.
2. Google Cloud Console → IAM → Service Accounts → Create.
3. Назначить роль "Release Manager".
4. Скачать JSON-ключ.

---

## 4. App Store — подготовка

### Требования:

- Аккаунт Apple Developer ($99/год)
- Подписанный IPA
- Скриншоты для всех необходимых размеров iPhone/iPad
- Иконка приложения (разные размеры, нет прозрачности)
- Описание, ключевые слова, поддержка приложения URL
- Политика конфиденциальности

### Sizes скриншотов (обязательные):

- iPhone 6.7" (1290×2796)
- iPhone 6.5" (1242×2688)
- iPad Pro 12.9" (2048×2732) — если поддерживается iPad

---

## 5. App Store Connect API Key

```json
// fastlane/app_store_connect_api_key.json
{
  "key_id": "D83848D23",
  "issuer_id": "227b0bbf-ada8-458c-9d62-3d3f8354c7b8",
  "key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "in_house": false
}
```

```bash
# Создать в App Store Connect:
# Users and Access → Integrations → App Store Connect API → Keys → +
```

---

## 6. iOS — сборка и публикация

```bash
# Создать ExportOptions.plist
cat > ios/ExportOptions.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" ...>
<plist version="1.0">
<dict>
    <key>method</key>
    <string>app-store</string>
    <key>teamID</key>
    <string>ABCDE12345</string>
    <key>uploadSymbols</key>
    <true/>
    <key>uploadBitcode</key>
    <false/>
</dict>
</plist>
EOF

# Сборка IPA
flutter build ipa --release --export-options-plist=ios/ExportOptions.plist

# Публикация через altool (устаревший) или xcrun notarytool
xcrun altool --upload-app -f build/ios/ipa/myapp.ipa \
  --apiKey D83848D23 \
  --apiIssuer 227b0bbf-ada8-458c-9d62-3d3f8354c7b8
```

---

## 7. Автоматические скриншоты (Fastlane Snapshot)

```ruby
# ios/fastlane/Snapfile
devices([
  "iPhone 15 Pro Max",
  "iPhone SE (3rd generation)",
  "iPad Pro (12.9-inch) (6th generation)",
])

languages(["ru-RU", "en-US"])
scheme("MyAppUITests")
output_directory("fastlane/screenshots")
clear_previous_screenshots(true)
```

```bash
fastlane snapshot          # снять скриншоты
fastlane deliver           # загрузить в App Store Connect
```

---

## 8. Version bump перед релизом

```dart
// pubspec.yaml управляет версией Flutter
version: 1.2.3+456
```

```bash
# Скрипт для автоматического инкремента
# Читает текущую версию → инкрементирует build number
CURRENT=$(grep 'version:' pubspec.yaml | awk '{print $2}')
NAME=$(echo $CURRENT | cut -d+ -f1)
BUILD=$(echo $CURRENT | cut -d+ -f2)
NEW_BUILD=$((BUILD + 1))

sed -i "s/version: .*/version: $NAME+$NEW_BUILD/" pubspec.yaml
```

---

## 9. Чеклист перед публикацией

**Android:**

- [ ] `minSdkVersion` установлен (рекомендуется 21+)
- [ ] `targetSdkVersion` актуальный (API 34+)
- [ ] Проверка на `release` подписи
- [ ] `debuggable false` в релизном build type
- [ ] ProGuard правила не ломают код
- [ ] Manifest не содержит `android:debuggable="true"`

**iOS:**

- [ ] `iOS Deployment Target` ≥ 12.0
- [ ] Все разрешения с описаниями в `Info.plist`
- [ ] Иконка без прозрачности
- [ ] `NSAppTransportSecurity` не отключает ATS без причины
- [ ] Privacy Manifest заполнен (iOS 17+)

---

## 10. Staged Rollout (постепенное развертывание)

```ruby
# Google Play
upload_to_play_store(
  track: "production",
  rollout: "0.2",  # 20% пользователей
)

# Увеличить охват
upload_to_play_store(
  track: "production",
  rollout: "0.5",  # 50%
)

# 100%
upload_to_play_store(
  track: "production",
  rollout: "1.0",
)
```

App Store также поддерживает Phased Release (7 дней, постепенно до 100%).

---

## 11. Типичные ошибки

| Ошибка                                  | Причина                             | Решение                         |
| --------------------------------------- | ----------------------------------- | ------------------------------- |
| `Version code already exists` (Android) | Повторный build number              | Увеличить `versionCode`         |
| `CFBundleVersion must be higher` (iOS)  | Нарастающий Build Number обязателен | Монотонно растущий build number |
| `Missing Privacy Manifest` (iOS 17+)    | Новое требование Apple              | Добавить PrivacyInfo.xcprivacy  |
| Отклонение App Store Review             | Нарушение Guidelines                | Тщательно читать Guidelines     |
| AAB не прошёл Play Store Review         | Нет целевого API                    | Обновить `targetSdkVersion`     |

---

## 12. Рекомендации

1. **Internal → Beta → Production** — не пуши сразу в Production.
2. **Staged rollout** для крупных обновлений — откатить можно быстро.
3. **TestFlight** для iOS-бета-тестирования (до 10 000 тестировщиков).
4. **Скриншоты автоматически** через Fastlane snapshot — экономит часы работы.
5. **Android App Bundle (AAB)** вместо APK — меньше размер загрузки для пользователей.
