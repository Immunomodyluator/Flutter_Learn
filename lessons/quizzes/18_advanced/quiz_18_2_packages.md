# Квиз: Package Development

**Тема:** 18.2 — Creating & Publishing Packages  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как создать новый Flutter/Dart пакет?

- A) `flutter create package my_package`
- B) `flutter create --template=package my_package` (Dart only) или `flutter create --template=plugin my_plugin` (с нативным кодом)
- C) `dart pub create my_package`
- D) Создать папку с `pubspec.yaml` вручную

<details>
<summary>Ответ</summary>

**B) `--template=package` или `--template=plugin`**

```bash
# Dart/Flutter пакет (только Dart код):
flutter create --template=package my_utils

# Flutter плагин (с нативным кодом Android/iOS):
flutter create --template=plugin my_native_plugin \
  --platforms=android,ios,web

# Flutter плагин с FFI:
flutter create --template=plugin_ffi my_ffi_plugin

# Структура пакета:
# my_utils/
#   lib/
#     my_utils.dart        ← основной export
#     src/
#       utils.dart         ← реализация
#   test/
#     my_utils_test.dart
#   pubspec.yaml
#   README.md
#   CHANGELOG.md
#   example/               ← пример использования
```

</details>

---

### Вопрос 2 🟢

Что такое `CHANGELOG.md` и как его правильно вести?

- A) История коммитов Git
- B) Файл с историей изменений по версиям в формате Keep a Changelog; обязателен для публикации на pub.dev
- C) Список зависимостей пакета
- D) `CHANGELOG.md` генерируется автоматически из PR

<details>
<summary>Ответ</summary>

**B) История изменений в формате Keep a Changelog**

```markdown
# Changelog

## 2.1.0

* Added `ThemeExtension` support
* Fixed memory leak in `StreamController` usage
* **BREAKING**: Renamed `fetchData()` to `loadData()`

## 2.0.1

* Fixed NPE when `config` is null

## 2.0.0

* **BREAKING**: Removed deprecated `oldMethod()`
* Added Flutter 3.x support
* Improved performance of image loading by 40%

## 1.0.0

* Initial release
```

При публикации на pub.dev — changelog показывается на странице пакета. `## [Unreleased]` для работающих изменений до релиза.

</details>

---

### Вопрос 3 🟢

Как правильно задать версию пакета в `pubspec.yaml`?

- A) Версия — произвольная строка
- B) `major.minor.patch` по semver; `+1` — build metadata; `^2.0.0` в constraints — "совместимый с 2.x"; `>=2.0.0 <3.0.0`
- C) Версия — только целое число
- D) semver обязателен только для pub.dev

<details>
<summary>Ответ</summary>

**B) Semantic Versioning (semver)**

```yaml
# pubspec.yaml:
name: my_package
version: 2.1.3  # MAJOR.MINOR.PATCH

# Правила semver:
# MAJOR — несовместимые изменения API (breaking changes)
# MINOR — новая функциональность, обратно совместимо
# PATCH — исправления багов, обратно совместимо

# Constraints в зависимостях:
dependencies:
  some_package: ^2.0.0     # >=2.0.0 <3.0.0 (caret syntax)
  other_package: '>=1.5.0 <2.0.0'
  exact_package: 1.0.0     # точная версия (не рекомендуется)

# Pre-release:
# version: 3.0.0-beta.1
# version: 3.0.0-rc.2

# MAJOR = 0 → 0.x.y = всё нестабильно
# Первый стабильный релиз: 1.0.0
```

</details>

---

### Вопрос 4 🟡

Что такое Platform Interface Pattern и зачем он нужен?

- A) Паттерн для работы с нативными API на разных платформах
- B) Архитектура Flutter плагинов: `*_platform_interface` пакет определяет абстрактный API; `*` (основной пакет) содержит Dart реализацию + подключение к платформам
- C) Interface для Flutter Web
- D) Только для плагинов с FFI

<details>
<summary>Ответ</summary>

**B) Разделение на `*_platform_interface` + реализации**

