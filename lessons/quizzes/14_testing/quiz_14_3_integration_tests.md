# Квиз: Integration Tests

**Тема:** 14.3 — Integration Testing  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Чем integration test отличается от widget test?

- A) Integration тесты работают быстрее
- B) Integration тесты запускаются на реальном устройстве/эмуляторе и тестируют полное приложение со всеми зависимостями
- C) Integration тесты — только для iOS
- D) Нет разницы, оба запускаются в тест-окружении

<details>
<summary>Ответ</summary>

**B) Integration тесты запускаются на реальном устройстве/эмуляторе**

| Тип теста | Среда | Скорость | Изоляция |
|-----------|-------|----------|----------|
| Unit | Dart VM | Очень быстро | Полная |
| Widget | Flutter test environment | Быстро | Частичная (без ОС) |
| Integration | Реальное устройство/эмулятор | Медленно | Минимальная |

Integration тесты проверяют полный пользовательский сценарий: навигацию, сеть, хранилище, нативные API.

</details>

---

### Вопрос 2 🟢

Как настроить проект для integration-тестов?

- A) Добавить `integration_test: ^1.0.0` в `dependencies`
- B) Добавить `integration_test` в `dev_dependencies`; создать `integration_test/app_test.dart` с `IntegrationTestWidgetsFlutterBinding.ensureInitialized()`
- C) Создать файл `test/integration/...`
- D) Никакой настройки не нужно

<details>
<summary>Ответ</summary>

**B) `integration_test` в `dev_dependencies` + инициализация binding**

```yaml
# pubspec.yaml:
dev_dependencies:
  flutter_test:
    sdk: flutter
  integration_test:
    sdk: flutter
```

```dart
// integration_test/app_test.dart:
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:my_app/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('полный сценарий входа', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    // Тест...
  });
}
```

Запуск: `flutter test integration_test/app_test.dart`

</details>

---

### Вопрос 3 🟢

Как запустить integration тест на Android-устройстве или эмуляторе?

- A) `flutter run integration_test/app_test.dart`
- B) `flutter test integration_test/app_test.dart` — запускает на подключённом устройстве/эмуляторе автоматически
- C) `adb shell flutter test`
- D) Только через Xcode Instruments

<details>
<summary>Ответ</summary>

**B) `flutter test integration_test/app_test.dart`**

```bash
# Убедиться, что устройство подключено:
flutter devices

# Запустить на конкретном устройстве:
flutter test integration_test/app_test.dart -d emulator-5554

# Запустить на реальном устройстве:
flutter test integration_test/app_test.dart -d <device_id>

# Запустить все integration тесты:
flutter test integration_test/

# С дополнительными опциями:
flutter test integration_test/app_test.dart --no-pub --verbose
```

Для Firebase Test Lab: упаковать в APK через `flutter build apk --debug`.

</details>

---

### Вопрос 4 🟡

Как взаимодействовать с виджетами в integration тесте?

- A) Только через find.byText()
- B) Через `FlutterDriver` — устаревший подход; сейчас через `WidgetTester` как в widget-тестах: `tap`, `enterText`, `drag`, `pumpAndSettle`
- C) Через Espresso (Android) / XCUITest (iOS)
- D) Через `IntegrationTester.interact()`

<details>
<summary>Ответ</summary>

**B) `WidgetTester` — тот же API что и в widget-тестах**

```dart
testWidgets('создание новой задачи', (tester) async {
  app.main();
  await tester.pumpAndSettle();

  // Нажать FAB для добавления задачи:
  await tester.tap(find.byType(FloatingActionButton));
  await tester.pumpAndSettle();

  // Ввести название:
  await tester.enterText(
    find.byKey(const Key('task_name_field')),
    'Купить продукты',
  );

  // Подтвердить:
  await tester.tap(find.text('Сохранить'));
  await tester.pumpAndSettle();

  // Проверить результат:
  expect(find.text('Купить продукты'), findsOneWidget);

  // Сделать скриншот:
  await binding.takeScreenshot('task_created');
});
```

</details>

---

### Вопрос 5 🟡

Как тестировать сценарий с реальной сетью в integration тесте?

- A) Всегда использовать реальную сеть — это цель integration тестов
- B) Настроить mock-сервер (e.g. WireMock, локальный HTTP-сервер); или использовать специальное тестовое окружение с предсказуемыми данными
- C) Отключить сеть в тесте через аннотацию
- D) Integration тесты не могут делать сетевые запросы

<details>
<summary>Ответ</summary>

**B) Mock-сервер или тестовое окружение**

```dart
// Вариант 1: dart-define для переключения окружения
// Запуск: flutter test integration_test/ --dart-define=ENV=test

// main.dart:
const env = String.fromEnvironment('ENV', defaultValue: 'prod');
final apiBaseUrl = env == 'test'
    ? 'http://localhost:8080'  // локальный mock-сервер
    : 'https://api.example.com';

// Вариант 2: dependency injection для замены в тестах
testWidgets('загружает список пользователей', (tester) async {
  // Подменить зависимость перед запуском app:
  getIt.registerSingleton<ApiClient>(FakeApiClient());
  app.main();
  await tester.pumpAndSettle();

  expect(find.byType(UserTile), findsNWidgets(3));
});

// Вариант 3: Firebase Emulator Suite для Firebase-зависимых тестов
```

</details>

---

### Вопрос 6 🟡

Как делать скриншоты в integration тестах?

- A) `Screenshot.take()` из пакета `screenshots`
- B) `await binding.takeScreenshot('name')` — через `IntegrationTestWidgetsFlutterBinding`
- C) `tester.screenshot()`
- D) `await tester.pump(); debugDumpRenderTree()`

<details>
<summary>Ответ</summary>

**B) `binding.takeScreenshot('name')`**

