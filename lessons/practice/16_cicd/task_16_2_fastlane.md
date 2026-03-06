# 16.2: Fastlane — автоматизация сборки и деплоя

> Project: FitMenu | Глава 16 — CI/CD

### 16.2: Настройка Fastlane lanes для Android и iOS

🎯 **Цель шага:** Настроить Fastlane lanes для FitMenu: автоматическое версионирование, сборка APK/IPA и загрузка в Firebase App Distribution для тестирования командой.

---

📝 **Техническое задание:**

**Установка:**
```bash
gem install fastlane
cd android && fastlane init
cd ../ios && fastlane init
```

**`android/fastlane/Fastfile`:**
```ruby
default_platform(:android)

platform :android do
  desc "Прогнать тесты"
  lane :test do
    gradle(task: "test")
  end

  desc "Собрать и загрузить в Firebase App Distribution"
  lane :beta do
    # Автоинкремент versionCode
    increment_version_code(
      gradle_file_path: "app/build.gradle",
    )

    gradle(
      task:        "bundle",
      build_type:  "Release",
      properties: {
        "android.injected.signing.store.file"     => ENV["KEYSTORE_PATH"],
        "android.injected.signing.store.password" => ENV["KEYSTORE_PASSWORD"],
        "android.injected.signing.key.alias"      => ENV["KEY_ALIAS"],
        "android.injected.signing.key.password"   => ENV["KEY_PASSWORD"],
      }
    )

    firebase_app_distribution(
      app:              "1:123456789:android:abcdef",
      groups:           "internal-testers",
      release_notes:    "Новые фичи: #{changelog_from_git_commits(commits_count: 5)}",
      firebase_cli_token: ENV["FIREBASE_TOKEN"],
    )
  end

  desc "Загрузить в Play Store (Internal Track)"
  lane :deploy do
    supply(
      track:          "internal",
      aab:            "app/build/outputs/bundle/release/app-release.aab",
      json_key_data:  ENV["PLAY_STORE_JSON_KEY"],
    )
  end
end
```

---

✅ **Критерии приёмки:**
- [ ] `fastlane test` запускает все тесты Android
- [ ] `fastlane beta` собирает подписанный AAB и загружает в Firebase App Distribution
- [ ] `increment_version_code` автоматически увеличивает версию
- [ ] Все чувствительные данные читаются из `ENV[...]`
- [ ] `fastlane --env staging` поддерживает разные конфигурации

---

💡 **Подсказка:** `fastlane env` создаёт `.env.default` файл с переменными. `Appfile` хранит `package_name` и `json_key_file` для Play Store. `match` (iOS) синхронизирует сертификаты через Git-репозиторий.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```ruby
# android/fastlane/Fastfile

require 'date'

default_platform(:android)

platform :android do
  before_all do |lane, options|
    # Убедиться что мы на актуальной ветке
    ensure_git_status_clean unless options[:skip_clean]
  end

  desc "Запустить unit тесты"
  lane :test do
    gradle(task: "test")
  end

  desc "Beta — собрать и отправить тестерам"
  lane :beta do |options|
    version_name = options[:version] || get_version_name(
      gradle_file_path: "app/build.gradle"
    )
    build_number = number_of_commits

    # Обновить versionCode = число коммитов
    android_set_version_code(version_code: build_number)

    gradle(
      task:       "bundle",
      build_type: "Release",
      properties: {
        "android.injected.signing.store.file"     => ENV["KEYSTORE_PATH"],
        "android.injected.signing.store.password" => ENV["KEYSTORE_PASSWORD"],
        "android.injected.signing.key.alias"      => ENV["KEY_ALIAS"],
        "android.injected.signing.key.password"   => ENV["KEY_PASSWORD"],
      },
    )

    # Заметки из последних коммитов
    notes = changelog_from_git_commits(
      commits_count: 10,
      pretty:        "- %s",
      date_format:   "short",
    )

    firebase_app_distribution(
      app:               "1:YOUR_PROJECT:android:YOUR_APP_ID",
      groups:            "beta-testers",
      release_notes:     "#{version_name} (#{build_number})\n#{notes}",
      firebase_cli_token: ENV["FIREBASE_CLI_TOKEN"],
    )

    # Тег в git
    add_git_tag(tag: "v#{version_name}-beta-#{build_number}")
    push_git_tags
  end

  desc "Production — загрузить в Play Store (internal track)"
  lane :deploy do
    supply(
      track:         "internal",
      aab:           lane_context[SharedValues::GRADLE_AAB_OUTPUT_PATH],
      json_key_data: ENV["PLAY_STORE_JSON_KEY"],
    )
  end

  after_all do |lane|
    if lane == :beta || lane == :deploy
      slack(
        message:     "✅ FitMenu #{lane} успешно задеплоен",
        webhook_url: ENV["SLACK_WEBHOOK_URL"],
      ) if ENV["SLACK_WEBHOOK_URL"]
    end
  end
end
```

</details>