```
Структура плагина:

my_plugin/                    ← основной пакет (Dart API)
  lib/my_plugin.dart
  
my_plugin_platform_interface/ ← абстрактный интерфейс
  lib/
    my_plugin_platform_interface.dart  ← abstract class
    
my_plugin_android/            ← Android реализация
  android/...
  
my_plugin_ios/                ← iOS реализация
  ios/...
  
my_plugin_web/                ← Web реализация
  lib/my_plugin_web.dart
```

```dart
// platform_interface:
abstract class MyPluginPlatform extends PlatformInterface {
  static MyPluginPlatform _instance = MethodChannelMyPlugin();
  static set instance(MyPluginPlatform instance) {
    PlatformInterface.verifyToken(instance, _token);
    _instance = instance;
  }
  
  Future<String?> getValue() => throw UnimplementedError();
}
```

</details>

---

### Вопрос 5 🟡

Как правильно экспортировать API пакета через barrel exports?

- A) Экспортировать все файлы по отдельности
- B) `lib/package_name.dart` — главный файл с `export 'src/...'`; `src/` — реализация; публичное API только через exports
- C) `export` в Dart не существует
- D) Все файлы в `lib/` автоматически публичны

<details>
<summary>Ответ</summary>

**B) Barrel file с `export` + приватная `src/`**

```dart
// lib/my_auth_package.dart (barrel — публичное API):
library my_auth_package;

// Экспортировать только публичные части:
export 'src/auth_service.dart';
export 'src/models/user.dart';
export 'src/models/auth_result.dart';
export 'src/widgets/login_button.dart';

// НЕ экспортировать внутренние детали:
// export 'src/internal/token_manager.dart'; // ← скрыто

// lib/src/auth_service.dart:
class AuthService {
  // Публичный API:
  Future<AuthResult> signIn(String email, String password) { ... }

  // Внутренние методы (dart не имеет package-private, но соглашение):
  // Можно использовать @visibleForTesting
}

// В pubspec.yaml тоже контролировать:
// flutter:
//   hide: ['src/*']  # не работает так, но структура src/ — соглашение
```

</details>

---

### Вопрос 6 🟡

Как опубликовать пакет на pub.dev?

- A) `dart publish`
- B) `dart pub publish` после `dart pub publish --dry-run` для проверки
- C) `flutter pub publish --release`
- D) Через веб-интерфейс pub.dev

<details>
<summary>Ответ</summary>

**B) `dart pub publish` после сухой проверки**

```bash
# 1. Проверить перед публикацией (dry run — не публикует):
dart pub publish --dry-run

# Автоматические проверки:
# ✅ pubspec.yaml корректен
# ✅ README.md существует
# ✅ CHANGELOG.md существует
# ✅ LICENSE файл существует
# ✅ Нет предупреждений dart analyze
# ✅ Версия новее чем текущая на pub.dev

# 2. Опубликовать:
dart pub publish

# 3. Авторизоваться через Google аккаунт (первый раз)

# Пункты чеклиста перед публикацией:
# ✅ Осмысленное описание в pubspec.yaml
# ✅ homepage, repository URL
# ✅ screenshots (3-5 изображений для pub.dev)
# ✅ topics: [flutter, dart, ...]
# ✅ example/ с рабочим кодом
# ✅ Покрытие тестами
# ✅ README с Quick Start
```

</details>

---

### Вопрос 7 🟡

Как обрабатывать breaking changes при обновлении мажорной версии?

- A) Переименовать публичные методы без предупреждения
- B) Добавить `@Deprecated` → опубликовать minor версию → убрать в major версии; задокументировать в CHANGELOG; migration guide
- C) Публиковать отдельный пакет `my_package_v2`
- D) Breaking changes запрещены в pub.dev пакетах

<details>
<summary>Ответ</summary>

**B) Deprecate → minor release → remove в major**

```dart
// 1. Добавить @Deprecated (версия X.Y.Z):
@Deprecated('Use loadData() instead. Will be removed in 3.0.0')
Future<Data> fetchData() => loadData();

// Новый метод:
Future<Data> loadData() async { ... }

// 2. Опубликовать 2.5.0 с @Deprecated
// Пользователи видят предупреждение компилятора

// 3. В 3.0.0 — убрать deprecated методы
```

