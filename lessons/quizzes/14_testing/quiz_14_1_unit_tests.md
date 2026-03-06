# Квиз: Unit Tests

**Тема:** 14.1 — Unit Testing  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какая функция является основным строительным блоком unit-теста в Dart?

- A) `testUnit()`
- B) `test('описание', () { ... })`
- C) `runTest()`
- D) `@Test`

<details>
<summary>Ответ</summary>

**B) `test('описание', () { ... })`**

```dart
import 'package:test/test.dart';

void main() {
  test('сложение двух чисел', () {
    final result = 2 + 3;
    expect(result, equals(5));
  });

  test('деление на ноль бросает исключение', () {
    expect(() => 1 ~/ 0, throwsA(isA<IntegerDivisionByZeroException>()));
  });
}
```

`test()` принимает описание и callback с телом теста. Группировка — через `group()`.

</details>

---

### Вопрос 2 🟢

Как работает `expect()` и какие матчеры используются чаще всего?

- A) `expect(actual, expected)` — сравнивает через `==`
- B) `expect(actual, matcher)` — `equals`, `isTrue`, `isNull`, `throwsA`, `contains`, `hasLength` и др.
- C) `assert(actual == expected)`
- D) `verify(actual).toBe(expected)`

<details>
<summary>Ответ</summary>

**B) `expect(actual, matcher)`**

```dart
expect(42, equals(42));
expect('hello', contains('ell'));
expect([1, 2, 3], hasLength(3));
expect(value, isNull);
expect(flag, isTrue);
expect(flag, isFalse);
expect(list, isEmpty);
expect(list, isNotEmpty);
expect(num, greaterThan(0));
expect(num, inInclusiveRange(1, 10));

// Исключения:
expect(() => fn(), throwsException);
expect(() => fn(), throwsA(isA<FormatException>()));
expect(() => fn(), throwsA(predicate((e) => e.message == 'Bad input')));
```

</details>

---

### Вопрос 3 🟢

Что делают `setUp()` и `tearDown()`?

- A) Запускают и останавливают Flutter приложение
- B) `setUp()` выполняется перед каждым тестом, `tearDown()` — после; используются для инициализации и очистки
- C) `setUp()` выполняется один раз перед всеми тестами в файле
- D) `tearDown()` обязателен, иначе тест упадёт

<details>
<summary>Ответ</summary>

**B) `setUp` перед каждым тестом, `tearDown` после**

```dart
group('UserRepository', () {
  late UserRepository repository;
  late MockDatabase mockDb;

  setUp(() {
    mockDb = MockDatabase();
    repository = UserRepository(mockDb);
  });

  tearDown(() {
    mockDb.close();
  });

  test('getUser возвращает пользователя', () async {
    when(() => mockDb.query('users', id: '1'))
        .thenAnswer((_) async => {'id': '1', 'name': 'Alice'});
    final user = await repository.getUser('1');
    expect(user.name, equals('Alice'));
  });
});

// Аналоги для группы в целом:
// setUpAll() и tearDownAll() — один раз для всей группы
```

</details>

---

### Вопрос 4 🟡

Как мокировать зависимости с помощью `mocktail`?

- A) `MockClass extends RealClass` без дополнительных аннотаций
- B) `class MockFoo extends Mock implements Foo {}` → `when(() => mock.method()).thenReturn(value)` → `verify(() => mock.method()).called(1)`
- C) `@Mock Foo mockFoo` + `MockitoAnnotations.init(this)`
- D) `Stub<Foo>()` из пакета `stub`

<details>
<summary>Ответ</summary>

**B) `Mock implements Foo` + `when`/`verify`**

```dart
// pubspec.yaml: mocktail: ^0.3.0
import 'package:mocktail/mocktail.dart';

class MockAuthService extends Mock implements AuthService {}

void main() {
  late AuthBloc bloc;
  late MockAuthService mockAuth;

  setUp(() {
    mockAuth = MockAuthService();
    bloc = AuthBloc(mockAuth);
  });

  test('успешный вход меняет состояние на Authenticated', () async {
    // Arrange
    when(() => mockAuth.login(any(), any()))
        .thenAnswer((_) async => User(id: '1', name: 'Bob'));

    // Act
    bloc.add(LoginEvent('bob@mail.com', 'pass123'));
    await bloc.stream.first;

    // Assert
    verify(() => mockAuth.login('bob@mail.com', 'pass123')).called(1);
    expect(bloc.state, isA<AuthAuthenticated>());
  });
}
```

