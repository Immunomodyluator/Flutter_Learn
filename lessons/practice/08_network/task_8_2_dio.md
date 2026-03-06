# 8.2: Dio — продвинутый HTTP-клиент с интерцепторами

> Project: FitMenu | Глава 8 — Networking

### 8.2: DioClient с интерцептором авторизации и логированием

🎯 **Цель шага:** Заменить `http` на `Dio` — настроить базовый URL, таймауты, интерцептор добавления Bearer-токена и интерцептор логирования запросов.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  dio: ^5.4.0
```

Реализуй `lib/core/network/dio_client.dart`.

**Конфигурация:**
```dart
class DioClient {
  DioClient({String? baseUrl, Dio? dio}) {
    _dio = dio ?? Dio(BaseOptions(
      baseUrl: baseUrl ?? 'https://api.spoonacular.com',
      connectTimeout: const Duration(seconds: 10),
      receiveTimeout: const Duration(seconds: 15),
      headers: {'Content-Type': 'application/json'},
    ));
    _addInterceptors();
  }
  late final Dio _dio;
}
```

**Интерцептор авторизации:**
```dart
class AuthInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    final token = AuthInterceptor._token;
    if (token != null) options.headers['Authorization'] = 'Bearer $token';
    handler.next(options);
  }
}
```

**LogInterceptor** добавлять только в `kDebugMode`.

**Обёртка DioException → ApiException:**
```dart
ApiException _mapDioError(DioException e) => switch (e.type) {
  DioExceptionType.connectionTimeout => ApiException('Таймаут', statusCode: 408),
  DioExceptionType.badResponse => ApiException('Ошибка ${e.response?.statusCode}', statusCode: e.response?.statusCode ?? 0),
  _ => ApiException('Нет подключения', statusCode: 0),
};
```

---

✅ **Критерии приёмки:**
- [ ] `BaseOptions` с `connectTimeout` и `receiveTimeout`
- [ ] `AuthInterceptor` добавляет `Authorization` заголовок
- [ ] `LogInterceptor` только в `kDebugMode`
- [ ] `DioException` конвертируется в `ApiException`
- [ ] `Dio` инжектируется через конструктор

---

💡 **Подсказка:** Интерцепторы выполняются в порядке добавления — логирование ставь последним. `handler.next()` — продолжить, `handler.reject()` — прервать цепочку. `kDebugMode` из `flutter/foundation.dart`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/network/dio_client.dart

import 'package:dio/dio.dart';
import 'package:flutter/foundation.dart';

class DioClient {
  DioClient({String? baseUrl, Dio? dio}) {
    _dio = dio ??
        Dio(BaseOptions(
          baseUrl: baseUrl ?? 'https://api.spoonacular.com',
          connectTimeout: const Duration(seconds: 10),
          receiveTimeout: const Duration(seconds: 15),
          headers: {'Content-Type': 'application/json', 'Accept': 'application/json'},
        ));
    _addInterceptors();
  }

  late final Dio _dio;

  void _addInterceptors() {
    _dio.interceptors.add(AuthInterceptor());
    if (kDebugMode) {
      _dio.interceptors.add(LogInterceptor(requestBody: true, responseBody: false));
    }
  }

  Future<T> get<T>(String path, {Map<String, dynamic>? params, T Function(dynamic)? fromJson}) async {
    try {
      final res = await _dio.get<dynamic>(path, queryParameters: params);
      return fromJson != null ? fromJson(res.data) : res.data as T;
    } on DioException catch (e) {
      throw _mapDioError(e);
    }
  }

  Future<T> post<T>(String path, {dynamic data, T Function(dynamic)? fromJson}) async {
    try {
      final res = await _dio.post<dynamic>(path, data: data);
      return fromJson != null ? fromJson(res.data) : res.data as T;
    } on DioException catch (e) {
      throw _mapDioError(e);
    }
  }

  ApiException _mapDioError(DioException e) => switch (e.type) {
        DioExceptionType.connectionTimeout => const ApiException('Таймаут соединения', statusCode: 408),
        DioExceptionType.receiveTimeout    => const ApiException('Таймаут ответа', statusCode: 408),
        DioExceptionType.badResponse       => ApiException('Ошибка ${e.response?.statusCode}', statusCode: e.response?.statusCode ?? 0),
        DioExceptionType.connectionError   => const ApiException('Нет подключения', statusCode: 0),
        _                                  => ApiException(e.message ?? 'Неизвестная ошибка', statusCode: 0),
      };
}

class AuthInterceptor extends Interceptor {
  static String? _token;
  static void setToken(String t) => _token = t;
  static void clearToken() => _token = null;

  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    if (_token != null) options.headers['Authorization'] = 'Bearer $_token';
    handler.next(options);
  }

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) {
    if (err.response?.statusCode == 401) clearToken();
    handler.next(err);
  }
}
```

</details>
