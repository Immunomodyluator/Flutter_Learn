# Квиз: Clean Architecture

**Тема:** 13.2 — Clean Architecture  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢
Какие слои присутствуют в Clean Architecture и в каком направлении разрешены зависимости?

- A) Presentation → Domain → Data; зависимости направлены наружу
- B) Data → Domain → Presentation; зависимости направлены внутрь (к Domain)
- C) Presentation, Domain, Data; зависимости только внутрь — внешние слои зависят от внутренних, но не наоборот
- D) Слои произвольны, главное — разделить UI и бизнес-логику

<details>
<summary>Ответ</summary>

**C) Presentation, Domain, Data; зависимости только внутрь**

Clean Architecture определяет три основных слоя:
- **Domain** — самый внутренний слой. Содержит Entities, UseCases и абстракции (интерфейсы) репозиториев. Не зависит ни от Flutter, ни от пакетов Data/Presentation.
- **Data** — реализует интерфейсы Domain. Содержит DataSources, Repository-реализации, DTO-модели и маппинг.
- **Presentation** — UI + State Management. Зависит от Domain (вызывает UseCases), но не знает о Data.

**Правило зависимостей (Dependency Rule):** стрелки зависимостей всегда направлены внутрь. `Data` зависит от `Domain`, `Presentation` зависит от `Domain`, но `Domain` не зависит ни от кого.

```
Presentation  ──▶  Domain  ◀──  Data
```

</details>

---

### Вопрос 2 🟢
Что такое Entity в Clean Architecture и чем она отличается от DTO или Model?

- A) Entity — это любой Dart-класс с полями; то же самое, что Model
- B) Entity — бизнес-объект ядра домена, без зависимостей от внешних пакетов; DTO — транспортный объект для сериализации
- C) Entity хранится в базе данных, а DTO используется только в UI
- D) Entity и DTO — синонимы; разница только в именовании

<details>
<summary>Ответ</summary>

**B) Entity — бизнес-объект ядра домена, без зависимостей от внешних пакетов; DTO — транспортный объект для сериализации**

| | Entity | DTO / Model |
|---|---|---|
| Слой | Domain | Data |
| Зависимости | Чистый Dart, никаких пакетов | `json_serializable`, `freezed`, `drift` и т.д. |
| Назначение | Бизнес-правила и инварианты | Сериализация/десериализация, ORM |

```dart
// Domain — Entity (чистый Dart)
class User {
  final String id;
  final String name;
  final Email email; // Value Object с валидацией

  const User({required this.id, required this.name, required this.email});
}

// Data — DTO (зависит от json_serializable)
@JsonSerializable()
class UserDto {
  final String id;
  final String name;
  final String email;

  const UserDto({required this.id, required this.name, required this.email});

  factory UserDto.fromJson(Map<String, dynamic> json) =>
      _$UserDtoFromJson(json);

  User toEntity() => User(
        id: id,
        name: name,
        email: Email(email), // маппинг в доменный Value Object
      );
}
```

</details>

---

### Вопрос 3 🟢
Что делает UseCase (Interactor) и какова его типичная структура?

- A) UseCase — это виджет, управляющий состоянием экрана
- B) UseCase инкапсулирует одну бизнес-операцию и оркестрирует вызовы репозиториев
- C) UseCase — синоним Repository; оба хранят данные
- D) UseCase напрямую вызывает DataSource, минуя Repository

<details>
<summary>Ответ</summary>

**B) UseCase инкапсулирует одну бизнес-операцию и оркестрирует вызовы репозиториев**

Каждый UseCase отвечает за одно действие (Single Responsibility). Он принимает репозитории через конструктор и реализует метод `call`, чтобы его можно было вызывать как функцию.

```dart
// Абстракция репозитория (Domain)
abstract class UserRepository {
  Future<User> getUserById(String id);
}

// UseCase (Domain)
class GetUserByIdUseCase {
  final UserRepository _repository;

  GetUserByIdUseCase(this._repository);

  Future<User> call(String userId) {
    return _repository.getUserById(userId);
  }
}

// Использование в Presentation (BLoC / ViewModel)
class UserCubit extends Cubit<UserState> {
  final GetUserByIdUseCase _getUserById;

  UserCubit(this._getUserById) : super(UserInitial());

  Future<void> loadUser(String id) async {
    emit(UserLoading());
    final user = await _getUserById(id); // вызов как функции
    emit(UserLoaded(user));
  }
}
```

