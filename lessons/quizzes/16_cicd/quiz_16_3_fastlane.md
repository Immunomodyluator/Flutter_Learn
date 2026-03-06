# Квиз: Fastlane

**Тема:** 16.3 — Fastlane Automation  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое Fastlane и для чего он используется в мобильной разработке?

- A) Быстрый эмулятор для Flutter
- B) Инструмент автоматизации сборки и деплоя мобильных приложений: сборка, тестирование, подпись, загрузка в магазины
- C) Система управления зависимостями
- D) Альтернатива Xcode

<details>
<summary>Ответ</summary>

**B) Автоматизация сборки и деплоя**

```
Fastlane умеет:
✅ Автоматически увеличивать версию/build number
✅ Собирать iOS/Android приложения
✅ Подписывать через Certificates (match)
✅ Запускать тесты (scan)
✅ Загружать в TestFlight / Google Play
✅ Создавать скриншоты (snapshot)
✅ Отправлять уведомления в Slack

Состоит из:
- Fastfile — описание lanes (workflow)
- Appfile — конфигурация приложения (bundle ID, package name)
- Matchfile — конфигурация управления сертификатами
```

```bash
# Установка:
gem install fastlane

# Инициализация в проекте:
cd ios && fastlane init
cd android && fastlane init
```

</details>

---

### Вопрос 2 🟢

Как выглядит базовый `Fastfile` для Flutter?

- A) Fastfile — JSON файл
- B) Ruby DSL с `default_platform`, `platform` блоками и `lane` определениями
- C) YAML файл с task описаниями
- D) Обычный shell script

<details>
<summary>Ответ</summary>

**B) Ruby DSL**

```ruby
# ios/fastlane/Fastfile:
default_platform(:ios)

platform :ios do
  desc "Run tests"
  lane :test do
    scan(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      devices: ["iPhone 15"],
    )
  end

  desc "Build and upload to TestFlight"
  lane :beta do
    # Инкрементировать build number:
    increment_build_number(xcodeproj: "Runner.xcodeproj")

    # Получить сертификаты:
    match(type: "appstore")

    # Собрать:
    gym(
      workspace: "Runner.xcworkspace",
      scheme: "Runner",
      configuration: "Release",
    )

    # Загрузить в TestFlight:
    upload_to_testflight
  end
end
```

</details>

---

### Вопрос 3 🟢

Что делает `gym` action в Fastlane?

- A) Запускает тренажёр для тестирования UI
- B) Собирает iOS приложение (`.ipa`) — аналог `xcodebuild archive`
- C) Управляет Android Gradle сборкой
- D) `gym` — устаревший alias для `build_ios_app`

<details>
<summary>Ответ</summary>

**B) Сборка iOS `.ipa` файла**

```ruby
# gym = build_ios_app (они идентичны):
gym(
  workspace: "Runner.xcworkspace",
  scheme: "Runner",
  configuration: "Release",
  export_method: "app-store",  # app-store, ad-hoc, development, enterprise
  output_directory: "./build",
  output_name: "MyApp.ipa",
  # Настройки подписи:
  codesigning_identity: "Apple Distribution: ..."
)

# Краткая форма — Fastlane читает из Matchfile/Gymfile:
gym

# Создаваемые файлы:
# MyApp.ipa — архив приложения
# MyApp.app.dSYM.zip — символы для crash reporting
```

</details>

---

### Вопрос 4 🟡

Что такое `match` и как он управляет iOS сертификатами?

- A) Match — поиск файлов в проекте
- B) Fastlane action для централизованного управления iOS сертификатами: хранит в Git/S3/Google Cloud, синхронизирует между командой
- C) Match заменяет Xcode Signing
- D) Match только для Enterprise дистрибуции

<details>
<summary>Ответ</summary>

**B) Централизованное хранилище сертификатов**

