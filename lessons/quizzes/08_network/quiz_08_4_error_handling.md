# Квиз: Обработка ошибок сети

**Тема:** 08.4 — Error Handling  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какие типичные ошибки возникают при HTTP запросах?

- A) Только сетевые ошибки (нет соединения)
- B) Нет соединения, таймаут, 4xx (ошибки клиента), 5xx (ошибки сервера), ошибки парсинга JSON
- C) Только 404 и 500
- D) HTTP запросы не выбрасывают исключения

<details>
<summary>Ответ</summary>

**B) Нет соединения, таймаут, 4xx, 5xx, ошибки парсинга**

Классификация:

- **Network**: `SocketException`, нет интернета
- **Timeout**: `TimeoutException`, медленное соединение
- **HTTP 4xx**: 400 (Bad Request), 401 (Unauthorized), 403 (Forbidden), 404 (Not Found),422 (Validation)
- **HTTP 5xx**: 500, 503 — ошибки сервера
- **Parse**: невалидный JSON, неожиданный формат

</details>

---

### Вопрос 2 🟢

Как проверить наличие интернета во Flutter?

- A) `Internet.isConnected()`
- B) Пакет `connectivity_plus`: `Connectivity().checkConnectivity()` → `ConnectivityResult`
- C) `Platform.hasInternet`
- D) `NetworkInfo.isReachable()`

<details>
<summary>Ответ</summary>

**B) Пакет `connectivity_plus`**

```dart
final connectivity = Connectivity();
final result = await connectivity.checkConnectivity();
if (result == ConnectivityResult.none) {
  showOfflineMessage();
}
// Подписка на изменения:
connectivity.onConnectivityChanged.listen((result) { ... });
```

Важно: `connectivity_plus` определяет тип соединения (wifi/mobile/none), но не гарантирует реальный доступ к интернету.

</details>

---

### Вопрос 3 🟢

Как отобразить пользователю ошибку сети?

- A) Не показывать — обработать тихо
- B) `SnackBar`, `AlertDialog`, или error state в UI с возможностью повтора
- C) `print(error)` для отладки
- D) Автоматически — Flutter показывает ошибку

<details>
<summary>Ответ</summary>

**B) SnackBar, AlertDialog или error state в UI**

Best practices:

- Кратко и понятно пользователю (не техническое сообщение)
- Кнопка "Повторить" для сетевых ошибок
- Различать: "нет интернета" vs "ошибка сервера" vs "ошибка авторизации"
- Не блокировать весь UI при некритичных ошибках

</details>

---

### Вопрос 4 🟡

Как реализовать `Result` тип для обработки ошибок без исключений?

- A) `return null` при ошибке
- B) Создать sealed class `Result<T>` с `Success<T>` и `Failure` — убирает throws и делает ошибки explicit
- C) Возвращать `Either` из какого-либо пакета
- D) B и C оба корректных подхода

<details>
<summary>Ответ</summary>

**D) B и C оба корректны**

```dart
// Вариант sealed class:
sealed class Result<T> {}
class Success<T> extends Result<T> { final T data; ... }
class Failure<T> extends Result<T> { final String message; ... }

// Использование:
Future<Result<User>> getUser(int id) async {
  try {
    final user = await api.getUser(id);
    return Success(user);
  } catch (e) {
    return Failure(e.toString());
  }
}
```

Пакет `dartz` предоставляет `Either<Left, Right>`.

</details>

---

### Вопрос 5 🟡

Что такое `retry` логика и как её реализовать?

- A) Повторить запрос один раз при любой ошибке
- B) Автоматически повторять запрос при временных ошибках (5xx, таймаут) с экспоненциальной задержкой и максимальным числом попыток
- C) Retry встроен в `Dio`
- D) Retry нужен только для WebSocket

<details>
<summary>Ответ</summary>

**B) Повторять временные ошибки с экспоненциальной задержкой**

```dart
Future<T> retryRequest<T>(Future<T> Function() request, {int maxAttempts = 3}) async {
  int attempt = 0;
  while (true) {
    try {
      return await request();
    } catch (e) {
      if (++attempt >= maxAttempts || !isRetryable(e)) rethrow;
      await Future.delayed(Duration(seconds: pow(2, attempt).toInt()));
    }
  }
}
```

Dio пакет `dio_smart_retry` автоматизирует retry.

