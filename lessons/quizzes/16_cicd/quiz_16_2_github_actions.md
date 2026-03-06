# Квиз: GitHub Actions для Flutter

**Тема:** 16.2 — GitHub Actions CI/CD  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как создать базовый GitHub Actions workflow для Flutter?

- A) Добавить `.github/flutter.yml`
- B) Создать файл в `.github/workflows/` с расширением `.yml`; определить `on`, `jobs`, `steps`
- C) `flutter ci init`
- D) Добавить в `pubspec.yaml` секцию `ci`

<details>
<summary>Ответ</summary>

**B) `.github/workflows/ci.yml`**

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
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Flutter
        uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.24.0'
          channel: 'stable'

      - name: Install dependencies
        run: flutter pub get

      - name: Analyze
        run: flutter analyze

      - name: Run tests
        run: flutter test --coverage
```

</details>

---

### Вопрос 2 🟢

Как кэшировать pub packages в GitHub Actions?

- A) Кэш pub packages не работает в CI
- B) `actions/cache@v4` с ключом на основе `pubspec.lock`; или через параметр `cache: true` в `subosito/flutter-action`
- C) `flutter pub get --cache`
- D) Кэш автоматически сохраняется между runs

<details>
<summary>Ответ</summary>

**B) `cache: true` в flutter-action или `actions/cache`**

```yaml
# Способ 1: встроенный кэш в subosito/flutter-action:
- uses: subosito/flutter-action@v2
  with:
    flutter-version: '3.24.0'
    cache: true  # кэширует Flutter SDK

# Способ 2: кэш pub packages:
- name: Cache pub packages
  uses: actions/cache@v4
  with:
    path: |
      ~/.pub-cache
      .dart_tool
    key: ${{ runner.os }}-pub-${{ hashFiles('pubspec.lock') }}
    restore-keys: |
      ${{ runner.os }}-pub-

# Экономия: flutter pub get занимает 30-120 секунд
# С кэшем: 2-5 секунд (cache hit)
```

</details>

---

### Вопрос 3 🟢

Как использовать секреты в GitHub Actions workflow?

- A) Писать секреты прямо в `yml` файл
- B) Добавить в `Settings → Secrets → Actions`; использовать в workflow как `${{ secrets.SECRET_NAME }}`
- C) `env: SECRET=value` в workflow
- D) Секреты хранятся в `.env` файле

<details>
<summary>Ответ</summary>

**B) `Settings → Secrets` + `${{ secrets.NAME }}`**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      # Переменные окружения из secrets:
      FIREBASE_TOKEN: ${{ secrets.FIREBASE_TOKEN }}

    steps:
      - name: Decode Android keystore
        run: |
          echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 --decode \
            > android/app/upload-keystore.jks

      - name: Setup signing
        run: |
          cat << EOF > android/key.properties
          storePassword=${{ secrets.STORE_PASSWORD }}
          keyPassword=${{ secrets.KEY_PASSWORD }}
          keyAlias=${{ secrets.KEY_ALIAS }}
          storeFile=upload-keystore.jks
          EOF

      - name: Build AAB
        run: flutter build appbundle --release

# Secrets никогда не логируются — заменяются на ***
# Repository secrets: доступны всем branches
# Environment secrets: только для конкретного environment (prod/staging)
```

</details>

---

### Вопрос 4 🟡

Как настроить matrix strategy для тестирования на нескольких версиях Flutter?

- A) Создать отдельный workflow для каждой версии
- B) `strategy.matrix` — определить список значений; `${{ matrix.flutter-version }}` для доступа к значению
- C) `flutter test --all-versions`
- D) Matrix strategy недоступна для flutter-action

<details>
<summary>Ответ</summary>

**B) `strategy.matrix`**

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        flutter-version: ['3.22.0', '3.24.0', 'stable']
        os: [ubuntu-latest, macos-latest, windows-latest]
        # Исключить некоторые комбинации:
        exclude:
          - os: windows-latest
            flutter-version: '3.22.0'

    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: ${{ matrix.flutter-version }}

      - run: flutter pub get
      - run: flutter test

# Результат: 3 × 3 - 1 = 8 параллельных jobs
# fail-fast: false — продолжать другие jobs при провале одного
    strategy:
      fail-fast: false
      matrix:
        ...
```

</details>

---

### Вопрос 5 🟡

Как сохранять артефакты сборки (APK, AAB, IPA) в GitHub Actions?

- A) Артефакты автоматически сохраняются
- B) `actions/upload-artifact@v4` с указанием пути к файлу; скачать через `actions/download-artifact@v4`
- C) `flutter upload --artifacts`
- D) Только через GitHub Releases

<details>
<summary>Ответ</summary>

**B) `actions/upload-artifact`**

```yaml
- name: Build Android AAB
  run: flutter build appbundle --release

- name: Upload Android AAB
  uses: actions/upload-artifact@v4
  with:
    name: android-release-${{ github.sha }}
    path: build/app/outputs/bundle/release/app-release.aab
    retention-days: 30  # хранить 30 дней

# Для iOS:
- name: Build iOS IPA
  run: |
    flutter build ios --release --no-codesign
    # дополнительные шаги подписи...