```ruby
# Matchfile:
git_url("https://github.com/org/certificates.git")
storage_mode("git")
type("appstore")  # development, adhoc, appstore, enterprise
app_identifier(["com.example.app"])
username("developer@example.com")

# Синхронизировать/создать сертификаты:
# fastlane match appstore
# fastlane match development

# В Fastfile:
lane :beta do
  match(type: "appstore", readonly: true)  # только скачать, не создавать
  gym
  upload_to_testflight
end

# Принцип работы:
# 1. Генерирует сертификат + provisioning profile
# 2. Шифрует passphrase
# 3. Коммитит в Git репозиторий сертификатов
# 4. Другие разработчики: match → скачивают и устанавливают локально
```

</details>

---

### Вопрос 5 🟡

Как `supply` используется для публикации Android приложений?

- A) `supply` — загрузка AAB в Google Play Store через Fastlane
- B) `supply` — управление зависимостями
- C) `supply` только для APK, не AAB
- D) `supply` устарел, используйте только `upload_to_play_store`

<details>
<summary>Ответ</summary>

**A) Загрузка в Google Play через Fastlane**

```ruby
# android/fastlane/Fastfile:
platform :android do
  lane :deploy_internal do
    # Собрать через Flutter:
    sh("flutter build appbundle --release")

    # Загрузить в Google Play (Internal Testing):
    upload_to_play_store(  # = supply
      track: "internal",
      aab: "../build/app/outputs/bundle/release/app-release.aab",
      json_key: "google-play-service-account.json",
      skip_upload_apk: true,
      skip_upload_metadata: false,
      skip_upload_images: false,
      skip_upload_screenshots: false,
    )
  end

  lane :promote_to_production do
    # Продвинуть с internal в production без пересборки:
    upload_to_play_store(
      track: "internal",
      track_promote_to: "production",
      json_key: "google-play-service-account.json",
      rollout: "0.1",  # 10% rollout
    )
  end
end
```

</details>

---

### Вопрос 6 🟡

Как автоматически увеличивать build number/version code в Fastlane?

- A) Редактировать `pubspec.yaml` вручную
- B) `increment_build_number` для iOS; для Android — скрипт; `flutter_version` плагин для pubspec
- C) Fastlane не умеет менять версии
- D) `bump_version` действие

<details>
<summary>Ответ</summary>

**B) `increment_build_number` + кастомные action'ы**

```ruby
# iOS — встроенная action:
increment_build_number(
  xcodeproj: "Runner.xcodeproj",
  build_number: ENV["GITHUB_RUN_NUMBER"] || "1",
)

# Android — через sh():
lane :build_android do
  # Читать текущий versionCode:
  version_code = sh("grep -m1 'versionCode' android/app/build.gradle | awk '{print $2}'").strip.to_i
  new_version_code = version_code + 1

  # Обновить:
  sh("sed -i 's/versionCode #{version_code}/versionCode #{new_version_code}/' android/app/build.gradle")

  sh("flutter build appbundle --release --build-number=#{new_version_code}")
end

# pubspec.yaml через fastlane-plugin-flutter_version:
# increment_flutter_version_code
# increment_flutter_version(version_number: "2.0.0")
```

</details>

---

### Вопрос 7 🟡

Как `deliver` используется для публикации iOS приложений в App Store?

- A) `deliver` — синхронизация метаданных и скриншотов + загрузка IPA в App Store Connect
- B) `deliver` только отправляет уведомления
- C) `deliver` устарел, используйте `upload_to_app_store`
- D) `deliver` = `gym`

<details>
<summary>Ответ</summary>

**A) Метаданные + скриншоты + загрузка IPA (= `upload_to_app_store`)**

```ruby
lane :release do
  match(type: "appstore")
  gym

  deliver(
    # Путь к IPA:
    ipa: "./build/MyApp.ipa",

    # Метаданные (из fastlane/metadata/ директории):
    metadata_path: "./fastlane/metadata",

    # Скриншоты (из fastlane/screenshots/ директории):
    screenshots_path: "./fastlane/screenshots",

    # Опции публикации:
    submit_for_review: false,   # не отправлять на ревью автоматически
    automatic_release: false,
    force: true,                # не открывать HTML preview

    # Compliance:
    skip_metadata: false,
    skip_screenshots: false,
  )
end

# Структура fastlane/metadata/:
# en-US/
#   name.txt
#   description.txt
#   keywords.txt
#   release_notes.txt
# ru/
#   ...
```

