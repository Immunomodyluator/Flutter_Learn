# Квиз: Flutter CLI

**Тема:** 02.2 — CLI  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какая команда создаёт новый Flutter проект?

- A) `flutter init my_app`
- B) `flutter new my_app`
- C) `flutter create my_app`
- D) `dart create my_app`

<details>
<summary>Ответ</summary>

**C) `flutter create my_app`**

`flutter create` создаёт новый проект с шаблоном. Дополнительные флаги: `--org com.example` (организация), `--template=plugin` (плагин), `--platforms=android,ios,web` (платформы).

</details>

---

### Вопрос 2 🟢

Как запустить приложение в режиме отладки на подключённом устройстве?

- A) `flutter start`
- B) `flutter run`
- C) `flutter debug`
- D) `dart run`

<details>
<summary>Ответ</summary>

**B) `flutter run`**

`flutter run` — компилирует и запускает приложение в debug режиме. `-d device_id` — выбор устройства. `--release` — release режим. `--profile` — profile режим (без DevTools overhead, но с символами).

</details>

---

### Вопрос 3 🟢

Какая команда установит зависимости из `pubspec.yaml`?

- A) `flutter install`
- B) `npm install`
- C) `flutter pub get`
- D) `flutter dependencies`

<details>
<summary>Ответ</summary>

**C) `flutter pub get`**

`flutter pub get` скачивает все зависимости из `pubspec.yaml` в `~/.pub-cache` и создаёт `.dart_tool/package_config.json`. `flutter pub upgrade` — обновляет до последних допустимых версий (по `^`).

</details>

---

### Вопрос 4 🟡

Что делает `flutter build apk --split-per-abi`?

- A) Создаёт несколько APK — по одному для каждой CPU архитектуры
- B) Разделяет APK на части для загрузки
- C) Создаёт APK для каждого flavor
- D) Генерирует несколько APK по размеру экрана

<details>
<summary>Ответ</summary>

**A) Создаёт несколько APK — по одному для каждой CPU архитектуры**

`--split-per-abi` создаёт отдельные APK для `arm64-v8a`, `armeabi-v7a`, `x86_64`. Каждый файл меньше, чем universal APK. Для Google Play используют AAB (`flutter build appbundle`), который автоматически доставляет нужную ABI.

</details>

---

### Вопрос 5 🟡

Что делает `flutter pub run build_runner build`?

- A) Перестраивает Flutter framework
- B) Запускает генераторы кода (json_serializable, freezed, injectable)
- C) Собирает production версию приложения
- D) Обновляет pub зависимости

<details>
<summary>Ответ</summary>

**B) Запускает генераторы кода (json_serializable, freezed, injectable)**

`build_runner` — инструмент кодогенерации. Создаёт `.g.dart` и `.freezed.dart` файлы. `--delete-conflicting-outputs` — удаляет устаревшие файлы. `watch` вместо `build` — отслеживает изменения.

</details>

---

### Вопрос 6 🟡

Как узнать список всех подключённых устройств?

- A) `flutter list`
- B) `flutter devices`
- C) `flutter status`
- D) `adb devices`

<details>
<summary>Ответ</summary>

**B) `flutter devices`**

`flutter devices` показывает все доступные устройства и эмуляторы с их ID. `flutter emulators` — список доступных эмуляторов. `flutter emulators --launch <id>` — запуск эмулятора.

</details>

---

### Вопрос 7 🟡

Что делает `flutter upgrade`?

- A) Обновляет зависимости проекта
- B) Обновляет сам Flutter SDK до последней версии в текущем канале
- C) Переключает канал Flutter (stable/beta/dev)
- D) Обновляет Dart SDK

<details>
<summary>Ответ</summary>

**B) Обновляет сам Flutter SDK до последней версии в текущем канале**

`flutter upgrade` обновляет Flutter framework (не зависимости проекта). `flutter channel stable/beta` — переключение канала. `flutter pub upgrade` — обновление зависимостей проекта.

</details>

---

### Вопрос 8 🔴

Чем `flutter build appbundle` отличается от `flutter build apk`?

- A) AAB работает только на Android 10+
- B) AAB — формат для Google Play с динамической доставкой нужных ресурсов; APK — готовый для установки файл
- C) Никакой разницы — это синонимы
- D) AAB меньше весит только за счёт удаления отладочной информации

<details>
<summary>Ответ</summary>

**B) AAB — формат для Google Play с динамической доставкой нужных ресурсов; APK — готовый для установки файл**

Google Play генерирует оптимизированные APK из AAB для каждого устройства (нужная ABI, плотность экрана, язык). Итоговый APK на устройстве ~15-20% меньше. Google Play требует AAB начиная с 2021 года для новых приложений.

</details>

---

### Вопрос 9 🔴

Что делает флаг `--dart-define=KEY=VALUE` при запуске?

- A) Определяет переменную среды операционной системы
- B) Передаёт compile-time константу, доступную через `String.fromEnvironment('KEY')`
- C) Добавляет переменную в `pubspec.yaml`
- D) Устанавливает переменную для нативного Android/iOS кода

<details>
<summary>Ответ</summary>

**B) Передаёт compile-time константу, доступную через `String.fromEnvironment('KEY')`**

`flutter run --dart-define=API_URL=https://dev.example.com` → в коде: `const apiUrl = String.fromEnvironment('API_URL')`. Значение «вшивается» на этапе компиляции. Используется для конфигурации без хранения секретов в коде.

</details>

---

### Вопрос 10 🔴

Как запустить конкретный тест-файл через CLI?

- A) `flutter test test/my_test.dart`
- B) `flutter run_test test/my_test.dart`
- C) `dart test test/my_test.dart`
- D) `flutter test --file=test/my_test.dart`

<details>
<summary>Ответ</summary>

**A) `flutter test test/my_test.dart`**

`flutter test <path>` запускает конкретный файл. `flutter test` без аргументов — все тесты в `test/`. `flutter test --name "test name"` — фильтр по имени. `flutter test --coverage` — с отчётом покрытия. `dart test` работает только для чистых Dart пакетов без Flutter.

</details>
