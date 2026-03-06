# 13.2 Clean Architecture

## 1. Суть

**Clean Architecture** (Чистая архитектура, Robert C. Martin) — архитектурный подход, при котором код организован по слоям с явным направлением зависимостей: внешние слои зависят от внутренних, но не наоборот.

```
┌──────────────────────────────────────────────────┐
│  Presentation (Flutter, виджеты, ViewModel)      │
│  ┌────────────────────────────────────────────┐  │
│  │  Domain (бизнес-логика, чистый Dart)        │  │
│  │  ┌──────────────────────────────────────┐  │  │
│  │  │  Data (API, DB, реализации)           │  │  │
│  │  └──────────────────────────────────────┘  │  │
│  └────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────┘
```

**Главное правило**: стрелки зависимостей направлены только внутрь. `Domain` не знает о `Data` и `Presentation`.

---

## 2. Слои и их ответственность

### Domain (ядро)

```dart
// --- entities/user.dart --- (чистая модель, только dart:core)
class User {
  final String id;
  final String name;
  final String email;
  const User({required this.id, required this.name, required this.email});
}

// --- repositories/user_repository.dart --- (абстракция)
abstract interface class UserRepository {
  Future<User> getUserById(String id);
  Future<List<User>> getAllUsers();
  Future<void> saveUser(User user);
}

// --- usecases/get_user_use_case.dart ---
class GetUserUseCase {
  final UserRepository _repository;
  const GetUserUseCase(this._repository);

  Future<User> call(String id) => _repository.getUserById(id);
}
```

### Data (реализации)

```dart
// --- models/user_dto.dart --- (DTO: сериализация/десериализация)
class UserDto {
  final String id;
  final String name;
  final String email;

  const UserDto({required this.id, required this.name, required this.email});

  factory UserDto.fromJson(Map<String, dynamic> json) => UserDto(
        id: json['id'] as String,
        name: json['name'] as String,
        email: json['email'] as String,
      );

  Map<String, dynamic> toJson() => {'id': id, 'name': name, 'email': email};

  // Маппинг DTO → Entity
  User toEntity() => User(id: id, name: name, email: email);
}

// --- datasources/user_remote_data_source.dart ---
abstract interface class UserRemoteDataSource {
  Future<UserDto> getUserById(String id);
}

class UserRemoteDataSourceImpl implements UserRemoteDataSource {
  final Dio _dio;
  const UserRemoteDataSourceImpl(this._dio);

  @override
  Future<UserDto> getUserById(String id) async {
    final response = await _dio.get('/users/$id');
    return UserDto.fromJson(response.data as Map<String, dynamic>);
  }
}

// --- repositories/user_repository_impl.dart ---
class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource _remote;
  final UserLocalDataSource _local;

  const UserRepositoryImpl({required UserRemoteDataSource remote, required UserLocalDataSource local})
      : _remote = remote, _local = local;

  @override
  Future<User> getUserById(String id) async {
    try {
      final dto = await _remote.getUserById(id);
      await _local.cacheUser(dto);       // кеш для offline
      return dto.toEntity();
    } catch (_) {
      final cached = await _local.getCachedUser(id);
      if (cached != null) return cached.toEntity();
      rethrow;
    }
  }

  @override
  Future<List<User>> getAllUsers() async {
    final dtos = await _remote.getAllUsers();
    return dtos.map((d) => d.toEntity()).toList();
  }

  @override
  Future<void> saveUser(User user) async {
    final dto = UserDto(id: user.id, name: user.name, email: user.email);
    await _remote.saveUser(dto);
  }
}
```

### Presentation (UI)

```dart
// --- viewmodels/profile_view_model.dart ---
class ProfileViewModel extends ChangeNotifier {
  final GetUserUseCase _getUserUseCase;
  ProfileViewModel(this._getUserUseCase);

  User? _user;
  bool _isLoading = false;
  String? _error;

  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> loadUser(String id) async {
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      _user = await _getUserUseCase(id); // UseCase скрывает детали реализации
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

---

## 3. Полная структура Feature-модуля

```
features/
  user/
    data/
      datasources/
        user_remote_data_source.dart
        user_local_data_source.dart
      models/
        user_dto.dart
      repositories/
        user_repository_impl.dart
    domain/
      entities/
        user.dart
      repositories/
        user_repository.dart       ← interface
      usecases/
        get_user_use_case.dart
        save_user_use_case.dart
    presentation/
      screens/
        profile_screen.dart
      viewmodels/
        profile_view_model.dart
      widgets/
        user_avatar.dart
    user_di.dart                   ← регистрация зависимостей фичи
```

---

## 4. UseCase: когда нужен, когда нет

```dart
// Простой UseCase — прокси к Repository. Прагматично убираем:
class GetUserUseCase {
  call(String id) => _repo.getUserById(id); // просто делегирует
}
// → Лучше вызывать Repository напрямую из ViewModel

// UseCase со смыслом — есть бизнес-логика:
class PlaceOrderUseCase {
  Future<Order> call(Cart cart) async {
    if (cart.items.isEmpty) throw EmptyCartException();
    final discount = await _discountRepo.getActiveDiscount();
    final total = _pricingService.calculate(cart, discount);
    return _orderRepo.create(cart, total);
  }
}
```

---

## 5. Под капотом

- `Domain` — чистый Dart, импортируй только `dart:core` и подобное.
- `Entity` — неизменяемый бизнес-объект (value object или identity object).
- `DTO` (Data Transfer Object) — объект сериализации, знает о JSON/SQL.
- `Repository` — разграничивает логику (что нужно) от реализации (как это получить).

---

## 6. Типичные ошибки

| Ошибка                       | Причина                        | Решение                                              |
| ---------------------------- | ------------------------------ | ---------------------------------------------------- |
| Flutter-импорты в Domain     | Нарушение правила зависимостей | Убрать `import 'package:flutter/...'` из Domain      |
| DTO используется в UI        | Слои перемешаны                | Маппить DTO → Entity на уровне Repository            |
| UseCase только делегирует    | Избыточная абстракция          | Вызывай Repository напрямую, если нет логики         |
| Repository знает о ViewModel | Обратная зависимость           | Repository возвращает Futures/Streams, не колбэки UI |

---

## 7. Рекомендации

1. **Domain — нулевая зависимость от Flutter**: только `dart:async`, `dart:core`.
2. **DTO ≠ Entity** — никогда не передавай `UserDto` в Domain.
3. **UseCase — только когда есть реальная бизнес-логика** (составные операции, правила).
4. **Один Repository на агрегат** — `UserRepository`, `OrderRepository`, а не один `AppRepository`.
5. **Интерфейсы** в Domain — реализации в Data; это позволяет тестировать Domain с mock-объектами.