- name: Upload iOS IPA
  uses: actions/upload-artifact@v4
  with:
    name: ios-release
    path: build/ios/ipa/*.ipa

# Скачать в следующем job:
- name: Download AAB
  uses: actions/download-artifact@v4
  with:
    name: android-release-${{ github.sha }}
    path: build/
```

</details>

---

### Вопрос 6 🟡

Как разделить CI на несколько jobs с зависимостями?

- A) Все шаги в одном job
- B) `needs: [job-name]` — job запустится только после завершения зависимых jobs
- C) `depends-on: job-name`
- D) Порядок в файле определяет последовательность

<details>
<summary>Ответ</summary>

**B) `needs:` для зависимостей между jobs**

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
      - run: flutter test

  build-android:
    needs: test  # запустится только если test прошёл
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
      - run: flutter build appbundle --release

  build-ios:
    needs: test  # параллельно с build-android
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
      - run: flutter build ios --release --no-codesign

  deploy:
    needs: [build-android, build-ios]  # ждёт оба
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to stores
        run: echo "Deploying..."
```

</details>

---

### Вопрос 7 🟡

Как запускать workflow только для конкретных файлов или веток?

- A) `if: github.ref == 'refs/heads/main'`
- B) `on.push.paths` — фильтр по файлам; `on.push.branches` — фильтр по веткам; `on.push.paths-ignore` — исключить пути
- C) `filter: paths: [lib/**]`
- D) Только через `when` условия в steps

<details>
<summary>Ответ</summary>

**B) `paths`, `branches`, `paths-ignore`**

```yaml
on:
  push:
    branches:
      - main
      - 'release/*'    # все release ветки
    paths:
      - 'lib/**'       # только при изменениях в lib/
      - 'pubspec.yaml' # или pubspec.yaml
      - 'pubspec.lock'
    paths-ignore:
      - '**.md'        # игнорировать markdown файлы
      - 'docs/**'

  # Запускать по расписанию (каждую ночь):
  schedule:
    - cron: '0 0 * * *'  # 00:00 UTC каждый день

  # Ручной запуск с параметрами:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Deploy to environment'
        required: true
        default: 'staging'
        type: choice
        options: [staging, production]
```

</details>

---

### Вопрос 8 🔴

Как реализовать автоматическую публикацию в Google Play через GitHub Actions?

- A) `flutter deploy --store google-play`
- B) `r0adkll/upload-google-play@v1` action — принимает AAB + service account JSON
- C) Только через Fastlane
- D) Play Store API не поддерживает автоматическую загрузку

<details>
<summary>Ответ</summary>

**B) `r0adkll/upload-google-play` action**

```yaml
- name: Build release AAB
  run: flutter build appbundle --release

- name: Upload to Google Play (Internal Testing)
  uses: r0adkll/upload-google-play@v1
  with:
    serviceAccountJsonPlainText: ${{ secrets.SERVICE_ACCOUNT_JSON }}
    packageName: com.example.myapp
    releaseFiles: build/app/outputs/bundle/release/app-release.aab
    track: internal          # internal, alpha, beta, production
    status: draft            # draft, completed, inProgress
    # Обновить release notes:
    whatsNewDirectory: distribution/whatsnew

# Настройка Service Account в Google Play Console:
# 1. Google Play Console → Setup → API access
# 2. Link Google Cloud project
# 3. Create Service Account с правами "Release manager"
# 4. Скачать JSON ключ → добавить как GitHub Secret
```

</details>

---

### Вопрос 9 🔴

Как оптимизировать время выполнения Flutter CI pipeline?

- A) Увеличить количество шагов
- B) Кэшировать Flutter SDK и pub packages; параллельные jobs; `--concurrency` ограничение; пропускать ненужные шаги по `paths`
- C) Использовать только самоходящие runner'ы (self-hosted)
- D) Выключить тесты в CI

<details>
<summary>Ответ</summary>

**B) Кэш + параллелизм + умные фильтры**

```yaml
jobs:
  # Параллельные jobs: test и lint одновременно
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          cache: true  # кэш Flutter SDK

      - name: Pub cache
        uses: actions/cache@v4
        with:
          path: ~/.pub-cache
          key: pub-${{ hashFiles('pubspec.lock') }}

      - run: flutter pub get
      - run: flutter test --concurrency=4  # параллельные тесты

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
        with:
          cache: true
      - run: flutter pub get
      - run: flutter analyze --no-pub
      - run: dart format --output=none --set-exit-if-changed .

# Типичное время:
# Без кэша: flutter pub get = 60-120s, Flutter setup = 60-180s
# С кэшем: pub get = 2-5s, Flutter setup = 5-15s
# Экономия: 3-5 минут на pipeline
```

</details>

---

### Вопрос 10 🔴

Как настроить code coverage отчёт в GitHub PR через Codecov?

- A) `flutter test --coverage` автоматически публикует в PR
- B) `codecov/codecov-action@v4` — загружает `coverage/lcov.info` в Codecov; показывает Coverage diff в PR
- C) Только через SonarQube
- D) GitHub Actions не поддерживает coverage отчёты

<details>
<summary>Ответ</summary>

**B) `codecov/codecov-action`**

```yaml
- name: Run tests with coverage
  run: flutter test --coverage

# Удалить сгенерированные файлы из coverage:
- name: Filter coverage
  run: |
    lcov --remove coverage/lcov.info \
      '*.g.dart' '*.freezed.dart' '*_test.dart' \
      -o coverage/lcov.info

- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v4
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    file: coverage/lcov.info
    flags: unittests
    fail_ci_if_error: true
    # Показывать patch coverage в PR:
    patch_coverage_fail_threshold: 80

# codecov.yml в корне проекта:
# coverage:
#   status:
#     project:
#       default:
#         target: 80%
#     patch:
#       default:
#         target: 80%
```

</details>
