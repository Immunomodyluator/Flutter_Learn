# Квиз: JSON и сериализация

**Тема:** 08.2 — JSON Serialization  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как вручную декодировать JSON объект в Dart?

- A) `JSON.parse(string)`
- B) `jsonDecode(string) as Map<String, dynamic>` из `dart:convert`
- C) `JsonDecoder.decode(string)`
- D) `fromJson(string)`

<details>
<summary>Ответ</summary>

**B) `jsonDecode(string) as Map<String, dynamic>`**

```dart
import 'dart:convert';

final String json = '{"name": "Alice", "age": 30}';
final Map<String, dynamic> map = jsonDecode(json);
print(map['name']); // Alice
```

`jsonDecode` возвращает `dynamic` — может быть Map, List, String, int, bool, null.

</details>

---

### Вопрос 2 🟢

Как добавить метод `fromJson` в Dart класс?

- A) Автоматически генерируется Flutter
- B) Вручную создать `factory User.fromJson(Map<String, dynamic> json) => User(name: json['name'])`
- C) `@fromJson` аннотация
- D) Через `implements Serializable`

<details>
<summary>Ответ</summary>

**B) Вручную создать `factory User.fromJson`**

```dart
class User {
  final String name;
  final int age;

  User({required this.name, required this.age});

  factory User.fromJson(Map<String, dynamic> json) => User(
    name: json['name'] as String,
    age: json['age'] as int,
  );

  Map<String, dynamic> toJson() => {'name': name, 'age': age};
}
```

</details>

---

### Вопрос 3 🟢

Что такое `json_serializable` пакет?

- A) Пакет для HTTP запросов
- B) Генератор кода — автоматически создаёт `fromJson`/`toJson` методы на основе аннотаций
- C) Встроенный Flutter инструмент для JSON
- D) Пакет для валидации JSON схем

<details>
<summary>Ответ</summary>

**B) Генератор кода для `fromJson`/`toJson`**

```dart
import 'package:json_annotation/json_annotation.dart';
part 'user.g.dart';

@JsonSerializable()
class User {
  final String name;
  final int age;
  User({required this.name, required this.age});

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
  Map<String, dynamic> toJson() => _$UserToJson(this);
}
```

Запуск: `flutter pub run build_runner build`.

</details>

---

### Вопрос 4 🟡

Как обработать nullable поля при JSON десериализации?

- A) Пометить поле как `late`
- B) Использовать `?` тип + `json['field'] as String?` или `@JsonKey(defaultValue: '')` аннотацию
- C) `JsonNull.orDefault(json, 'field', '')`
- D) Обернуть в `try/catch`

<details>
<summary>Ответ</summary>

**B) `?` тип или `@JsonKey(defaultValue: ...)`**

```dart
@JsonSerializable()
class User {
  final String name;
  @JsonKey(defaultValue: '')
  final String bio; // если null в JSON → ''
  final String? phone; // может быть null

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
```

`@JsonKey(name: 'user_name')` — маппинг snake_case ↔ camelCase.

</details>

---

### Вопрос 5 🟡

Как десериализовать список объектов из JSON?

- A) `List.fromJson(jsonList)`
- B) `(jsonDecode(response.body) as List).map((e) => User.fromJson(e)).toList()`
- C) `jsonDecode(response.body).cast<User>()`
- D) `json_serializable` автоматически обрабатывает List

<details>
<summary>Ответ</summary>

**B) `(jsonDecode(body) as List).map((e) => User.fromJson(e)).toList()`**

```dart
final List<dynamic> jsonList = jsonDecode(response.body);
final users = jsonList.map((j) => User.fromJson(j as Map<String, dynamic>)).toList();
```

C `json_serializable` для класса-контейнера:

```dart
@JsonSerializable()
class UsersResponse {
  final List<User> users; // автоматически
}
```

</details>

---

### Вопрос 6 🟡

Что такое `Freezed` пакет и чем он лучше `json_serializable`?

