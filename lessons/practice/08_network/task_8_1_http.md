# 8.1: HTTP — базовые запросы к API рецептов

> Project: FitMenu | Глава 8 — Networking

### 8.1: Получение рецептов через пакет http

🎯 **Цель шага:** Подключить пакет `http`, реализовать `RecipeApiService` с методами GET и обработать типичные сетевые ошибки — создание базового слоя работы с API.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  http: ^1.2.0
```

Реализуй `lib/core/network/recipe_api_service.dart`.

**RecipeApiService:**
```dart
class RecipeApiService {
  RecipeApiService({http.Client? client})
      : _client = client ?? http.Client();

  final http.Client _client;
  static const _baseUrl = 'https://api.spoonacular.com';
  static const _apiKey = 'YOUR_API_KEY';

  Future<List<Map<String,dynamic>>> searchRecipes(String query, {int number = 10}) async { ... }
  Future<Map<String,dynamic>> getRecipeById(int id) async { ... }
  void dispose() => _client.close();
}
```

**searchRecipes:**
- Метод: GET `/recipes/complexSearch?query=...&number=...&apiKey=...`
- Парсинг: `jsonDecode(utf8.decode(response.bodyBytes))['results']`
- Обработка статусов: 200, 401, 429, 5xx

**ApiException:**
```dart
class ApiException implements Exception {
  const ApiException(this.message, {required this.statusCode});
  final String message;
  final int statusCode;
}
```

---

✅ **Критерии приёмки:**
- [ ] `http.Client` инжектируется через конструктор (для тестируемости)
- [ ] Все HTTP-ошибки выбрасывают `ApiException` с кодом
- [ ] `utf8.decode(response.bodyBytes)` используется для декодирования (не `response.body`)
- [ ] `_client.close()` вызывается при уничтожении сервиса
- [ ] `Uri.https()` используется вместо конкатенации строк
- [ ] Таймаут запроса настроен через `.timeout(Duration(seconds: 10))`

---

💡 **Подсказка:** `Uri.https(authority, path, queryParameters)` безопасно строит URL без ручного кодирования параметров. `response.bodyBytes` + `utf8.decode()` решает проблемы с кириллицей. `http.Client` как поле класса позволяет легко подменить в тестах через mock.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/network/recipe_api_service.dart

import 'dart:convert';
import 'package:http/http.dart' as http;

class ApiException implements Exception {
  const ApiException(this.message, {required this.statusCode});
  final String message;
  final int statusCode;

  @override
  String toString() => 'ApiException($statusCode): $message';
}

class RecipeApiService {
  RecipeApiService({http.Client? client}) : _client = client ?? http.Client();

  final http.Client _client;
  static const _authority = 'api.spoonacular.com';
  static const _apiKey = 'YOUR_API_KEY';

  Future<List<Map<String, dynamic>>> searchRecipes(
    String query, {
    int number = 10,
  }) async {
    final uri = Uri.https(_authority, '/recipes/complexSearch', {
      'query': query,
      'number': number.toString(),
      'apiKey': _apiKey,
    });

    final response = await _client
        .get(uri, headers: {'Accept': 'application/json'})
        .timeout(const Duration(seconds: 10));

    return _handleResponse(response, (json) {
      final results = json['results'] as List<dynamic>;
      return results.cast<Map<String, dynamic>>();
    });
  }

  Future<Map<String, dynamic>> getRecipeById(int id) async {
    final uri = Uri.https(_authority, '/recipes/$id/information', {
      'apiKey': _apiKey,
      'includeNutrition': 'true',
    });
    final response = await _client.get(uri).timeout(const Duration(seconds: 10));
    return _handleResponse(response, (json) => json);
  }

  T _handleResponse<T>(
    http.Response response,
    T Function(Map<String, dynamic>) parser,
  ) {
    final body = utf8.decode(response.bodyBytes);
    final json = jsonDecode(body) as Map<String, dynamic>;

    switch (response.statusCode) {
      case 200:
        return parser(json);
      case 401:
        throw const ApiException('Неверный API ключ', statusCode: 401);
      case 404:
        throw const ApiException('Ресурс не найден', statusCode: 404);
      case 429:
        throw const ApiException('Превышен лимит запросов', statusCode: 429);
      default:
        throw ApiException(
          'Ошибка сервера: ${json['message'] ?? response.statusCode}',
          statusCode: response.statusCode,
        );
    }
  }

  void dispose() => _client.close();
}
```

</details>