</details>

---

### Вопрос 5 🟡

В чём разница между mock-объектом и fake-объектом?

- A) Это одно и то же
- B) Mock — объект с верифицируемым поведением (через `verify`); Fake — рабочая упрощённая реализация без верификации
- C) Fake использует реальную сеть; Mock — нет
- D) Fake только для UI, Mock только для логики

<details>
<summary>Ответ</summary>

**B) Mock — для верификации вызовов; Fake — рабочая упрощённая реализация**

```dart
// Mock — поведение через when/verify:
class MockProductRepository extends Mock implements ProductRepository {}

// Fake — полноценная реализация без внешних зависимостей:
class FakeProductRepository implements ProductRepository {
  final _store = <String, Product>{};

  @override
  Future<Product> getProduct(String id) async =>
      _store[id] ?? (throw NotFoundException(id));

  @override
  Future<void> saveProduct(Product p) async => _store[p.id] = p;

  @override
  Future<List<Product>> getAll() async => _store.values.toList();
}

// Fake удобен когда нужно тестировать несколько методов
// во взаимодействии, без stub-ов на каждый вызов
```

</details>

---

### Вопрос 6 🟡

Как запустить тесты и получить отчёт о покрытии кода?

- A) `flutter test --coverage` → отчёт в `coverage/lcov.info`
- B) `dart analyze --coverage`
- C) `flutter run --test-mode`
- D) Покрытие доступно только через CI/CD

<details>
<summary>Ответ</summary>

**A) `flutter test --coverage` → `coverage/lcov.info`**

```bash
# Запустить тесты с покрытием:
flutter test --coverage

# Сгенерировать HTML отчёт (нужен genhtml из lcov):
genhtml coverage/lcov.info -o coverage/html
open coverage/html/index.html

# Исключить сгенерированные файлы из покрытия (lcov.info фильтрация):
# В pubspec.yaml можно не добавлять .g.dart к coverage

# Посмотреть покрытие по файлу прямо в терминале:
flutter test --coverage && lcov --summary coverage/lcov.info
```

Минимальный рекомендуемый порог — 80% строк.  
В CI проверяют через `lcov --rc lcov_branch_coverage=1 --fail-under-lines 80`.

</details>

---

### Вопрос 7 🟡

Как тестировать асинхронный код (Future / Stream)?

- A) Синхронно вызвать async-метод, игнорируя Future
- B) Использовать `async/await` в теле теста; для Stream — `expectLater(stream, emitsInOrder([...]))` или `emits`, `emitsDone`
- C) Обернуть в `Timer(Duration.zero, ...)`
- D) Асинхронный код нельзя unit-тестировать

<details>
<summary>Ответ</summary>

**B) `async/await` для Future; `expectLater` + матчеры для Stream**

```dart
// Future:
test('загружает данные асинхронно', () async {
  when(() => mockApi.fetchUsers()).thenAnswer((_) async => [User('Alice')]);
  final users = await repository.getUsers();
  expect(users, hasLength(1));
  expect(users.first.name, 'Alice');
});

// Stream:
test('поток событий', () async {
  final controller = StreamController<int>();
  controller.add(1);
  controller.add(2);
  controller.close();

  await expectLater(
    controller.stream,
    emitsInOrder([1, 2, emitsDone]),
  );
});

// BLoC Stream:
test('Bloc эмитит состояния в правильном порядке', () {
  expectLater(
    bloc.stream,
    emitsInOrder([LoadingState(), LoadedState([])]),
  );
  bloc.add(LoadEvent());
});
```

</details>

---

### Вопрос 8 🔴

Как тестировать код, зависящий от `DateTime.now()` или случайных значений?

