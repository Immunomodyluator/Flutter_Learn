# 08.3 JSON сериализация

## 1. Суть

JSON-сериализация — преобразование между JSON (строка/Map) и Dart-объектами. Flutter использует три подхода:

| Подход                          | Когда использовать                         |
| ------------------------------- | ------------------------------------------ |
| Ручной `fromJson`/`toJson`      | Небольшие проекты, 3–5 моделей             |
| `json_serializable`             | Средние/большие проекты, автогенерация     |
| `freezed` + `json_serializable` | Когда нужны immutable модели + union types |

---

## 2. Базовый синтаксис — ручная сериализация

```dart
import 'dart:convert';

class User {
  final int id;
  final String name;
  final String? email;
  final Address address;

  const User({
    required this.id,
    required this.name,
    this.email,
    required this.address,
  });

  // Из JSON Map
  factory User.fromJson(Map<String, dynamic> json) => User(
        id: json['id'] as int,
        name: json['name'] as String,
        email: json['email'] as String?,
        address: Address.fromJson(json['address'] as Map<String, dynamic>),
      );

  // В JSON Map
  Map<String, dynamic> toJson() => {
        'id': id,
        'name': name,
        if (email != null) 'email': email,
        'address': address.toJson(),
      };
}

// Парсинг строки → объект
final user = User.fromJson(jsonDecode(responseBody) as Map<String, dynamic>);

// Парсинг списка
final users = (jsonDecode(responseBody) as List<dynamic>)
    .map((json) => User.fromJson(json as Map<String, dynamic>))
    .toList();

// Объект → строка
final body = jsonEncode(user.toJson());
```

---

## 3. json_serializable — автогенерация

```yaml
# pubspec.yaml
dependencies:
  json_annotation: ^4.8.1

dev_dependencies:
  json_serializable: ^6.7.1
  build_runner: ^2.4.8
```

```dart
import 'package:json_annotation/json_annotation.dart';

part 'product.g.dart'; // будет сгенерирован

@JsonSerializable()
class Product {
  final int id;
  final String title;
  @JsonKey(name: 'price_usd') // кастомное имя поля
  final double priceUsd;
  @JsonKey(defaultValue: false) // значение по умолчанию
  final bool inStock;
  final Category? category;

  const Product({
    required this.id,
    required this.title,
    required this.priceUsd,
    required this.inStock,
    this.category,
  });

  factory Product.fromJson(Map<String, dynamic> json) => _$ProductFromJson(json);
  Map<String, dynamic> toJson() => _$ProductToJson(this);
}

@JsonSerializable()
class Category {
  final int id;
  final String name;

  const Category({required this.id, required this.name});

  factory Category.fromJson(Map<String, dynamic> json) => _$CategoryFromJson(json);
  Map<String, dynamic> toJson() => _$CategoryToJson(this);
}
```

```bash
# Генерация кода (запускать после изменений)
dart run build_runner build --delete-conflicting-outputs

# Режим watch (авто-генерация при изменениях)
dart run build_runner watch --delete-conflicting-outputs
```

---

## 4. freezed — immutable модели + Union types

```yaml
# pubspec.yaml
dependencies:
  freezed_annotation: ^2.4.1
  json_annotation: ^4.8.1

dev_dependencies:
  freezed: ^2.4.6
  json_serializable: ^6.7.1
  build_runner: ^2.4.8
```

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'order.freezed.dart';
part 'order.g.dart';

@freezed
class Order with _$Order {
  const factory Order({
    required String id,
    required List<OrderItem> items,
    required double totalPrice,
    @Default(OrderStatus.pending) OrderStatus status,
    DateTime? createdAt,
  }) = _Order;

  factory Order.fromJson(Map<String, dynamic> json) => _$OrderFromJson(json);
}

// Union-тип (sealed-подобное поведение)
@freezed
class OrderResult with _$OrderResult {
  const factory OrderResult.success(Order order) = OrderSuccess;
  const factory OrderResult.error(String message, int? code) = OrderError;
  const factory OrderResult.loading() = OrderLoading;

  factory OrderResult.fromJson(Map<String, dynamic> json) =>
      _$OrderResultFromJson(json);
}

// Использование union-типа
void handleResult(OrderResult result) {
  result.when(
    success: (order) => print('Заказ ${order.id}'),
    error: (message, code) => print('Ошибка $code: $message'),
    loading: () => print('Загрузка...'),
  );

  // Или map (возвращает значение)
  final title = result.map(
    success: (_) => 'Успех',
    error: (_) => 'Ошибка',
    loading: (_) => 'Загрузка',
  );
}

// copyWith — удобное обновление полей
final updated = order.copyWith(status: OrderStatus.shipped);
```

---

## 5. Вложенные объекты и списки

```dart
@JsonSerializable(explicitToJson: true) // важно для вложенных объектов!
class Cart {
  final String id;
  final List<CartItem> items; // список вложенных объектов

  const Cart({required this.id, required this.items});

  factory Cart.fromJson(Map<String, dynamic> json) => _$CartFromJson(json);
  Map<String, dynamic> toJson() => _$CartToJson(this);
}
```

---

## 6. Под капотом

- `jsonDecode` — парсит JSON строку в `dynamic` (Map, List, String, int, double, bool, null).
- `json_serializable` генерирует `.g.dart` файлы с `_$ClassFromJson` / `_$ClassToJson`.
- `freezed` генерирует `.freezed.dart` с `copyWith`, `==`, `hashCode`, `toString`.
- Оба используют `build_runner` для кодогенерации во время сборки.

---

## 7. Типичные ошибки

| Ошибка                                       | Причина                                         | Решение                                                     |
| -------------------------------------------- | ----------------------------------------------- | ----------------------------------------------------------- |
| `type 'Null' is not a subtype of type 'int'` | Поле в JSON равно null, а в модели non-nullable | Сделай поле nullable или добавь `@JsonKey(defaultValue: 0)` |
| `toJson()` не сериализует вложенные объекты  | Нет `explicitToJson: true`                      | Добавь `@JsonSerializable(explicitToJson: true)`            |
| Устаревший `.g.dart`                         | Забыл перегенерировать                          | `dart run build_runner build`                               |
| `part` директива не найдена                  | Сгенерированный файл ещё не создан              | Запусти `build_runner`                                      |
| Ключ `snake_case` в JSON, `camelCase` в Dart | Разные соглашения                               | `@JsonKey(name: 'snake_key')` или fieldRename               |

---

## 8. Рекомендации

1. **Начинай с ручного `fromJson`** на маленьких проектах — меньше накладных расходов.
2. **Переходи на `json_serializable`** как только моделей становится больше 5.
3. **Используй `freezed`** для state-объектов и моделей, которые часто копируются через `copyWith`.
4. **`explicitToJson: true`** всегда при вложенных объектах — иначе они сериализуются как `Instance of ...`.
5. **Добавь скрипт** `"build": "dart run build_runner build --delete-conflicting-outputs"` в `Makefile` или `package.json`.
6. **Проверяй nullable поля** — API может вернуть null там, где ты не ожидаешь.
