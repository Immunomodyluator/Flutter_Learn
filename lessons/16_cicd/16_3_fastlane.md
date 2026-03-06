# 16.3 Fastlane

## 1. Суть

**Fastlane** — инструмент автоматизации для iOS и Android: подписи сертификатов, сборки, скриншоты, публикация в App Store и Google Play. Работает локально и в CI.

```bash
# Установка
gem install fastlane
# или
brew install fastlane
```

---

## 2. Инициализация

```bash
# В корне Flutter-проекта
cd ios && fastlane init
# или
cd android && fastlane init
```

Создаётся `Fastfile`, `Appfile`, опционально `Matchfile`.

---

## 3. Android — Fastfile

```ruby
# android/fastlane/Fastfile

default_platform(:android)

platform :android do

  # Константы
  PACKAGE_NAME = "com.mycompany.myapp"

  desc "Запустить тесты"
  lane :test do
    gradle(task: "test")
  end

  desc "Собрать AAB"
  lane :build do
    gradle(
      task: "bundle",
      build_type: "Release",
      properties: {
        "android.injected.signing.store.file" => ENV["KEYSTORE_PATH"],
        "android.injected.signing.store.password" => ENV["KEYSTORE_PASSWORD"],
        "android.injected.signing.key.alias" => ENV["KEY_ALIAS"],
        "android.injected.signing.key.password" => ENV["KEY_PASSWORD"],
      }
    )
  end

  desc "Загрузить во Internal Testing"
  lane :deploy_internal do
    build
    upload_to_play_store(
      track: "internal",
      package_name: PACKAGE_NAME,
      aab: "app/build/outputs/bundle/release/app-release.aab",
      json_key_data: ENV["PLAY_STORE_JSON_KEY"],
      skip_upload_apk: true,
      release_status: "draft",
    )
  end

  desc "Продвинуть в Production"
  lane :promote_to_production do
    upload_to_play_store(
      package_name: PACKAGE_NAME,
      track: "internal",
      track_promote_to: "production",
      json_key_data: ENV["PLAY_STORE_JSON_KEY"],
      rollout: "0.1",  # 10% пользователей
    )
  end

end
```

---

## 4. iOS — Fastfile

```ruby
# ios/fastlane/Fastfile

default_platform(:ios)

APP_IDENTIFIER = "com.mycompany.myapp"
TEAM_ID = "ABCDE12345"

platform :ios do

  desc "Синхронизировать сертификаты (match)"
  lane :sync_signing do
    match(
      type: "appstore",
      app_identifier: APP_IDENTIFIER,
      team_id: TEAM_ID,
      git_url: ENV["MATCH_GIT_URL"],
      git_basic_authorization: Base64.strict_encode64(ENV["MATCH_GIT_AUTH"]),
      readonly: is_ci,  # в CI только читаем
    )
  end

  desc "Собрать IPA"
  lane :build do
    sync_signing
    build_app(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      export_method: "app-store",
    )
  end

  desc "Загрузить в TestFlight"
  lane :beta do
    build
    upload_to_testflight(
      api_key_path: "fastlane/app_store_connect_api_key.json",
      skip_waiting_for_build_processing: true,
    )
  end

  desc "Опубликовать в App Store"
  lane :release do
    build
    upload_to_app_store(
      api_key_path: "fastlane/app_store_connect_api_key.json",
      submit_for_review: true,
      automatic_release: false,
      force: true,                   # не открывать браузер
      skip_screenshots: true,
    )
  end

end
```

---

## 5. Fastlane match — управление сертификатами

```ruby
# ios/fastlane/Matchfile
git_url(ENV["MATCH_GIT_URL"])
app_identifier(["com.mycompany.myapp"])
username("developer@mycompany.com")
type("appstore")
```

```bash
# Инициализировать (первый раз)
fastlane match init

# Сгенерировать и сохранить новые сертификаты
fastlane match appstore

# Прочитать существующие (для CI)
fastlane match appstore --readonly
```

---

## 6. Flutter-специфичный Fastfile

```ruby
# fastlane/Fastfile в корне Flutter-проекта

platform :android do
  lane :flutter_build_android do
    # Сначала собираем через Flutter
    sh("cd .. && flutter build appbundle --release \
        --build-name=#{ENV['BUILD_NAME']} \
        --build-number=#{ENV['BUILD_NUMBER']}")

    upload_to_play_store(
      track: "internal",
      aab: "../build/app/outputs/bundle/release/app-release.aab",
      json_key_data: ENV["PLAY_STORE_JSON_KEY"],
    )
  end
end
```

---

## 7. Appfile

```ruby
# ios/fastlane/Appfile
app_identifier("com.mycompany.myapp")
apple_id("developer@example.com")
team_id("ABCDE12345")

# android/fastlane/Appfile
json_key_file("fastlane/google-play-key.json")
package_name("com.mycompany.myapp")
```

---

## 8. Переменные окружения

```bash
# .env.default (не коммитить!)
KEYSTORE_PASSWORD=MyPass
KEY_ALIAS=upload
KEY_PASSWORD=MyPass
PLAY_STORE_JSON_KEY={"type":"service_account",...}

# Использование:
fastlane deploy_internal --env default
```

---

## 9. Запуск в GitHub Actions

```yaml
- name: Deploy to Internal Track
  run: |
    cd android
    bundle exec fastlane deploy_internal
  env:
    KEYSTORE_PATH: upload-keystore.jks
    KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
    KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
    KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
    PLAY_STORE_JSON_KEY: ${{ secrets.PLAY_STORE_JSON_KEY }}
```

---

## 10. Полезные действия Fastlane

```ruby
# Инкремент build number
increment_build_number(
  build_number: ENV["BUILD_NUMBER"],
  xcodeproj: "Runner.xcodeproj"
)

# Снять скриншоты
capture_screenshots

# Slack-уведомление
slack(
  message: "Новый билд готов!",
  slack_url: ENV["SLACK_WEBHOOK_URL"],
)

# Отправить в Firebase App Distribution
firebase_app_distribution(
  app: ENV["FIREBASE_APP_ID"],
  testers: "qa@mycompany.com",
  release_notes: "Новые функции: ...",
)
```

---

## 11. Типичные ошибки

| Ошибка                             | Причина                    | Решение                                 |
| ---------------------------------- | -------------------------- | --------------------------------------- |
| `bundle exec fastlane` не работает | Ruby/Bundler не установлен | `gem install bundler && bundle install` |
| Сертификат не найден (iOS)         | `match` не синхронизирован | `fastlane match appstore --readonly`    |
| `json_key_data` ошибка             | Неверный формат JSON       | Убедиться, что это строка, не файл      |
| `sh()` не находит flutter          | PATH в Environment         | Использовать полный путь к flutter      |

---

## 12. Рекомендации

1. **`bundle exec fastlane`** вместо `fastlane` — изолированное окружение Ruby.
2. **`Gemfile`** в репозитории — фиксирует версию Fastlane для команды.
3. **`match`** — единственный разумный способ управлять iOS-сертификатами в команде.
4. **Локальный тест** перед CI: `fastlane deploy_internal` локально.
5. **`skip_waiting_for_build_processing: true`** для TestFlight — CI не ждёт Apple (20-30 мин).
