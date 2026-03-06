# Квиз: Widget Tests

**Тема:** 14.2 — Widget Testing  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какая функция используется для запуска widget-теста в Flutter?

- A) `test()`
- B) `testWidgets('описание', (WidgetTester tester) async { ... })`
- C) `widgetTest()`
- D) `runWidgetTest()`

<details>
<summary>Ответ</summary>

**B) `testWidgets('описание', (WidgetTester tester) async { ... })`**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('кнопка отображает правильный текст', (tester) async {
    await tester.pumpWidget(
      const MaterialApp(
        home: Scaffold(
          body: ElevatedButton(
            onPressed: null,
            child: Text('Нажми меня'),
          ),
        ),
      ),
    );

    expect(find.text('Нажми меня'), findsOneWidget);
  });
}
```

`WidgetTester` предоставляет методы для взаимодействия с виджетами и управления временем.

</details>

---

### Вопрос 2 🟢

Как найти виджет в дереве с помощью `find`?

- A) `find.widget(Text('hello'))`
- B) `find.byType(Text)`, `find.text('hello')`, `find.byKey(Key('k'))`, `find.byIcon(Icons.add)`
- C) `tester.locate(Text)`
- D) `WidgetFinder.find(context, Text)`

<details>
<summary>Ответ</summary>

**B) Различные `find.*` методы**

```dart
// По типу виджета:
expect(find.byType(ElevatedButton), findsOneWidget);

// По тексту:
expect(find.text('Привет'), findsOneWidget);

// По Key:
expect(find.byKey(const Key('submit_button')), findsOneWidget);

// По иконке:
expect(find.byIcon(Icons.favorite), findsOneWidget);

// По семантической метке:
expect(find.bySemanticsLabel('Загрузить'), findsOneWidget);

// По предикату:
expect(
  find.byWidgetPredicate((w) => w is Text && w.data!.startsWith('Ошибка')),
  findsOneWidget,
);

// Матчеры количества:
expect(find.byType(ListTile), findsNWidgets(3));
expect(find.text('нет'), findsNothing);
```

</details>

---

### Вопрос 3 🟢

В чём разница между `pump()` и `pumpAndSettle()`?

- A) Это одно и то же
- B) `pump()` делает один кадр рендеринга; `pumpAndSettle()` повторяет `pump()` пока нет анимаций/таймеров
- C) `pumpAndSettle()` ждёт сетевых запросов
- D) `pump()` — для StatelessWidget, `pumpAndSettle()` — для StatefulWidget

<details>
<summary>Ответ</summary>

**B) `pump()` — один кадр; `pumpAndSettle()` — до завершения всех анимаций**

```dart
testWidgets('показывает индикатор загрузки и скрывает его', (tester) async {
  await tester.pumpWidget(const MyLoadingWidget());

  // pump(duration) — прокрутить время на duration:
  await tester.pump(); // один кадр
  expect(find.byType(CircularProgressIndicator), findsOneWidget);

  // pumpAndSettle — ждать завершения всех анимаций:
  await tester.pumpAndSettle();
  expect(find.byType(CircularProgressIndicator), findsNothing);
  expect(find.text('Готово'), findsOneWidget);
});