```dart
void main() {
  final binding = IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  testWidgets('скриншот главного экрана', (tester) async {
    app.main();
    await tester.pumpAndSettle();

    await binding.takeScreenshot('home_screen');

    await tester.tap(find.text('Товары'));
    await tester.pumpAndSettle();

    await binding.takeScreenshot('products_screen');
  });
}

// В CI скриншоты собираются как артефакты:
// flutter test integration_test/screenshots_test.dart \
//   --dart-define=SCREENSHOTS=true
```

Скриншоты используются для визуального регрессионного тестирования и App Store submissions.

</details>

---

### Вопрос 7 🟡

Как запустить integration тесты на Firebase Test Lab?

- A) Напрямую через `firebase test`
- B) Собрать APK (Android) или .app (iOS), загрузить через `gcloud firebase test android run`
- C) Только через GitHub Actions
- D) Firebase Test Lab не поддерживает Flutter

<details>
<summary>Ответ</summary>

**B) Собрать артефакт и запустить через gcloud**

```bash
# Android:
# Собрать тестовый APK:
pushd android
./gradlew app:assembleDebug app:assembleAndroidTest
popd

# Запустить на Firebase Test Lab:
gcloud firebase test android run \
  --type instrumentation \
  --app build/app/outputs/apk/debug/app-debug.apk \
  --test build/app/outputs/apk/androidTest/debug/app-debug-androidTest.apk \
  --device model=Pixel2,version=30,locale=en,orientation=portrait

# iOS:
flutter build ios --simulator
cd ios
xcodebuild -workspace Runner.xcworkspace \
  -scheme Runner -sdk iphonesimulator \
  -derivedDataPath ../build/ios_integ \
  OTHER_SWIFT_FLAGS=-DAUTOMATION build-for-testing
```

</details>

---

### Вопрос 8 🔴

Как изолировать integration тесты от персистентных данных (SharedPreferences, SQLite)?

- A) Удалять базу данных в tearDown()
- B) Использовать временные директории через `path_provider` + `IntegrationTestWidgetsFlutterBinding`; очищать состояние в `setUp()`; использовать разные environment-ы
- C) Integration тесты всегда используют реальные данные
- D) `SharedPreferences.setMockInitialValues()` в integration тестах

<details>
<summary>Ответ</summary>

**B) Временные директории + очистка состояния в setUp()**

```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  setUp(() async {
    // Очистить SharedPreferences:
    final prefs = await SharedPreferences.getInstance();
    await prefs.clear();

    // Удалить тестовую БД:
    final dir = await getApplicationDocumentsDirectory();
    final dbFile = File('${dir.path}/app_test.db');
    if (await dbFile.exists()) await dbFile.delete();

    // Сбросить GetIt:
    await getIt.reset();
    setupDependencies(testMode: true);
  });

  testWidgets('онбординг показывается при первом запуске', (tester) async {
    app.main();
    await tester.pumpAndSettle();
    expect(find.byType(OnboardingScreen), findsOneWidget);
  });

  testWidgets('онбординг не показывается повторно', (tester) async {
    // Записать флаг:
    final prefs = await SharedPreferences.getInstance();
    await prefs.setBool('onboarding_done', true);

    app.main();
    await tester.pumpAndSettle();
    expect(find.byType(HomeScreen), findsOneWidget);
  });
}
```

</details>

---

### Вопрос 9 🔴

Как правильно тестировать Platform Channels в integration тестах?

- A) Platform Channels нельзя тестировать
- B) В integration тестах на реальном устройстве каналы работают нативно; для unit/widget тестов использовать `TestDefaultBinaryMessengerBinding` с mock-хендлерами
- C) Мокировать через `PlatformException`
- D) Только через espresso / XCUITest

<details>
<summary>Ответ</summary>

**B) В integration тестах нативные каналы работают реально; в unit/widget — mock-хендлеры**

```dart
// В widget/unit тестах — мок:
TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
    .setMockMethodCallHandler(
  const MethodChannel('com.example/battery'),
  (MethodCall call) async {
    if (call.method == 'getBatteryLevel') return 85;
    return null;
  },
);

// В integration тестах — реальный канал:
testWidgets('считывает уровень заряда батареи', (tester) async {
  app.main();
  await tester.pumpAndSettle();

  // Нажать кнопку, вызывающую platform channel:
  await tester.tap(find.text('Проверить батарею'));
  await tester.pumpAndSettle();

  // Проверить, что реальный результат отображается:
  expect(find.textContaining('%'), findsOneWidget);
});
```

</details>

---

### Вопрос 10 🔴

Как оптимизировать время выполнения integration-тестов в CI/CD пайплайне?

- A) Запускать только критические сценарии
- B) Параллельный запуск на нескольких устройствах/эмуляторах; разделение тестов по тегам; использование кэша pub packages; headless-эмуляторы; Firebase Test Lab sharding
- C) Отказаться от integration тестов
- D) Запускать только по пятницам

<details>
<summary>Ответ</summary>

**B) Параллелизация + кэширование + sharding**

```yaml
# GitHub Actions — параллельные задания:
jobs:
  integration-tests:
    strategy:
      matrix:
        test-suite: [auth, products, checkout, profile]
    steps:
      - uses: actions/checkout@v4
      - uses: subosito/flutter-action@v2
      - name: Run integration tests
        run: |
          flutter test integration_test/${{ matrix.test-suite }}_test.dart \
            -d emulator-5554

# Теги для выборочного запуска:
testWidgets('smoke test', (tester) async { ... },
  tags: ['smoke', 'critical'],
);

# Запуск по тегу:
# flutter test integration_test/ --tags smoke
```

Стратегия: unit тесты (секунды) → widget тесты (минуты) → integration (нужное устройство).

</details>