</details>

---

### Вопрос 6 🟡

Как реализовать offline-first архитектуру?

- A) Отключить сетевые запросы
- B) Показывать кэшированные данные → фоново синхронизировать с сервером → обновлять UI при получении новых данных
- C) Загружать данные только при наличии WiFi
- D) Хранить все данные локально — никаких запросов

<details>
<summary>Ответ</summary>

**B) Кэш → фоновая синхронизация → обновление UI**

Паттерн:

1. Запрос → показать локальные данные из БД (Hive/Drift/Isar)
2. Параллельно — HTTP запрос
3. При успехе — обновить БД и UI
4. При ошибке — остаться на кэшированных данных

`Riverpod` `AsyncValue` + `keepPreviousData: true` элегантно реализует это.

</details>

---

### Вопрос 7 🟡

Как правильно логировать ошибки сети в production?

- A) `print(error)` — достаточно
- B) Firebase Crashlytics, Sentry или собственный error reporting с stack trace, device info, user context
- C) Записывать в `SharedPreferences`
- D) Логирование не нужно в production

<details>
<summary>Ответ</summary>

**B) Firebase Crashlytics, Sentry или собственный error reporting**

```dart
// Глобальный обработчик:
FlutterError.onError = (details) {
  FirebaseCrashlytics.instance.recordFlutterFatalError(details);
};
PlatformDispatcher.instance.onError = (error, stack) {
  FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
  return true;
};

// Для сетевых ошибок в Dio Interceptor:
onError: (e, handler) {
  Crashlytics.instance.recordError(e, e.stackTrace, reason: 'Network error');
  return handler.next(e);
}
```

</details>

---

### Вопрос 8 🔴

Что такое Circuit Breaker паттерн и зачем он нужен?

- A) Паттерн для отключения сети при переполнении буфера
- B) Паттерн который "открывает цепь" (прекращает запросы) при превышении порога ошибок и автоматически закрывает после timeout — защищает перегруженный сервер
- C) Аналог retry с ограничением
- D) Метод для принудительного закрытия HTTP соединения

<details>
<summary>Ответ</summary>

**B) Открывает/закрывает "цепь" для защиты сервера**

Состояния:

- **Closed** (normal): запросы проходят
- **Open** (failing): запросы блокируются сразу (fail fast)
- **Half-Open** (recovery): пробный запрос для проверки

После открытия — ждёт timeout (напр. 60 сек) → Half-Open → пробный запрос → Closed/Open. Пакет `circuit_breaker` реализует этот паттерн для Dart.

</details>

---

### Вопрос 9 🔴

Как тестировать сценарии ошибок сети (таймаут, 500 ошибки)?

- A) Нельзя тестировать без реального сервера с ошибками
- B) `MockClient`/`DioAdapter` с возвратом ошибочных ответов или исключений
- C) Отключить интернет в тестовой среде
- D) `flutter test --simulate-error 500`

<details>
<summary>Ответ</summary>

**B) Mock с возвратом ошибок**

```dart
// Для http package:
final client = MockClient((req) async => Response('', 500));

// Для Dio:
final adapter = DioAdapter(dio: dio);
adapter.onGet('/users', (s) => s.throws(
  DioException(
    requestOptions: RequestOptions(path: '/users'),
    type: DioExceptionType.connectionTimeout,
  ),
));

// Тест:
expect(repository.getUsers(), throwsA(isA<NetworkException>()));
```

</details>

---

### Вопрос 10 🔴

Как реализовать оптимистичный UI (immediate update + rollback on error)?

- A) Flutter не поддерживает оптимистичный UI
- B) Немедленно обновить local state → отправить запрос → при ошибке откатить state к предыдущему значению + показать ошибку
- C) Только через Redux middleware
- D) Через WebSocket с подтверждением

<details>
<summary>Ответ</summary>

**B) Немедленный update → rollback при ошибке**

```dart
Future<void> toggleLike(Post post) async {
  // Немедленно обновить UI:
  final previous = _posts[post.id]!;
  _updatePost(post.copyWith(isLiked: !post.isLiked));

  try {
    await api.toggleLike(post.id);
  } catch (e) {
    // Откатить:
    _updatePost(previous);
    showErrorSnackBar('Не удалось поставить лайк');
  }
}
```

</details>
