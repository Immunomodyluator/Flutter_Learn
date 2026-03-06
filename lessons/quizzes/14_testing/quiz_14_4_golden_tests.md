# Квиз: Golden Tests

**Тема:** 14.4 — Golden Tests  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое golden test в Flutter?

- A) Тест производительности (золотой стандарт скорости)
- B) Тест, сравнивающий скриншот виджета с эталонным изображением (golden file)
- C) UI-тест без кода — только через записанные клики
- D) Тест на iOS (Apple = золото)

<details>
<summary>Ответ</summary>

**B) Сравнение скриншота с эталонным изображением**

```dart
testWidgets('ProfileCard выглядит корректно', (tester) async {
  await tester.pumpWidget(
    const MaterialApp(
      home: ProfileCard(
        name: 'Алиса',
        role: 'Flutter Developer',
        avatarUrl: null, // использует placeholder
      ),
    ),
  );

  // Сравнить с эталонным файлом:
  await expectLater(
    find.byType(ProfileCard),
    matchesGoldenFile('goldens/profile_card.png'),
  );
});
```

При первом запуске создаётся эталонный файл. При последующих — сравнивается пиксель за пикселем.

</details>

---

### Вопрос 2 🟢

Как создать или обновить эталонный golden-файл?

- A) Удалить старый файл и запустить тест
- B) `flutter test --update-goldens` — обновляет все golden-файлы
- C) `flutter golden --create`
- D) Вручную сделать скриншот и положить в `test/goldens/`

<details>
<summary>Ответ</summary>

**B) `flutter test --update-goldens`**

```bash
# Создать/обновить все golden-файлы:
flutter test --update-goldens

# Обновить конкретный файл:
flutter test test/widgets/profile_card_test.dart --update-goldens

# Обновить тесты с определённым тегом:
flutter test --update-goldens --tags golden

# ВАЖНО: коммитить golden-файлы в Git!
# Они хранятся рядом с тестом или в поддиректории goldens/
```

При CI должна быть установлена платформа, идентичная той, на которой создавались golden-файлы — иначе пиксельные различия сломают тест.

</details>

---

### Вопрос 3 🟢

Где по умолчанию хранятся golden-файлы?

- A) В `assets/goldens/`
- B) Относительно файла теста — по пути, указанному в `matchesGoldenFile('path/file.png')`
- C) В `build/goldens/`
- D) В директории кэша Flutter

<details>
<summary>Ответ</summary>

**B) Относительно файла теста**

```
test/
  widgets/
    profile_card_test.dart          ← тест
    goldens/
      profile_card.png              ← golden-файл (путь 'goldens/profile_card.png')
    profile_card_light.png          ← или рядом (путь 'profile_card_light.png')
```

```dart
// Относительный путь от файла теста:
await expectLater(
  find.byType(MyWidget),
  matchesGoldenFile('goldens/my_widget_default.png'),
);

// Рекомендуется организовывать по темам/платформам:
await expectLater(
  find.byType(MyWidget),
  matchesGoldenFile('goldens/light/my_widget.png'),
);
```

</details>

---

### Вопрос 4 🟡

Почему golden-тесты могут давать разные результаты на разных платформах?

- A) Это баг Flutter — нужно сообщить в GitHub
- B) Рендеринг шрифтов, субпиксельное сглаживание, алгоритмы сжатия PNG — различаются между macOS/Linux/Windows/CI
- C) Golden-тесты всегда одинаковы
- D) Только разные версии Flutter дают отличия

<details>
<summary>Ответ</summary>

**B) Различия рендеринга шрифтов и субпиксельного сглаживания**