```markdown
// CHANGELOG.md:
## 3.0.0

**BREAKING CHANGES:**
* Removed `fetchData()` — use `loadData()` instead
* `Config` constructor now requires `apiKey` parameter

**Migration guide:**
* Replace all `fetchData()` calls with `loadData()`
* Pass `apiKey` to Config: `Config(apiKey: '...')`
See: https://github.com/org/my_package/blob/main/MIGRATION.md
```

</details>

---

### Вопрос 8 🔴

Как тестировать Flutter пакет с нативными зависимостями?

- A) Только integration тестами на реальном устройстве
- B) Mock platform channel через `TestDefaultBinaryMessengerBinding`; unit тесты для Dart кода; integration тесты для нативного поведения
- C) `flutter test` не поддерживает тестирование плагинов
- D) Только через Firebase Test Lab

<details>
<summary>Ответ</summary>

**B) Mock channels для unit тестов + integration для нативного**

```dart
// Тест Dart API с мок platform channel:
void main() {
  TestWidgetsFlutterBinding.ensureInitialized();

  group('MyPlugin', () {
    const channel = MethodChannel('com.example/my_plugin');

    setUp(() {
      TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
          .setMockMethodCallHandler(channel, (MethodCall call) async {
        switch (call.method) {
          case 'getValue':
            return 'mock_value';
          case 'setValue':
            return null;
          default:
            return null;
        }
      });
    });

    tearDown(() {
      TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
          .setMockMethodCallHandler(channel, null);
    });

    test('getValue returns value', () async {
      final plugin = MyPlugin();
      expect(await plugin.getValue(), equals('mock_value'));
    });
  });
}
```

</details>

---

### Вопрос 9 🔴

Как настроить pub.dev score и получить максимальный рейтинг?

- A) Pub score вычисляется только по количеству скачиваний
- B) `dart pub publish --dry-run` показывает проблемы; pub score = Likes + Pub Points + Popularity; Points за: документацию, анализ, платформы, null safety
- C) Score нельзя повлиять
- D) Нужно платить за повышение score

<details>
<summary>Ответ</summary>

**B) Pub Points — автоматический скор по критериям**

```
Pub Points (max 160):
📝 Follow Dart file conventions (20 pt)
📚 Provide documentation (20 pt)  — >20% pub members documented
⬆️ Support up-to-date dependencies (20 pt)
🖥️ Support multiple platforms (20 pt) — Android, iOS, Web, Desktop
📦 Pass analysis checks (50 pt) — 0 errors, 0 warnings
🔒 Support sound null safety (20 pt)
✅ Pass static analysis (10 pt)

Практика для 160/160:
✅ dartdoc комментарии на всех публичных классах/методах
✅ flutter analyze — 0 issues
✅ environment: sdk: '>=3.0.0 <4.0.0'
✅ platforms в pubspec.yaml
✅ Актуальные зависимости (^latest)
✅ Все тесты проходят
✅ Example в example/ директории
```

</details>

---

### Вопрос 10 🔴

Как организовать monorepo с несколькими связанными пакетами?

- A) Monorepo для Flutter пакетов невозможен
- B) `melos` инструмент для управления монорепозиторием: bootstrap, publish, run scripts по всем пакетам
- C) Только `workspace` в pubspec.yaml
- D) Создать отдельный Git репозиторий для каждого пакета

<details>
<summary>Ответ</summary>

**B) `melos` для Flutter monorepo**

```yaml
# melos.yaml в корне monorepo:
name: my_monorepo

packages:
  - packages/**
  - apps/**

scripts:
  analyze:
    run: melos exec -- dart analyze
    description: Analyze all packages

  test:
    run: melos exec --concurrency=4 -- flutter test
    description: Run all tests

  publish:
    run: melos publish --no-dry-run
    description: Publish all packages

  bootstrap:
    description: Link local packages
```

```bash
# Установка:
dart pub global activate melos

# Инициализация:
melos bootstrap  # устанавливает зависимости и создаёт symlinks

# Запустить скрипты:
melos run analyze
melos run test

# Структура:
# my_monorepo/
#   melos.yaml
#   packages/
#     my_auth/
#     my_ui_kit/
#     my_network/
#   apps/
#     my_app/
#     my_admin/
```

</details>
