# 08.4 Обработка ошибок сети

## 1. Суть

Сетевые запросы могут падать по трём причинам:

1. **Нет соединения** — устройство offline
2. **Таймаут** — сервер не ответил вовремя
3. **Ошибка сервера** — API вернул 4xx/5xx

Цель — перехватить все ошибки на уровне Repository и вернуть виджету понятный результат, а не необработанное исключение.

---

## 2. Типы ошибок Dio

```dart
// DioExceptionType
DioExceptionType.connectionTimeout   // Не удалось подключиться за connectTimeout
DioExceptionType.sendTimeout         // Не успели отправить запрос за sendTimeout
DioExceptionType.receiveTimeout      // Ответ не пришёл за receiveTimeout
DioExceptionType.badResponse         // Сервер ответил (statusCode != 2xx)
DioExceptionType.cancel              // Запрос отменён через CancelToken
DioExceptionType.connectionError     // Нет интернета / SocketException
DioExceptionType.unknown             // Всё остальное

// Доступ к деталям
catch (DioException e) {
  e.type            // DioExceptionType
  e.response        // Response? (null если нет ответа от сервера)
  e.response?.statusCode  // HTTP статус
  e.response?.data        // тело ответа
  e.message         // строка описания
  e.error           // оригинальное исключение (SocketException и др.)
}
```

---

## 3. Result паттерн

Вместо выброса исключений — возвращай `Result<T>` из Repository:

```dart
// Sealed класс результата
sealed class Result<T> {
  const Result();
}

final class Success<T> extends Result<T> {
  final T data;
  const Success(this.data);
}

final class Failure<T> extends Result<T> {
  final AppException error;
  const Failure(this.error);
}

// Доменные исключения
sealed class AppException implements Exception {
  const AppException();
}

class NetworkException extends AppException {
  final String message;
  const NetworkException(this.message);
}

class ServerException extends AppException {
  final int statusCode;
  final String message;
  const ServerException(this.statusCode, this.message);
}

class UnauthorizedException extends AppException {
  const UnauthorizedException();
}

class NotFoundException extends AppException {
  const NotFoundException();
}
```

---

## 4. Реальный пример — Repository с обработкой ошибок

```dart
class UserRepository {
  final Dio _dio;

  const UserRepository(this._dio);

  Future<Result<User>> getUser(String id) async {
    try {
      final response = await _dio.get<Map<String, dynamic>>('/users/$id');
      final user = User.fromJson(response.data!);
      return Success(user);
    } on DioException catch (e) {
      return Failure(_mapDioError(e));
    } catch (e) {
      return Failure(NetworkException('Неизвестная ошибка: $e'));
    }
  }

  AppException _mapDioError(DioException e) {
    switch (e.type) {
      case DioExceptionType.connectionError:
      case DioExceptionType.connectionTimeout:
        return const NetworkException('Нет соединения с интернетом');
      case DioExceptionType.receiveTimeout:
      case DioExceptionType.sendTimeout:
        return const NetworkException('Сервер не отвечает, попробуйте позже');
      case DioExceptionType.badResponse:
        return _mapStatusCode(e.response!.statusCode!, e.response!.data);
      case DioExceptionType.cancel:
        return const NetworkException('Запрос отменён');
      default:
        return NetworkException(e.message ?? 'Неизвестная ошибка');
    }
  }

  AppException _mapStatusCode(int code, dynamic data) {
    final message = _extractMessage(data);
    return switch (code) {
      401 => const UnauthorizedException(),
      404 => const NotFoundException(),
      422 => ServerException(code, message ?? 'Данные не прошли валидацию'),
      >= 500 => ServerException(code, 'Ошибка сервера, попробуйте позже'),
      _ => ServerException(code, message ?? 'Ошибка запроса'),
    };
  }

  String? _extractMessage(dynamic data) {
    if (data is Map<String, dynamic>) {
      return data['message'] as String?;
    }
    return null;
  }
}

// --- Использование в ViewModel/Cubit ---
Future<void> loadUser(String id) async {
  emit(const UserState.loading());

  final result = await _repository.getUser(id);

  switch (result) {
    case Success(:final data):
      emit(UserState.loaded(data));
    case Failure(:final error):
      final message = switch (error) {
        NetworkException(:final message) => message,
        UnauthorizedException() => 'Необходима авторизация',
        NotFoundException() => 'Пользователь не найден',
        ServerException(:final message) => message,
        _ => 'Произошла ошибка',
      };
      emit(UserState.error(message));
  }
}
```