</details>

---

### Вопрос 4 🟡
Как правильно реализовать Repository pattern с абстракцией в Domain-слое?

- A) Интерфейс и реализация репозитория размещаются в Data-слое
- B) Интерфейс репозитория объявляется в Domain, реализация — в Data; Domain знает только интерфейс
- C) Repository — это просто класс с методами `save`/`load`; слой не важен
- D) Репозиторий должен напрямую работать с `http.Client` из Domain-слоя

<details>
<summary>Ответ</summary>

**B) Интерфейс репозитория объявляется в Domain, реализация — в Data; Domain знает только интерфейс**

Это — практическое применение Dependency Inversion Principle. Domain определяет контракт, Data его исполняет.

```dart
// lib/domain/repositories/user_repository.dart (Domain)
abstract class UserRepository {
  Future<User?> getUserById(String id);
  Future<void> saveUser(User user);
  Stream<List<User>> watchUsers();
}

// lib/data/repositories/user_repository_impl.dart (Data)
class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource _remote;
  final UserLocalDataSource _local;

  UserRepositoryImpl(this._remote, this._local);

  @override
  Future<User?> getUserById(String id) async {
    try {
      final dto = await _remote.fetchUser(id);
      await _local.cacheUser(dto);
      return dto.toEntity();
    } on NetworkException {
      final cached = await _local.getUser(id);
      return cached?.toEntity();
    }
  }

  @override
  Future<void> saveUser(User user) =>
      _local.saveUser(UserDto.fromEntity(user));

  @override
  Stream<List<User>> watchUsers() =>
      _local.watchUsers().map((dtos) => dtos.map((d) => d.toEntity()).toList());
}
```

</details>

---

### Вопрос 5 🟡
Чем DataSource отличается от Repository и какие бывают виды DataSource?

- A) DataSource и Repository — одно и то же, просто разные названия
- B) DataSource — источник сырых данных (сеть, БД, кэш); Repository агрегирует несколько DataSource и применяет бизнес-логику кэширования
- C) DataSource работает только с сетью, Repository — только с локальной БД
- D) Repository вызывает DataSource напрямую из Domain-слоя

<details>
<summary>Ответ</summary>

**B) DataSource — источник сырых данных; Repository агрегирует их и управляет логикой**

```dart
// Абстракции DataSource (Data-слой)
abstract class UserRemoteDataSource {
  Future<UserDto> fetchUser(String id);
}

abstract class UserLocalDataSource {
  Future<UserDto?> getUser(String id);
  Future<void> cacheUser(UserDto dto);
}

// Remote реализация
class UserRemoteDataSourceImpl implements UserRemoteDataSource {
  final http.Client _client;
  UserRemoteDataSourceImpl(this._client);

  @override
  Future<UserDto> fetchUser(String id) async {
    final response = await _client.get(Uri.parse('/users/$id'));
    if (response.statusCode != 200) throw ServerException();
    return UserDto.fromJson(jsonDecode(response.body));
  }
}

// Local реализация (SharedPreferences / Hive / Drift)
class UserLocalDataSourceImpl implements UserLocalDataSource {
  final SharedPreferences _prefs;
  UserLocalDataSourceImpl(this._prefs);

  @override
  Future<UserDto?> getUser(String id) {
    final json = _prefs.getString('user_$id');
    return Future.value(json != null ? UserDto.fromJson(jsonDecode(json)) : null);
  }

  @override
  Future<void> cacheUser(UserDto dto) =>
      _prefs.setString('user_${dto.id}', jsonEncode(dto.toJson()));
}
```

Repository решает: откуда брать данные, как долго хранить кэш, как обрабатывать ошибки.

</details>

---

### Вопрос 6 🟡
Как Dependency Injection помогает соблюдать Clean Architecture и почему Domain не должен создавать зависимости сам?

- A) DI не нужен — достаточно глобальных синглтонов и `static` методов
- B) DI позволяет передавать реализации снаружи, Domain работает только с абстракциями — это обеспечивает тестируемость и соблюдение Dependency Rule
- C) DI — это только про get_it; без этого пакета Clean Architecture невозможна
- D) Domain должен создавать свои репозитории через `new` для полного контроля

<details>
<summary>Ответ</summary>

**B) DI позволяет передавать реализации снаружи; Domain работает только с абстракциями**

Если Domain создаёт зависимости сам (`UserRepositoryImpl()`), он начинает зависеть от Data-слоя — нарушение Dependency Rule. DI «переворачивает» управление.