// ВНИМАНИЕ: pumpAndSettle зависает при бесконечных анимациях!
// Тогда используй pump(Duration(seconds: 1)) вручную.
```

</details>

---

### Вопрос 4 🟡

Как симулировать нажатие кнопки и ввод текста?

- A) `tester.click()` и `tester.type()`
- B) `tester.tap(finder)` + `pump()`; `tester.enterText(finder, text)` + `pump()`
- C) `tester.press()` и `tester.input()`
- D) Только через `GestureDetector` в тесте

<details>
<summary>Ответ</summary>

**B) `tester.tap()` и `tester.enterText()`**

```dart
testWidgets('форма логина работает корректно', (tester) async {
  await tester.pumpWidget(const MaterialApp(home: LoginScreen()));

  // Ввод текста:
  await tester.enterText(find.byKey(const Key('email_field')), 'test@mail.com');
  await tester.enterText(find.byKey(const Key('pass_field')), 'secret123');
  await tester.pump();

  // Нажатие кнопки:
  await tester.tap(find.byKey(const Key('login_button')));
  await tester.pump();

  // Другие жесты:
  await tester.drag(find.byType(ListView), const Offset(0, -300)); // скролл
  await tester.longPress(find.text('элемент'));
  await tester.doubleTap(find.byType(Image));

  await tester.pumpAndSettle();
  expect(find.text('Добро пожаловать'), findsOneWidget);
});
```

</details>

---

### Вопрос 5 🟡

Как тестировать виджет, зависящий от Provider / Riverpod?

- A) Widget-тесты не могут работать с провайдерами
- B) Оборачивать виджет в `ProviderScope` (Riverpod) или `ChangeNotifierProvider` с mock-зависимостью
- C) Использовать реальные зависимости в тестах
- D) Только через `InheritedWidget`

<details>
<summary>Ответ</summary>

**B) Оборачивать в провайдер с мок-зависимостью**

```dart
// Riverpod:
testWidgets('ProductList показывает товары', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        productsProvider.overrideWith(
          (ref) => AsyncValue.data([
            Product(id: '1', name: 'Товар А'),
            Product(id: '2', name: 'Товар Б'),
          ]),
        ),
      ],
      child: const MaterialApp(home: ProductListScreen()),
    ),
  );

  await tester.pumpAndSettle();
  expect(find.text('Товар А'), findsOneWidget);
  expect(find.text('Товар Б'), findsOneWidget);
});

// Provider:
testWidgets('Provider override', (tester) async {
  final mockVM = MockProductViewModel();
  when(() => mockVM.products).thenReturn([Product(id: '1', name: 'Test')]);

  await tester.pumpWidget(
    ChangeNotifierProvider<ProductViewModel>.value(
      value: mockVM,
      child: const MaterialApp(home: ProductScreen()),
    ),
  );
});
```

</details>

---

### Вопрос 6 🟡

Как проверить навигацию в widget-тесте?

- A) Навигацию нельзя тестировать в widget-тестах
- B) Использовать `NavigatorObserver` / мокировать роутер; проверять наличие нового виджета после `pumpAndSettle()`
- C) `expect(navigator.currentRoute, '/home')`
- D) Только в integration tests

<details>
<summary>Ответ</summary>

**B) `NavigatorObserver` для проверки навигации**

```dart
class MockNavigatorObserver extends Mock implements NavigatorObserver {}

testWidgets('кнопка Back переходит на главную', (tester) async {
  final mockObserver = MockNavigatorObserver();

  await tester.pumpWidget(
    MaterialApp(
      navigatorObservers: [mockObserver],
      home: const HomeScreen(),
      routes: {'/detail': (_) => const DetailScreen()},
    ),
  );

  // Нажать кнопку, запускающую навигацию:
  await tester.tap(find.text('Открыть детали'));
  await tester.pumpAndSettle();

  // Проверить, что произошёл push:
  verify(() => mockObserver.didPush(any(), any())).called(1);

  // Или проверить наличие нового виджета:
  expect(find.byType(DetailScreen), findsOneWidget);
});
```

</details>

---

### Вопрос 7 🟡

Как тестировать кастомный виджет с параметрами без MaterialApp?

- A) Всегда нужен `MaterialApp`
- B) Можно обернуть в минимальный `Directionality` или `MediaQuery`; использовать `Scaffold` только при необходимости
- C) Только через `Widget.test()` хелпер
- D) Кастомные виджеты тестировать нельзя

<details>
<summary>Ответ</summary>

**B) Минимальная обёртка — `Directionality`**

```dart
testWidgets('BadgeWidget отображает счётчик', (tester) async {
  await tester.pumpWidget(
    // Минимально необходимая обёртка для текста:
    const Directionality(
      textDirection: TextDirection.ltr,
      child: BadgeWidget(count: 5),
    ),
  );

  expect(find.text('5'), findsOneWidget);
});

