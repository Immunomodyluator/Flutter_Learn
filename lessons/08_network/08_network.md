# 8. Работа с сетью

## 1. Суть концепции

Сетевые запросы во Flutter всегда асинхронны и выполняются через `Future`. Flutter предоставляет два основных инструмента: встроенный пакет `http` (минимальный) и `Dio` (расширенный, с интерцепторами, отменой, ретраями).

---

## 2. Архитектура сетевого слоя

```
UI (Widget/ViewModel)
      ↓ вызов метода
Repository (абстракция над источниками данных)
      ↓ формирование запроса
ApiService / Dio Client
      ↓ HTTP запрос
Remote Server / REST API
```

---

## 3. Базовые HTTP методы

| Метод  | Назначение        | Пример           |
| ------ | ----------------- | ---------------- |
| GET    | Получить данные   | Список товаров   |
| POST   | Создать ресурс    | Создать заказ    |
| PUT    | Заменить ресурс   | Обновить профиль |
| PATCH  | Частично обновить | Изменить аватар  |
| DELETE | Удалить ресурс    | Удалить товар    |

---

## 4. Обработка ответов

```dart
// Типичная структура ответа REST API
{
  "status": "success",
  "data": { ... },
  "message": "OK"
}

// Коды ответа
// 200 OK, 201 Created, 204 No Content
// 400 Bad Request, 401 Unauthorized, 403 Forbidden
// 404 Not Found, 422 Unprocessable Entity
// 500 Internal Server Error
```

---

## 5. Слой данных — паттерн Repository

```dart
// Интерфейс
abstract class ProductRepository {
  Future<List<Product>> getProducts({int page = 1});
  Future<Product> getProductById(String id);
  Future<Product> createProduct(CreateProductDto dto);
}

// Реализация с HTTP
class RemoteProductRepository implements ProductRepository {
  final ApiClient _client;
  RemoteProductRepository(this._client);

  @override
  Future<List<Product>> getProducts({int page = 1}) async {
    final response = await _client.get('/products', params: {'page': page});
    return (response.data as List).map(Product.fromJson).toList();
  }
  // ...
}
```

---

## 6. Практические рекомендации

1. **URL никогда в виджетах** — только в `ApiService` или `Repository`.
2. **Базовый URL в константе** или через `--dart-define`.
3. **Обрабатывай ошибки на уровне репозитория** — бросай собственные исключения.
4. **Кешируй данные** — не делай один и тот же запрос при каждом widget rebuild.
5. **Используй `Dio`** — `http` слишком базовый для продакшна.