- A) Freezed — только для BLoC состояний
- B) Freezed генерирует: `copyWith`, `==`/`hashCode`, `toString`, sealed classes, `fromJson`/`toJson` — полноценные иммутабельные value objects
- C) Freezed — устаревший `json_serializable`
- D) Freezed работает только с Dart, не Flutter

<details>
<summary>Ответ</summary>

**B) Freezed = иммутабельные value objects + JSON**

```dart
@freezed
class User with _$User {
  const factory User({
    required String name,
    required int age,
    String? phone,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
// Автом.: user.copyWith(name: 'Bob'), user == user2, sealed union types
```

</details>

---

### Вопрос 7 🟡

Как маппировать snake_case JSON → camelCase Dart поля?

- A) Dart автоматически конвертирует snake_case
- B) `@JsonKey(name: 'first_name')` на поле или `@JsonSerializable(fieldRename: FieldRename.snake)` на класс
- C) `SnakeCaseDecoder` утилита
- D) Переименовать поля в Dart в snake_case

<details>
<summary>Ответ</summary>

**B) `@JsonKey(name: 'first_name')` или `@JsonSerializable(fieldRename: FieldRename.snake)`**

```dart
@JsonSerializable(fieldRename: FieldRename.snake) // все поля авто
class User {
  final String firstName; // ← 'first_name' в JSON

  @JsonKey(name: 'user_id') // явное переопределение
  final int id;
}
```

</details>

---

### Вопрос 8 🔴

Как обработать полиморфизм в JSON (разные типы объектов в одном массиве)?

- A) `json_serializable` не поддерживает полиморфизм
- B) Ручная логика: читать поле `type` и диспетчеризировать создание через `switch`/factory
- C) `@JsonDiscriminator('type')` аннотация
- D) Только через `Freezed` sealed classes

<details>
<summary>Ответ</summary>

**B) Ручная диспетчеризация по полю `type`**

```dart
abstract class Shape { }

factory Shape.fromJson(Map<String, dynamic> json) {
  switch (json['type'] as String) {
    case 'circle': return Circle.fromJson(json);
    case 'rect': return Rectangle.fromJson(json);
    default: throw UnimplementedError('Unknown type: ${json['type']}');
  }
}
```

`Freezed` sealed classes элегантнее для sealed иерархий.

</details>

---

### Вопрос 9 🔴

Как работает `@JsonConverter` в `json_serializable`?

- A) Конвертирует JSON в другой формат
- B) Позволяет задать кастомную логику сериализации/десериализации для конкретного типа
- C) Alias для `@JsonKey(toJson: ..., fromJson: ...)`
- D) B и C оба правильны

<details>
<summary>Ответ</summary>

**D) B и C описывают одно и то же**

```dart
class DateTimeConverter implements JsonConverter<DateTime, String> {
  const DateTimeConverter();
  @override
  DateTime fromJson(String json) => DateTime.parse(json);
  @override
  String toJson(DateTime object) => object.toIso8601String();
}

@JsonSerializable()
class Event {
  @DateTimeConverter()
  final DateTime createdAt;
}
```

</details>

---

### Вопрос 10 🔴

Как обеспечить type-safety при работе с API у которого нет Dart клиента?

- A) Использовать `dynamic` везде
- B) Использовать OpenAPI + `openapi_generator` для генерации типизированного клиента; или `retrofit` + `dio` + `json_serializable` с вручную описанными моделями
- C) Только ручное написание `fromJson`/`toJson`
- D) `GraphQL` единственный type-safe способ

<details>
<summary>Ответ</summary>

**B) OpenAPI generator или retrofit + json_serializable**

`retrofit` + аннотации:

```dart
@RestApi(baseUrl: 'https://api.example.com')
abstract class ApiClient {
  factory ApiClient(Dio dio) = _ApiClient;

  @GET('/users/{id}')
  Future<User> getUser(@Path('id') int id);

  @POST('/users')
  Future<User> createUser(@Body() CreateUserRequest request);
}
```

Генератор создаёт `_ApiClient` класс с Dio запросами.

</details>