---

## 5. Retry — автоматический повтор запроса

```dart
class RetryInterceptor extends Interceptor {
  final Dio dio;
  final int maxRetries;
  final Duration retryDelay;

  const RetryInterceptor({
    required this.dio,
    this.maxRetries = 3,
    this.retryDelay = const Duration(seconds: 1),
  });

  @override
  void onError(DioException err, ErrorInterceptorHandler handler) async {
    final shouldRetry = err.type == DioExceptionType.connectionError ||
        err.type == DioExceptionType.connectionTimeout;

    if (!shouldRetry) {
      handler.next(err);
      return;
    }

    final retryCount = err.requestOptions.extra['retryCount'] as int? ?? 0;

    if (retryCount >= maxRetries) {
      handler.next(err);
      return;
    }

    await Future.delayed(retryDelay * (retryCount + 1)); // экспоненциальная задержка

    err.requestOptions.extra['retryCount'] = retryCount + 1;

    try {
      final response = await dio.fetch(err.requestOptions);
      handler.resolve(response);
    } catch (e) {
      handler.next(err);
    }
  }
}

// Подключение
dio.interceptors.add(
  RetryInterceptor(dio: dio, maxRetries: 3),
);
```

---

## 6. Проверка соединения перед запросом

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

class NetworkChecker {
  final Connectivity _connectivity;

  const NetworkChecker(this._connectivity);

  Future<bool> get isConnected async {
    final result = await _connectivity.checkConnectivity();
    return result != ConnectivityResult.none;
  }
}

// В Repository
Future<Result<List<Post>>> getPosts() async {
  if (!await _networkChecker.isConnected) {
    return const Failure(NetworkException('Нет подключения к интернету'));
  }
  // ... запрос
}
```

---

## 7. Под капотом

- `DioException.type` определяется на уровне `HttpClient` / `dart:io`.
- `connectionError` — обёртка над `SocketException` из `dart:io`.
- `timeout` определяется таймером, запущенным в `BaseOptions`.
- `Result` паттерн пришёл из функционального программирования (Rust, Haskell) — полная замена `try/catch` на уровне API Repository.

---

## 8. Типичные ошибки

| Ошибка                             | Причина                            | Решение                                      |
| ---------------------------------- | ---------------------------------- | -------------------------------------------- |
| Исключение долетает до виджета     | Не обработано в Repository         | Оборачивай всё в `try/catch` в слое данных   |
| `badResponse` при 404 падает с NPE | `e.response` может быть null       | Проверяй `e.response != null` перед доступом |
| Retry бесконечно повторяет         | Нет счётчика попыток               | Храни `retryCount` в `requestOptions.extra`  |
| Ошибки UI при свайпе назад         | Запросы продолжают работать        | Используй `CancelToken` + cancel в `dispose` |
| Пользователь видит DioException    | Ошибку не преобразовали в доменную | Всегда маппи в `AppException` в Repository   |

---

## 9. Рекомендации

1. **Никогда не показывай `DioException`** пользователю — всегда маппи в понятное сообщение.
2. **Result паттерн** — явный способ указать, что вызов может завершиться ошибкой.
3. **Retry только для сетевых ошибок** — не для 4xx (это логика, не инфраструктура).
4. **Логируй ошибки** через Crashlytics/Sentry, но показывай пользователю упрощённое сообщение.
5. **Проверяй offline** до запроса — лучший UX, чем ждать таймаут.
6. **Экспоненциальная задержка** при retry — не DDoS серверы при массовом восстановлении соединения.