Решения:
```dart
// 1. Использовать один и тот же runner (Linux CI = Linux разработка)

// 2. Загружать шрифты явно в тестах:
void main() {
  setUpAll(() async {
    // Загрузить шрифты из assets:
    final fontLoader = FontLoader('Roboto')
      ..addFont(rootBundle.load('assets/fonts/Roboto-Regular.ttf'));
    await fontLoader.load();
  });
}

// 3. Использовать кастомный goldenFileComparator с порогом:
class ToleranceComparator extends LocalFileComparator {
  ToleranceComparator(String testFile) : super(Uri.parse(testFile));

  @override
  Future<bool> compare(Uint8List imageBytes, Uri golden) async {
    final result = await GoldenFileComparator.compareLists(
      imageBytes,
      await getGoldenBytes(golden),
    );
    return result.passed || result.diffPercent < 0.01; // 1% допуск
  }
}
```

</details>

---

### Вопрос 5 🟡

Как протестировать виджет в нескольких темах (light/dark) с golden-тестом?

- A) Один тест на одну тему — нельзя комбинировать
- B) Написать отдельные `testWidgets` для каждой темы с разными golden-файлами
- C) `--theme=dark` флаг при запуске
- D) `ThemeData.test(isDark: true)`

<details>
<summary>Ответ</summary>

**B) Отдельные тесты с отдельными golden-файлами**

```dart
group('Button golden tests', () {
  for (final brightness in [Brightness.light, Brightness.dark]) {
    testWidgets('Button — ${brightness.name}', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          theme: brightness == Brightness.light
              ? ThemeData.light()
              : ThemeData.dark(),
          home: const Scaffold(
            body: Center(child: PrimaryButton(label: 'Купить')),
          ),
        ),
      );

      await expectLater(
        find.byType(PrimaryButton),
        matchesGoldenFile('goldens/button_${brightness.name}.png'),
      );
    });
  }
});

// golden-файлы:
// goldens/button_light.png
// goldens/button_dark.png
```

</details>

---

### Вопрос 6 🟡

Как настроить кастомный `goldenFileComparator`?

- A) Нельзя изменить компаратор
- B) Присвоить `goldenFileComparator` в `flutter_test_config.dart` или в `setUpAll()`
- C) Добавить в `pubspec.yaml` секцию `golden_comparator`
- D) Только через плагин `golden_toolkit`

<details>
<summary>Ответ</summary>

**B) `goldenFileComparator` в `flutter_test_config.dart`**

```dart
// test/flutter_test_config.dart — запускается перед всеми тестами:
import 'dart:async';
import 'package:flutter_test/flutter_test.dart';

Future<void> testExecutable(FutureOr<void> Function() testMain) async {
  // Кастомный компаратор для всех golden-тестов:
  goldenFileComparator = MyCustomComparator(
    '${Directory.current.path}/test',
  );
  await testMain();
}

// Или в setUp для отдельного теста:
setUp(() {
  goldenFileComparator = LocalFileComparator(
    Uri.parse('file://${Directory.current.path}/test/my_test.dart'),
  );
});
```

Популярный пакет: `alchemist` — удобное API для golden-тестов с группами сценариев.

</details>

---

### Вопрос 7 🟡

Что такое `alchemist` пакет и какие преимущества он даёт?

- A) Пакет для шифрования данных в тестах
- B) Flutter пакет для golden-тестов с удобным API: `GoldenTestGroup`, `GoldenTestScenario`, автоматические CI/local пресеты
- C) Замена `mocktail` для mocking
- D) Пакет для скриншотов App Store

<details>
<summary>Ответ</summary>

**B) Удобное API для golden-тестов**

```dart
// pubspec.yaml: alchemist: ^0.7.0
import 'package:alchemist/alchemist.dart';

void main() {
  goldenTest(
    'PrimaryButton сценарии',
    fileName: 'primary_button',
    builder: () => GoldenTestGroup(
      scenarioConstraints: const BoxConstraints(maxWidth: 200),
      children: [
        GoldenTestScenario(
          name: 'по умолчанию',
          child: const PrimaryButton(label: 'Нажми'),
        ),
        GoldenTestScenario(
          name: 'отключена',
          child: const PrimaryButton(label: 'Нажми', enabled: false),
        ),
        GoldenTestScenario(
          name: 'загрузка',
          child: const PrimaryButton(label: 'Нажми', isLoading: true),
        ),
      ],
    ),
  );
}
// Создаёт один golden-файл со всеми сценариями рядом
```

