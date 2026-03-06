# 16.3: GitHub Actions — CI pipeline для FitMenu

> Project: FitMenu | Глава 16 — CI/CD

### 16.3: GitHub Actions workflow: тесты + сборка + деплой

🎯 **Цель шага:** Создать GitHub Actions CI/CD pipeline для FitMenu: автозапуск тестов на PR, сборка release APK на push в main и деплой в Firebase App Distribution.

---

📝 **Техническое задание:**

Создай `.github/workflows/flutter-ci.yml`:

```yaml
name: FitMenu CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    name: Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.0'
          channel: 'stable'
          cache: true

      - name: Зависимости
        run: flutter pub get

      - name: Analyze
        run: flutter analyze

      - name: Unit + Widget тесты
        run: flutter test --coverage

      - name: Загрузить coverage в Codecov
        uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info

  build_android:
    name: Build Android
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.19.0'
          cache: true

      - name: Decode keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode > android/app/release.keystore

      - name: Build AAB
        env:
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
        run: |
          flutter build appbundle --release \
            --dart-define=FLAVOR=production

      - name: Upload AAB artifact
        uses: actions/upload-artifact@v3
        with:
          name: release-aab
          path: build/app/outputs/bundle/release/app-release.aab
```

**Матрица тестов для нескольких версий Flutter:**
```yaml
strategy:
  matrix:
    flutter-version: ['3.16.0', '3.19.0']
```

---

✅ **Критерии приёмки:**
- [ ] `flutter analyze` + `flutter test` запускаются на каждый PR
- [ ] Build запускается только при пуше в `main`
- [ ] `secrets.KEYSTORE_BASE64` используется для подписания (не хардкод)
- [ ] AAB артефакт загружается через `upload-artifact`
- [ ] `needs: test` гарантирует что build после успешных тестов

---

💡 **Подсказка:** `subosito/flutter-action` с `cache: true` кэширует Flutter SDK между запусками. `--dart-define=FLAVOR=production` передаёт параметры сборки. `concurrency` в workflow отменяет предыдущий запуск при новом пуше в ту же ветку.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```yaml
# .github/workflows/flutter-ci.yml

name: FitMenu CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Отменяем предыдущий запуск при новом пуше
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  FLUTTER_VERSION: '3.19.0'

jobs:
  # ─── Тесты ──────────────────────────────────────────────────────────────
  test:
    name: 🧪 Tests
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          channel: stable
          cache: true

      - run: flutter pub get
      - run: flutter analyze --fatal-infos
      - run: flutter test --coverage --reporter=compact

      - uses: codecov/codecov-action@v3
        with:
          file: coverage/lcov.info
          fail_ci_if_error: false

  # ─── Android Build ──────────────────────────────────────────────────────
  build_android:
    name: 🤖 Android Build
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-java@v3
        with:
          distribution: temurin
          java-version: 17

      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ env.FLUTTER_VERSION }}
          cache: true

      - name: Decode Keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode \
            > android/app/release.keystore

      - name: Flutter Build AAB
        env:
          KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
          KEY_ALIAS:         ${{ secrets.KEY_ALIAS }}
          KEY_PASSWORD:      ${{ secrets.KEY_PASSWORD }}
        run: |
          flutter build appbundle \
            --release \
            --obfuscate \
            --split-debug-info=build/debug-symbols \
            --dart-define=FLAVOR=production \
            -P storeFile=release.keystore \
            -P storePassword="$KEYSTORE_PASSWORD" \
            -P keyAlias="$KEY_ALIAS" \
            -P keyPassword="$KEY_PASSWORD"

      - uses: actions/upload-artifact@v3
        with:
          name: fitmenu-release-${{ github.sha }}
          path: build/app/outputs/bundle/release/app-release.aab
          retention-days: 14

  # ─── Deploy to Firebase ─────────────────────────────────────────────────
  deploy_firebase:
    name: 🚀 Firebase Distribution
    needs: build_android
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: fitmenu-release-${{ github.sha }}

      - uses: wzieba/Firebase-Distribution-Github-Action@v1
        with:
          appId:       ${{ secrets.FIREBASE_APP_ID }}
          token:       ${{ secrets.FIREBASE_CLI_TOKEN }}
          groups:      internal-testers
          file:        app-release.aab
          releaseNotes: "Commit ${{ github.sha }}: ${{ github.event.head_commit.message }}"
```

</details>