```dart
// БЕЗ DI — нарушение: Domain знает о Data
class GetUserUseCase {
  // ❌ Domain зависит от конкретной реализации из Data
  final _repo = UserRepositoryImpl(UserRemoteDataSourceImpl(http.Client()));
}

// С DI — Domain знает только об абстракции
class GetUserUseCase {
  final UserRepository _repo; // ✅ только интерфейс из Domain
  GetUserUseCase(this._repo);
}

// Composition Root (main.dart или DI-модуль) собирает граф зависимостей:
void main() {
  final httpClient = http.Client();
  final remote = UserRemoteDataSourceImpl(httpClient);
  final local = UserLocalDataSourceImpl(await SharedPreferences.getInstance());
  final repo = UserRepositoryImpl(remote, local);
  final useCase = GetUserUseCase(repo);

  runApp(MyApp(getUserUseCase: useCase));
}
```

</details>

---

### Вопрос 7 🟡
Как обрабатывать ошибки в Domain-слое через паттерн Either/Result без бросания исключений?

- A) Domain должен бросать исключения — `try/catch` обрабатывается в UI
- B) Используется Either<Failure, T> или Result<T> — UseCase возвращает явный тип с двумя вариантами исхода
- C) Ошибки логируются в Domain и возвращается `null`
- D) Domain возвращает Future<bool> — true означает успех

<details>
<summary>Ответ</summary>

**B) Either<Failure, T> — явное представление успеха и ошибки в типе**

```dart
// Базовый класс ошибок (Domain)
abstract class Failure {
  final String message;
  const Failure(this.message);
}

class NetworkFailure extends Failure {
  const NetworkFailure() : super('Нет соединения с сетью');
}

class NotFoundFailure extends Failure {
  const NotFoundFailure(String id) : super('Пользователь $id не найден');
}

// Простая реализация Result (без сторонних пакетов)
sealed class Result<T> {
  const Result();
}

class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

class Failure_<T> extends Result<T> {
  final Failure failure;
  const Failure_(this.failure);
}

// UseCase возвращает Result
class GetUserUseCase {
  final UserRepository _repository;
  GetUserUseCase(this._repository);

  Future<Result<User>> call(String id) async {
    try {
      final user = await _repository.getUserById(id);
      if (user == null) return Failure_(NotFoundFailure(id));
      return Success(user);
    } on NetworkException {
      return const Failure_(NetworkFailure());
    }
  }
}

// В Presentation
Future<void> loadUser(String id) async {
  final result = await _getUserById(id);
  switch (result) {
    case Success(:final data):
      emit(UserLoaded(data));
    case Failure_(:final failure):
      emit(UserError(failure.message));
  }
}
```

</details>

---

### Вопрос 8 🔴
Как организовать маппинг между слоями (Entity ↔ DTO ↔ LocalModel) и где должна располагаться логика преобразования?

- A) Маппинг выполняется прямо в UseCase через статические методы Entity
- B) Каждый DTO/LocalModel содержит методы `toEntity()` / `fromEntity()`; маппинг принадлежит Data-слою, Domain остаётся чистым
- C) Entity должна уметь сериализоваться в JSON — маппинг не нужен
- D) Маппинг централизован в одном глобальном `Mapper`-классе в Domain

<details>
<summary>Ответ</summary>

**B) DTO/LocalModel содержат методы маппинга; маппинг принадлежит Data-слою**

Domain-слой не должен знать о сериализации. Маппинг реализуется в Data через extension-методы или методы самого DTO.

```dart
// Domain (чистый Dart, никаких зависимостей)
class Product {
  final String id;
  final String name;
  final double price;
  const Product({required this.id, required this.name, required this.price});
}

// Data — Remote DTO
@JsonSerializable()
class ProductDto {
  final String id;
  final String name;
  @JsonKey(name: 'price_usd')
  final double priceUsd;

  const ProductDto({required this.id, required this.name, required this.priceUsd});
  factory ProductDto.fromJson(Map<String, dynamic> json) =>
      _$ProductDtoFromJson(json);

  // Маппинг DTO → Entity (Data знает о Domain, но не наоборот)
  Product toEntity() => Product(id: id, name: name, price: priceUsd);
}

// Data — Local Model (например, Drift)
class ProductLocalModel {
  final String id;
  final String name;
  final double price;
  const ProductLocalModel({required this.id, required this.name, required this.price});

  Product toEntity() => Product(id: id, name: name, price: price);

  static ProductLocalModel fromEntity(Product e) =>
      ProductLocalModel(id: e.id, name: e.name, price: e.price);

  // Маппинг между слоями Data (Remote → Local)
  static ProductLocalModel fromDto(ProductDto dto) =>
      ProductLocalModel(id: dto.id, name: dto.name, price: dto.priceUsd);
}
```