- A) Мокировать `DateTime` напрямую через `mocktail`
- B) Инжектировать Clock/DateTimeProvider абстракцию; использовать `clock` пакет или `FakeClock`; заменять в тестах
- C) Не тестировать такой код
- D) Использовать `@Patch` аннотацию

<details>
<summary>Ответ</summary>

**B) Инжектировать абстракцию времени**

```dart
// Абстракция:
abstract class Clock {
  DateTime now();
}

class SystemClock implements Clock {
  @override DateTime now() => DateTime.now();
}

// Fake для тестов:
class FakeClock implements Clock {
  final DateTime _fixed;
  FakeClock(this._fixed);
  @override DateTime now() => _fixed;
}

// Использование в коде:
class TokenValidator {
  final Clock _clock;
  TokenValidator(this._clock);

  bool isExpired(DateTime expiresAt) => _clock.now().isAfter(expiresAt);
}

// Тест:
test('токен просрочен', () {
  final fakeClock = FakeClock(DateTime(2025, 1, 1));
  final validator = TokenValidator(fakeClock);
  final expiry = DateTime(2024, 12, 31);
  expect(validator.isExpired(expiry), isTrue);
});

// Альтернатива — пакет clock:
// withClock(Clock.fixed(DateTime(2025, 1, 1)), () { ... });
```

</details>

---

### Вопрос 9 🔴

Как тестировать extension-методы и static-методы в Dart?

- A) `@MockExtension` для extension-методов
- B) Extension-методы тестируются напрямую как обычные функции; static-методы — через мок-зависимость или функциональную инъекцию
- C) Статические методы нельзя тестировать
- D) Требуется рефлексия через `dart:mirrors`

<details>
<summary>Ответ</summary>

**B) Extension тестируются напрямую; static — через инъекцию зависимости**

```dart
// Extension тест — напрямую:
extension StringExtensions on String {
  String toTitleCase() => split(' ')
      .map((w) => w[0].toUpperCase() + w.substring(1))
      .join(' ');
}

test('toTitleCase работает корректно', () {
  expect('hello world'.toTitleCase(), equals('Hello World'));
  expect(''.toTitleCase(), equals(''));
  expect('single'.toTitleCase(), equals('Single'));
});

// Static-метод — инъекция через функцию:
class HttpHelper {
  static Future<String> get(String url) => http.get(Uri.parse(url))
      .then((r) => r.body);
}

// Лучше — инжектировать зависимость:
class ApiClient {
  final Future<http.Response> Function(Uri) _get;
  ApiClient({required this.get}) : _get = get;

  factory ApiClient.http() => ApiClient(get: http.get);
  factory ApiClient.fake(String body) => ApiClient(
    get: (_) async => http.Response(body, 200),
  );
}
```

</details>

---

### Вопрос 10 🔴

Что такое test doubles и когда использовать каждый тип?

- A) Специальные Flutter виджеты для тестирования
- B) Dummy / Stub / Spy / Mock / Fake — пять типов подмен; каждый для своей задачи
- C) Mock и Stub — одно и то же
- D) Test doubles — только для интеграционных тестов

<details>
<summary>Ответ</summary>

**B) Пять типов test doubles**

| Тип | Назначение | Пример |
|-----|-----------|--------|
| **Dummy** | Заглушка для параметра, не используется в тесте | `null` или пустой объект |
| **Stub** | Возвращает заданные значения, без верификации | `when(() => repo.get()).thenReturn([])` |
| **Spy** | Реальный объект + запись вызовов | `spyOn(service, 'method')` |
| **Mock** | Верификация ожидаемых взаимодействий | `verify(() => mock.save(any())).called(1)` |
| **Fake** | Рабочая упрощённая реализация | `InMemoryRepository` |

```dart
// Когда использовать:
// Dummy: параметр нужен по сигнатуре, но не важен в тесте
// Stub: нужно контролировать возвращаемые значения
// Mock: нужно проверить, что метод был вызван с нужными аргументами
// Fake: тест проверяет взаимодействие нескольких методов
// Spy: нужно реальное поведение + контроль над частью методов

// Правило: предпочитайте Fake и Stub Mockam,
// когда не нужно верифицировать взаимодействие
```

</details>
