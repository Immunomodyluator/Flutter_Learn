# 14.1 Unit-тесты

## 1. Суть

**Unit-тест** проверяет один изолированных модуль (класс, функцию) без реальных зависимостей. Зависимости заменяются **моками** (mock) или **фейками** (fake).

---

## 2. Базовый синтаксис

```dart
// test/unit/calculator_test.dart
import 'package:flutter_test/flutter_test.dart';

int add(int a, int b) => a + b;

void main() {
  // group — группировка связанных тестов
  group('add()', () {
    test('складывает два положительных числа', () {
      expect(add(2, 3), equals(5));
    });

    test('складывает с нулём', () {
      expect(add(0, 5), equals(5));
    });

    test('складывает отрицательные числа', () {
      expect(add(-3, -2), equals(-5));
    });
  });
}
```

---

## 3. Матчеры expect

```dart
expect(result, equals(42));
expect(result, isNotNull);
expect(result, isNull);
expect(result, isTrue);
expect(result, isFalse);
expect(result, isA<User>());          // проверка типа
expect(list, hasLength(3));
expect(list, contains('item'));
expect(list, isEmpty);
expect(list, isNotEmpty);
expect(string, startsWith('Hello'));
expect(string, contains('world'));
expect(value, greaterThan(10));
expect(value, lessThanOrEqualTo(100));
expect(value, inInclusiveRange(0, 100));

// Исключения
expect(() => riskyOp(), throwsException);
expect(() => riskyOp(), throwsA(isA<ArgumentError>()));
expect(() => riskyOp(), throwsA(
  predicate<FormatException>((e) => e.message.contains('invalid')),
));

// Async
await expectLater(future, completion(equals('result')));
await expectLater(stream, emitsInOrder(['a', 'b', 'c']));
await expectLater(stream, emitsDone);
await expectLater(failingFuture, throwsA(isA<NetworkException>()));
```

---

## 4. Мокирование с mocktail

```dart
// --- Интерфейс ---
abstract interface class UserRepository {
  Future<User> getUserById(String id);
  Future<void> deleteUser(String id);
}

// --- Мок ---
class MockUserRepository extends Mock implements UserRepository {}

// --- Тест ---
void main() {
  late MockUserRepository mockRepo;
  late GetUserUseCase useCase;

  setUp(() {
    mockRepo = MockUserRepository();
    useCase = GetUserUseCase(mockRepo);
  });

  test('возвращает пользователя при успешном запросе', () async {
    // Arrange
    final user = User(id: '1', name: 'Ivan', email: 'ivan@example.com');
    when(() => mockRepo.getUserById('1')).thenAnswer((_) async => user);

    // Act
    final result = await useCase('1');

    // Assert
    expect(result, equals(user));
    verify(() => mockRepo.getUserById('1')).called(1);
  });

  test('пробрасывает исключение при ошибке репозитория', () async {
    // Arrange
    when(() => mockRepo.getUserById(any())).thenThrow(NetworkException('Нет сети'));

    // Act + Assert
    expect(() => useCase('1'), throwsA(isA<NetworkException>()));
  });
}
```

---

## 5. Тестирование ViewModel

```dart
class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late MockUserRepository mockRepo;
  late ProfileViewModel viewModel;

  setUp(() {
    mockRepo = MockUserRepository();
    viewModel = ProfileViewModel(mockRepo);
  });

  tearDown(() {
    viewModel.dispose();
  });

  group('ProfileViewModel', () {
    test('начальное состояние — загрузка не активна', () {
      expect(viewModel.isLoading, isFalse);
      expect(viewModel.user, isNull);
      expect(viewModel.error, isNull);
    });

    test('loadUser — успешно загружает пользователя', () async {
      final user = User(id: '1', name: 'Ivan', email: 'ivan@example.com');
      when(() => mockRepo.getUserById('1')).thenAnswer((_) async => user);

      // Слушаем нотификации
      final states = <bool>[];
      viewModel.addListener(() => states.add(viewModel.isLoading));

      await viewModel.loadUser('1');

      expect(viewModel.user, equals(user));
      expect(viewModel.isLoading, isFalse);
      expect(viewModel.error, isNull);
      // Были: true (начало), false (конец)
      expect(states, equals([true, false]));
    });

    test('loadUser — устанавливает error при исключении', () async {
      when(() => mockRepo.getUserById(any()))
          .thenThrow(NetworkException('Ошибка сети'));

      await viewModel.loadUser('1');

      expect(viewModel.user, isNull);
      expect(viewModel.isLoading, isFalse);
      expect(viewModel.error, isNotNull);
    });
  });
}
```

---

## 6. setUp / tearDown / setUpAll / tearDownAll

```dart
void main() {
  late Database db;

  // Запускается один раз перед всеми тестами в group
  setUpAll(() async {
    db = await openTestDatabase();
  });

  // Запускается перед каждым тестом
  setUp(() {
    db.clear(); // очищаем данные
  });

  // Запускается после каждого теста
  tearDown(() {
    // очистка если нужна
  });

  // Запускается один раз после всех тестов
  tearDownAll(() async {
    await db.close();
  });

  test('test 1', () { ... });
  test('test 2', () { ... });
}
```

---

## 7. Параметризованные тесты

```dart
void main() {
  final cases = [
    ('ivan@example.com', true),
    ('invalid-email', false),
    ('test@test', false),
    ('a@b.co', true),
  ];

  for (final (email, expected) in cases) {
    test('validateEmail("$email") == $expected', () {
      expect(Validators.email(email), equals(expected));
    });
  }
}
```

---

## 8. Тестирование стримов

```dart
test('стрим выдаёт обновления корзины', () async {
  when(() => mockRepo.watchCart())
      .thenAnswer((_) => Stream.fromIterable([
            Cart(items: []),
            Cart(items: [CartItem(productId: '1', qty: 1)]),
          ]));

  await expectLater(
    viewModel.cartStream,
    emitsInOrder([
      predicate<Cart>((c) => c.isEmpty),
      predicate<Cart>((c) => c.items.length == 1),
    ]),
  );
});
```

---

## 9. Типичные ошибки

| Ошибка                    | Причина                           | Решение                                         |
| ------------------------- | --------------------------------- | ----------------------------------------------- |
| `MissingStubError`        | Вызов не заглушённого метода мока | Добавить `when(...)` для каждого вызова         |
| Тест зависит от другого   | Общее состояние между тестами     | Инициализировать в `setUp()`                    |
| Тест тестирует реализацию | Завязан на детали, не поведение   | Тестировать input → output                      |
| `verify` всегда проходит  | Неверная проверка                 | Использовать `verifyNever(...)` или `called(N)` |

---

## 10. Рекомендации

1. **AAA-паттерн**: Arrange → Act → Assert — читаемая структура теста.
2. **Один тест = одно утверждение** (в идеале).
3. **Имена тестов** — описывают поведение: `'loadUser — возвращает пользователя при успехе'`.
4. **`mocktail` без кодогенерации** — проще `mockito` в большинстве случаев.
5. **`setUp/tearDown`** — не дублируй инициализацию в каждом тесте.