</details>

---

### Вопрос 8 🔴

Как использовать Fastlane в GitHub Actions?

- A) Fastlane не работает с GitHub Actions
- B) Запустить `fastlane lane_name` в CI step; использовать `ruby/setup-ruby@v1`; передавать secrets через ENV
- C) Только через `fastlane/fastlane-action`
- D) Требуется self-hosted runner

<details>
<summary>Ответ</summary>

**B) `setup-ruby` + `fastlane lane_name`**

```yaml
# .github/workflows/deploy.yml:
jobs:
  deploy-ios:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4

      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.2'
          bundler-cache: true  # кэширует gems

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'

      - name: Install Flutter deps
        run: flutter pub get

      - name: Deploy to TestFlight
        run: bundle exec fastlane beta
        env:
          MATCH_PASSWORD: ${{ secrets.MATCH_PASSWORD }}
          MATCH_GIT_BASIC_AUTHORIZATION: ${{ secrets.MATCH_GIT_AUTH }}
          APP_STORE_CONNECT_API_KEY_ID: ${{ secrets.ASC_KEY_ID }}
          APP_STORE_CONNECT_API_ISSUER_ID: ${{ secrets.ASC_ISSUER_ID }}
          APP_STORE_CONNECT_API_KEY_CONTENT: ${{ secrets.ASC_KEY_CONTENT }}
```

</details>

---

### Вопрос 9 🔴

Как управлять environment-специфичными переменными в Fastlane?

- A) Жёстко кодировать в Fastfile
- B) `.env`, `.env.staging`, `.env.production` файлы; `ENV['VAR']` в Fastfile; `--env` параметр при запуске
- C) Только через Appfile
- D) `fastlane env: production lane_name`

<details>
<summary>Ответ</summary>

**B) `.env` файлы по environment**

```bash
# fastlane/.env (базовые, не секретные):
APP_NAME="MyApp"
BUNDLE_ID="com.example.myapp"

# fastlane/.env.staging:
APP_NAME="MyApp Staging"
BUNDLE_ID="com.example.myapp.staging"
FIREBASE_APP_ID="1:xxx:ios:yyy"

# fastlane/.env.production:
APP_NAME="MyApp"
BUNDLE_ID="com.example.myapp"
FIREBASE_APP_ID="1:xxx:ios:zzz"
```

```ruby
# Fastfile:
lane :deploy do |options|
  env = options[:env] || "staging"

  app_identifier = ENV["BUNDLE_ID"]
  app_name = ENV["APP_NAME"]

  match(app_identifier: app_identifier, type: "appstore")
  gym(scheme: app_name)
  upload_to_testflight
end
```

```bash
# Запуск:
fastlane deploy env:staging
fastlane deploy env:production

# В CI — через --env параметр:
bundle exec fastlane deploy env:production
```

</details>

---

### Вопрос 10 🔴

Как `scan` используется для запуска iOS тестов и как интегрировать результаты в CI?

- A) `scan` — сканер безопасности
- B) Запускает iOS тесты через `xcodebuild test`; генерирует JUnit XML отчёты; интегрируется с GitHub Actions через test reporter
- C) `scan` только для UI тестов
- D) `scan` = `flutter test`

<details>
<summary>Ответ</summary>

**B) iOS тесты + JUnit отчёты**

```ruby
# Fastfile:
lane :test do
  scan(
    workspace: "Runner.xcworkspace",
    scheme: "Runner",
    devices: ["iPhone 15 (17.2)"],
    output_types: "html,junit",         # форматы отчётов
    output_directory: "fastlane/test_output",
    output_files: "report.html,report.xml",
    fail_build: true,                   # падать при провале тестов
    code_coverage: true,
    # Только конкретные тесты:
    # only_testing: "RunnerTests/AuthTests",
  )
end
```

```yaml
# GitHub Actions — публикация результатов:
- name: Run iOS Tests
  run: bundle exec fastlane test

- name: Publish Test Results
  uses: dorny/test-reporter@v1
  if: always()
  with:
    name: iOS Tests
    path: ios/fastlane/test_output/report.xml
    reporter: java-junit
```

</details>