// Для виджетов с Theme:
testWidgets('ColoredCard использует цвет темы', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      theme: ThemeData(primaryColor: Colors.red),
      home: const Scaffold(body: ColoredCard()),
    ),
  );

  final card = tester.widget<Container>(find.byType(Container));
  expect((card.decoration as BoxDecoration).color, Colors.red);
});
```

</details>

---

### Вопрос 8 🔴

Как тестировать виджеты с анимациями, не дожидаясь их завершения?

- A) Только `pumpAndSettle()` — нет альтернатив
- B) `tester.pump(Duration)` — прокрутить время на конкретный интервал; `tester.binding.clock` для контроля; использовать `FakeAsync`
- C) Отключить анимации флагом `--no-animations`
- D) `AnimationController.stop()` в тесте

<details>
<summary>Ответ</summary>

**B) `pump(Duration)` для прокрутки времени**

```dart
testWidgets('анимация появления работает поэтапно', (tester) async {
  await tester.pumpWidget(const MaterialApp(home: FadeInWidget()));

  // В начале — непрозрачный:
  expect(
    tester.widget<FadeTransition>(find.byType(FadeTransition)).opacity.value,
    0.0,
  );

  // Прокрутить на половину анимации (500ms из 1000ms):
  await tester.pump(const Duration(milliseconds: 500));
  final opacity = tester.widget<FadeTransition>(
    find.byType(FadeTransition),
  ).opacity.value;
  expect(opacity, greaterThan(0.0));
  expect(opacity, lessThan(1.0));

  // Завершить анимацию:
  await tester.pump(const Duration(milliseconds: 500));
  expect(
    tester.widget<FadeTransition>(find.byType(FadeTransition)).opacity.value,
    1.0,
  );
});
```

</details>

---

### Вопрос 9 🔴

Как тестировать виджет, вызывающий асинхронные операции при построении?

- A) `async` тест с `await tester.pump()` — недостаточно
- B) Управлять Future через `Completer`; использовать мок-зависимость; pump на каждом этапе
- C) Только integration tests для async виджетов
- D) `tester.runAsync(() async { ... })`

<details>
<summary>Ответ</summary>

**B) `Completer` + mocks + поэтапный `pump`; или `tester.runAsync`**

```dart
testWidgets('показывает загрузку и затем данные', (tester) async {
  final completer = Completer<List<String>>();
  final mockRepo = MockRepository();
  when(() => mockRepo.getData()).thenAnswer((_) => completer.future);

  await tester.pumpWidget(
    ProviderScope(
      overrides: [repoProvider.overrideWithValue(mockRepo)],
      child: const MaterialApp(home: DataScreen()),
    ),
  );

  // До завершения Future — загрузка:
  await tester.pump();
  expect(find.byType(CircularProgressIndicator), findsOneWidget);

  // Завершить Future:
  completer.complete(['Данные А', 'Данные Б']);
  await tester.pumpAndSettle();

  // После завершения — данные:
  expect(find.byType(CircularProgressIndicator), findsNothing);
  expect(find.text('Данные А'), findsOneWidget);
});
```

</details>

---

### Вопрос 10 🔴

Как проверить accessibility-свойства виджета в тесте?

- A) Accessibility нельзя тестировать
- B) `tester.getSemantics(finder)` → проверить `SemanticsData`; или `SemanticsHandle` через `tester.ensureSemantics()`
- C) `find.bySemanticsLabel()` только для поиска
- D) Только через TalkBack на реальном устройстве

<details>
<summary>Ответ</summary>

**B) `tester.getSemantics()` + `SemanticsData`**

```dart
testWidgets('кнопка имеет правильную семантику', (tester) async {
  final semanticsHandle = tester.ensureSemantics();

  await tester.pumpWidget(
    MaterialApp(
      home: Scaffold(
        body: Semantics(
          label: 'Добавить в избранное',
          button: true,
          child: IconButton(
            onPressed: () {},
            icon: const Icon(Icons.favorite_border),
          ),
        ),
      ),
    ),
  );

  final semantics = tester.getSemantics(find.byType(IconButton));
  expect(semantics.label, 'Добавить в избранное');
  expect(semantics.hasFlag(SemanticsFlag.isButton), isTrue);

  semanticsHandle.dispose();
});

// Проверка контраста через AccessibilityGuideline:
testWideets('виджет соответствует рекомендациям доступности', (tester) async {
  await tester.pumpWidget(const MaterialApp(home: MyWidget()));
  await expectLater(
    tester, meetsGuideline(textContrastGuideline),
  );
});
```

</details>
