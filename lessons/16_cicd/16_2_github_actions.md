# 16.2 GitHub Actions для Flutter

## 1. Суть

GitHub Actions — встроенный CI/CD в GitHub. Запускается автоматически при push, PR или по расписанию. Конфигурация — YAML-файлы в `.github/workflows/`.

---

## 2. Базовый CI — тесты и анализ

```yaml
# .github/workflows/ci.yml
name: Flutter CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Test & Analyze
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.22.0"
          channel: stable
          cache: true

      - name: Install dependencies
        run: flutter pub get

      - name: Check formatting
        run: dart format --output=none --set-exit-if-changed .

      - name: Run analyzer
        run: flutter analyze

      - name: Run tests
        run: flutter test --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: coverage/lcov.info
```

---

## 3. CD — сборка Android AAB

```yaml
# .github/workflows/release-android.yml
name: Release Android

on:
  push:
    tags:
      - "v*.*.*" # Запускается при git tag v1.2.3

jobs:
  build-android:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.22.0"
          channel: stable
          cache: true

      - name: Install dependencies
        run: flutter pub get

      - name: Decode keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/upload-keystore.jks

      - name: Create key.properties
        run: |
          cat > android/key.properties << EOF
          storePassword=${{ secrets.KEYSTORE_PASSWORD }}
          keyPassword=${{ secrets.KEY_PASSWORD }}
          keyAlias=${{ secrets.KEY_ALIAS }}
          storeFile=../upload-keystore.jks
          EOF

      - name: Get version from tag
        id: version
        run: |
          TAG="${GITHUB_REF#refs/tags/v}"
          BUILD_NUM=$(git rev-list --count HEAD)
          echo "name=$TAG" >> $GITHUB_OUTPUT
          echo "number=$BUILD_NUM" >> $GITHUB_OUTPUT

      - name: Build AAB
        run: |
          flutter build appbundle --release \
            --build-name=${{ steps.version.outputs.name }} \
            --build-number=${{ steps.version.outputs.number }}

      - name: Upload AAB artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-release.aab
          path: build/app/outputs/bundle/release/app-release.aab
          retention-days: 7
```

---

## 4. CD — сборка iOS IPA (macOS runner)

```yaml
# .github/workflows/release-ios.yml
name: Release iOS

on:
  push:
    tags: ["v*.*.*"]

jobs:
  build-ios:
    runs-on: macos-latest # iOS только на macOS

    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.22.0"
          channel: stable

      - name: Install dependencies
        run: flutter pub get

      - name: Install Apple certificate
        uses: apple-actions/import-codesign-certs@v3
        with:
          p12-file-base64: ${{ secrets.IOS_CERT_BASE64 }}
          p12-password: ${{ secrets.IOS_CERT_PASSWORD }}

      - name: Install provisioning profile
        run: |
          mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
          echo "${{ secrets.IOS_PROV_PROFILE_BASE64 }}" | base64 --decode \
            > ~/Library/MobileDevice/Provisioning\ Profiles/profile.mobileprovision

      - name: Build IPA
        run: flutter build ipa --release --export-options-plist=ios/ExportOptions.plist

      - name: Upload to TestFlight
        uses: apple-actions/upload-testflight-build@v1
        with:
          app-path: build/ios/ipa/myapp.ipa
          issuer-id: ${{ secrets.APPSTORE_ISSUER_ID }}
          api-key-id: ${{ secrets.APPSTORE_KEY_ID }}
          api-private-key: ${{ secrets.APPSTORE_PRIVATE_KEY }}
```

---

## 5. Matrix стратегия — тесты на нескольких Flutter-версиях

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        flutter-version: ["3.19.0", "3.22.0", "stable"]

    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ matrix.flutter-version }}

      - run: flutter pub get
      - run: flutter test
```

---

## 6. Кеширование Flutter и зависимостей

```yaml
- uses: subosito/flutter-action@v2
  with:
    flutter-version: "3.22.0"
    cache: true # кешировать Flutter SDK

# Кеширование pub зависимостей
- uses: actions/cache@v4
  with:
    path: |
      ~/.pub-cache
      .dart_tool
    key: ${{ runner.os }}-pub-${{ hashFiles('pubspec.lock') }}
    restore-keys: ${{ runner.os }}-pub-
```

---

## 7. Уведомления в Telegram/Slack

```yaml
- name: Notify Telegram on success
  if: success()
  uses: appleboy/telegram-action@master
  with:
    to: ${{ secrets.TELEGRAM_CHAT_ID }}
    token: ${{ secrets.TELEGRAM_TOKEN }}
    message: |
      ✅ Build ${{ github.ref_name }} успешно собран
      Commit: ${{ github.event.head_commit.message }}

- name: Notify on failure
  if: failure()
  uses: appleboy/telegram-action@master
  with:
    to: ${{ secrets.TELEGRAM_CHAT_ID }}
    token: ${{ secrets.TELEGRAM_TOKEN }}
    message: "❌ Build ${{ github.ref_name }} провалился"
```

---

## 8. Workflow для Feature Branch

```yaml
on:
  pull_request:
    branches: [main, develop]

jobs:
  pr-checks:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: "3.22.0"
          cache: true

      - run: flutter pub get
      - run: dart format --output=none --set-exit-if-changed .
      - run: flutter analyze --fatal-infos
      - run: flutter test

      # Блокировать merge если размер AAB вырос на >5%
      - name: Build size check
        run: |
          flutter build appbundle --release
          SIZE=$(wc -c < build/app/outputs/bundle/release/app-release.aab)
          echo "AAB size: $SIZE bytes"
```

---

## 9. Типичные ошибки

| Ошибка                 | Причина                         | Решение                                 |
| ---------------------- | ------------------------------- | --------------------------------------- | ---------------------------------- |
| `License not accepted` | Android SDK лицензии            | Добавить шаг `yes                       | flutter doctor --android-licenses` |
| Медленный pipeline     | Нет кеша                        | Добавить `cache: true` в flutter-action |
| iOS runner дорогой     | macOS-агенты тарифицируются x10 | Запускать iOS-сборку только для тегов   |
| Secrets в логах        | `echo $SECRET` выводит значение | Использовать `::add-mask::$SECRET`      |

---

## 10. Рекомендации

1. **Ветки**: CI на каждый PR; CD только из main/develop или тегов.
2. **`cache: true`** в flutter-action — ускоряет pipeline на 2-3 минуты.
3. **Секреты через GitHub Secrets** — никогда в коде или yaml.
4. **iOS дорого** — запускай macOS только для релизных сборок.
5. **Timeout**: `timeout-minutes: 30` на job — защита от зависших workflows.