**Цепочка:** `JSON → ProductDto → ProductLocalModel → Product (Entity)`  
Каждый слой отвечает только за свой шаг преобразования.

</details>

---

### Вопрос 9 🔴
Как тестировать UseCase изолированно, не поднимая реальных зависимостей?

- A) UseCase нельзя тестировать без настоящего сервера и базы данных
- B) Создаётся мок-реализация Repository (вручную или через mockito/mocktail), UseCase тестируется с ней в unit-тестах
- C) UseCase тестируется только через интеграционные тесты виджетов
- D) Для тестирования UseCase нужно поднять полный DI-граф

<details>
<summary>Ответ</summary>

**B) Мок-реализация Repository + unit-тест**

Поскольку UseCase зависит только от абстракции (интерфейса) репозитория, мок легко подставить.

```dart
// pubspec.yaml: dev_dependencies: mocktail: ^1.0.0

import 'package:mocktail/mocktail.dart';
import 'package:test/test.dart';

class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late MockUserRepository mockRepo;
  late GetUserByIdUseCase useCase;

  setUp(() {
    mockRepo = MockUserRepository();
    useCase = GetUserByIdUseCase(mockRepo);
  });

  group('GetUserByIdUseCase', () {
    const tId = 'user-123';
    final tUser = User(id: tId, name: 'Иван', email: Email('ivan@example.com'));

    test('должен вернуть User при успешном запросе', () async {
      // Arrange
      when(() => mockRepo.getUserById(tId)).thenAnswer((_) async => tUser);

      // Act
      final result = await useCase(tId);

      // Assert
      expect(result, equals(tUser));
      verify(() => mockRepo.getUserById(tId)).called(1);
      verifyNoMoreInteractions(mockRepo);
    });

    test('должен бросить исключение при ошибке репозитория', () async {
      when(() => mockRepo.getUserById(any()))
          .thenThrow(const NetworkException());

      expect(() => useCase(tId), throwsA(isA<NetworkException>()));
    });
  });
}
```

Преимущество: тест запускается за миллисекунды, полностью детерминирован, не требует сети/БД.

</details>

---

### Вопрос 10 🔴
Как избежать «архитектурной астронавтики» — ситуации, когда Clean Architecture усложняет проект без реальной пользы?

- A) Clean Architecture нужно применять всегда, иначе код станет неуправляемым
- B) Стоит оценивать сложность проекта: для простых фич достаточно упрощённой структуры; полную CA вводить итеративно по мере роста
- C) Нужно сразу создавать все слои, интерфейсы и маппинг даже для CRUD-экранов
- D) Упрощение архитектуры — это технический долг, который всегда надо избегать

<details>
<summary>Ответ</summary>

**B) Оценивать сложность и вводить слои итеративно**

Признаки избыточной архитектуры («over-engineering»):
- UseCase содержит одну строку: `return repository.getItems()`
- Создаются интерфейсы, у которых никогда не будет второй реализации
- Маппинг дублирует поля 1-в-1 без какого-либо преобразования

**Практические эвристики:**

```dart
// ❌ Избыточно для простого экрана настроек
abstract class SettingsRepository { Future<Settings> getSettings(); }
class GetSettingsUseCase {
  final SettingsRepository _repo;
  GetSettingsUseCase(this._repo);
  Future<Settings> call() => _repo.getSettings(); // одна строка
}

// ✅ Для простых случаев — Repository напрямую в ViewModel
class SettingsViewModel extends ChangeNotifier {
  final SettingsRepository _repo; // достаточно без UseCase-обёртки
  SettingsViewModel(this._repo);
  Future<void> load() async {
    _settings = await _repo.getSettings();
    notifyListeners();
  }
}
```

**Правило:** UseCase оправдан, когда он:
1. Комбинирует несколько репозиториев
2. Содержит нетривиальную бизнес-логику
3. Переиспользуется в нескольких местах Presentation

Для MVP или простых CRUD-фич — начинайте с `Repository → ViewModel`, добавляйте слои по мере необходимости.

</details>