</details>

---

### Вопрос 8 🔴

Как интегрировать golden-тесты в CI/CD пайплайн?

- A) Пропускать golden-тесты в CI из-за различий платформ
- B) Зафиксировать Linux CI-окружение; хранить golden-файлы в Git; падение CI при различиях; PR review показывает diff
- C) Запускать только на macOS runner
- D) Использовать `--skip-goldens` флаг

<details>
<summary>Ответ</summary>

**B) Linux CI + golden-файлы в Git + diff в PR**

```yaml
# GitHub Actions:
- name: Run golden tests
  run: flutter test test/ --tags golden

# При провале — загрузить артефакт с различиями:
- name: Upload golden failures
  if: failure()
  uses: actions/upload-artifact@v4
  with:
    name: golden-failures
    path: test/**/failures/

# Скрипт для обновления golden-файлов (ручное действие):
# flutter test --update-goldens --tags golden
# git add test/**/*.png
# git commit -m "Update golden files"

# .gitattributes для корректного хранения PNG:
# *.png binary
# *.png diff=exif
```

В directories `failures/` Flutter сохраняет изображения: `_masterImage`, `_testImage`, `_isolatedDiff`.

</details>

---

### Вопрос 9 🔴

Как тестировать виджет с сетевыми изображениями в golden-тестах?

- A) Реальные URL — golden-тест сам загрузит
- B) Подменить `Image.network` через `imageBuilder` параметр или использовать `NetworkImageMock` / мокировать `HttpClient`
- C) Заменить все `Image.network` на `Image.asset` перед тестом
- D) Сетевые изображения нельзя тестировать golden-тестами

<details>
<summary>Ответ</summary>

**B) Мокировать HTTP-клиент или подменять image builder**

```dart
// Вариант 1: mocktail_image_network пакет:
import 'package:mocktail_image_network/mocktail_image_network.dart';

testWidgets('UserAvatar golden test', (tester) async {
  await mockNetworkImages(() async {
    // Внутри — Image.network будет заменён на placeholder
    await tester.pumpWidget(
      const MaterialApp(home: UserAvatar(imageUrl: 'https://...')),
    );
    await expectLater(
      find.byType(UserAvatar),
      matchesGoldenFile('goldens/user_avatar.png'),
    );
  });
});

// Вариант 2: мокировать HttpClient через HttpOverrides:
class MockHttpOverrides extends HttpOverrides {
  @override
  HttpClient createHttpClient(SecurityContext? context) {
    return MockHttpClient(); // возвращает 1x1 прозрачный PNG
  }
}

setUp(() => HttpOverrides.global = MockHttpOverrides());
```

</details>

---

### Вопрос 10 🔴

Как организовать golden-тесты в большом проекте для максимальной поддерживаемости?

- A) Один большой golden-файл на весь проект
- B) Структурировать по feature/компоненту; использовать `flutter_test_config.dart` глобально; теги для golden-тестов; разделить CI-задачи от обычных тестов
- C) Хранить golden-файлы вне репозитория
- D) Только для критичных UI-элементов, остальное игнорировать

<details>
<summary>Ответ</summary>

**B) Структурирование + теги + отдельная CI-задача**

```
test/
  flutter_test_config.dart         ← глобальный компаратор
  unit/
    ...
  widgets/
    buttons/
      primary_button_test.dart
      goldens/
        primary_button_light.png
        primary_button_dark.png
    cards/
      product_card_test.dart
      goldens/
        product_card_default.png
        product_card_loading.png
    screens/
      home_screen_test.dart
      goldens/
        home_screen_empty.png
        home_screen_loaded.png
```

```dart
// Тегировать golden-тесты:
testWidgets('HomeScreen golden', (tester) async { ... },
  tags: ['golden'],
);

// В CI — отдельная задача только для golden:
// flutter test test/ --tags golden

// В pre-commit — быстрые unit/widget без golden:
// flutter test test/ --exclude-tags golden
```

</details>
