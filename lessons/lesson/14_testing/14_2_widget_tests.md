# 14.2 Widget-тесты

## 1. Суть

**Widget-тест** проверяет UI-компонент в виртуальном Flutter-окружении (без реального устройства). Можно:

- проверить, что виджет отображается корректно,
- симулировать нажатия, ввод текста, скролл,
- проверить, что показываются нужные виджеты при разных состояниях.

---

## 2. Базовый синтаксис

```dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('кнопка показывает текст "Нажми меня"', (tester) async {
    // Рендерим виджет
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

    // Ищем текст
    expect(find.text('Нажми меня'), findsOneWidget);
    expect(find.byType(ElevatedButton), findsOneWidget);
  });
}
```

---

## 3. Finders — поиск виджетов

```dart
// По тексту
find.text('Привет')
find.textContaining('При')

// По типу виджета
find.byType(ElevatedButton)
find.byType(CircularProgressIndicator)

// По Key
find.byKey(const Key('login_button'))
find.byKey(const ValueKey('email_field'))

// По иконке
find.byIcon(Icons.search)

// По семантике (accessibility)
find.bySemanticsLabel('Войти')

// Комбинирование
find.descendant(
  of: find.byType(Card),
  matching: find.byType(Text),
)

// Количество
expect(find.byType(ListTile), findsNWidgets(3));
expect(find.text('Ошибка'), findsNothing);
expect(find.text('Загрузка'), findsOne);   // Flutter 3.16+
```

---

## 4. Взаимодействие

```dart
// Нажатие
await tester.tap(find.byType(ElevatedButton));
await tester.pump(); // перестроить после события

// Двойное нажатие / долгое нажатие
await tester.doubleTap(find.byType(ListTile));
await tester.longPress(find.byKey(const Key('item')));

// Ввод текста
await tester.enterText(find.byType(TextField), 'ivan@example.com');
await tester.pump();

// Скролл
await tester.drag(find.byType(ListView), const Offset(0, -300));
await tester.pump();

// Скролл до виджета
await tester.scrollUntilVisible(find.text('Последний элемент'), 200);

// Swipe
await tester.fling(find.byType(Dismissible), const Offset(-200, 0), 1000);
await tester.pump();
```

---

## 5. pump и pumpAndSettle

```dart
// pump() — один кадр анимации
await tester.pump();

// pump(Duration) — симулирует прошедшее время
await tester.pump(const Duration(seconds: 2));

// pumpAndSettle() — ждёт завершения всех анимаций
await tester.pumpAndSettle();

// Пример: кнопка запускает Future, потом показывает результат
await tester.tap(find.byType(ElevatedButton));
await tester.pump();                    // начало Future (isLoading: true)
expect(find.byType(CircularProgressIndicator), findsOneWidget);

await tester.pumpAndSettle();          // ждём завершения Future
expect(find.text('Успех'), findsOneWidget);
```

---

## 6. Тестирование экрана с Provider/ChangeNotifier

```dart
class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late MockUserRepository mockRepo;

  setUp(() {
    mockRepo = MockUserRepository();
  });

  testWidgets('ProfileScreen — показывает имя пользователя', (tester) async {
    final user = User(id: '1', name: 'Ivan', email: 'ivan@example.com');
    when(() => mockRepo.getUserById('1')).thenAnswer((_) async => user);

    await tester.pumpWidget(
      MaterialApp(
        home: ChangeNotifierProvider(
          create: (_) => ProfileViewModel(mockRepo)..loadUser('1'),
          child: const ProfileScreen(),
        ),
      ),
    );

    // Первый кадр — загрузка
    expect(find.byType(CircularProgressIndicator), findsOneWidget);

    // После загрузки
    await tester.pumpAndSettle();
    expect(find.text('Ivan'), findsOneWidget);
    expect(find.text('ivan@example.com'), findsOneWidget);
  });

  testWidgets('ProfileScreen — показывает ошибку при неуспехе', (tester) async {
    when(() => mockRepo.getUserById(any()))
        .thenThrow(NetworkException('Нет сети'));

    await tester.pumpWidget(
      MaterialApp(
        home: ChangeNotifierProvider(
          create: (_) => ProfileViewModel(mockRepo)..loadUser('1'),
          child: const ProfileScreen(),
        ),
      ),
    );

    await tester.pumpAndSettle();
    expect(find.textContaining('Ошибка'), findsOneWidget);
  });
}
```

---

## 7. Тестирование форм

```dart
testWidgets('LoginForm — валидация email', (tester) async {
  await tester.pumpWidget(const MaterialApp(home: LoginForm()));

  // Пробуем отправить пустую форму
  await tester.tap(find.byKey(const Key('submit_button')));
  await tester.pump();
  expect(find.text('Введите email'), findsOneWidget);

  // Вводим неверный email
  await tester.enterText(find.byKey(const Key('email_field')), 'notanemail');
  await tester.tap(find.byKey(const Key('submit_button')));
  await tester.pump();
  expect(find.text('Неверный формат email'), findsOneWidget);

  // Вводим верный
  await tester.enterText(find.byKey(const Key('email_field')), 'test@example.com');
  await tester.pump();
  expect(find.text('Неверный формат email'), findsNothing);
});
```

---

## 8. Keys — обязательны для тестируемых виджетов

```dart
// В виджете
ElevatedButton(
  key: const Key('login_button'),
  onPressed: _onLogin,
  child: const Text('Войти'),
)

TextField(
  key: const Key('email_field'),
  controller: _emailController,
)

// В тесте
await tester.tap(find.byKey(const Key('login_button')));
await tester.enterText(find.byKey(const Key('email_field')), 'test@test.com');
```

---

## 9. Под капотом

- `tester.pumpWidget` создаёт виртуальное Flutter-окружение.
- Тесты выполняются без GPU и без реальных платформенных APIs.
- `pump()` выполняет один «тик» event loop → перестраивает дерево.
- `pumpAndSettle()` вызывает `pump()` в цикле до отсутствия ожидающих анимаций/таймеров.

---

## 10. Типичные ошибки

| Ошибка                            | Причина                                    | Решение                                       |
| --------------------------------- | ------------------------------------------ | --------------------------------------------- |
| `findsNothing` когда не ожидалось | Виджет не отрендерился                     | Вызвать `pump()` или `pumpAndSettle()`        |
| Тест зависает                     | `pumpAndSettle()` при бесконечной анимации | Использовать `pump(Duration)`                 |
| `MissingWidget` для MediaQuery    | Нет MaterialApp обёртки                    | Оборачивать в `MaterialApp`                   |
| Нельзя найти виджет               | Нет нужного finder                         | Добавить `Key` или использовать `find.byType` |

---

## 11. Рекомендации

1. **Key на интерактивных элементах** — кнопки, поля ввода, элементы списка.
2. **MaterialApp-обёртка** — обязательна, иначе нет темы и директива.
3. **pump() после взаимодействий**, **pumpAndSettle() после async** операций.
4. **Тестируй конечное состояние**, не промежуточное (если не нужно).
5. **Один testWidgets = один сценарий**: не смешивай несколько потоков в одном тесте.
