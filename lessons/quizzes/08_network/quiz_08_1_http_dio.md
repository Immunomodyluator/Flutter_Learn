# Квиз: HTTP и Dio

**Тема:** 08.1 — HTTP / Dio  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как выполнить GET запрос с пакетом `http`?

- A) `http.fetch('https://api.example.com/data')`
- B) `final response = await http.get(Uri.parse('https://api.example.com/data'))`
- C) `Http.get(url: 'https://api.example.com/data')`
- D) `await HttpClient().get('https://api.example.com/data')`

<details>
<summary>Ответ</summary>

**B) `await http.get(Uri.parse('...'))`**

```dart
import 'package:http/http.dart' as http;

final response = await http.get(Uri.parse('https://api.example.com/users'));
if (response.statusCode == 200) {
  final data = jsonDecode(response.body);
}
```

`http.get` принимает `Uri`, не `String`. `response.statusCode`, `response.body`, `response.headers`.

</details>

---

### Вопрос 2 🟢

Как декодировать JSON ответ?

- A) `response.toJson()`
- B) `jsonDecode(response.body)` из `dart:convert`
- C) `JsonParser.parse(response.body)`
- D) `response.json()`

<details>
<summary>Ответ</summary>

**B) `jsonDecode(response.body)` из `dart:convert`**

```dart
import 'dart:convert';

final body = jsonDecode(response.body) as Map<String, dynamic>;
final user = User.fromJson(body);
```

`jsonDecode` возвращает `dynamic` — кастует к нужному типу. Для списка: `jsonDecode(body) as List<dynamic>`.

</details>

---

### Вопрос 3 🟢

В чём преимущество `Dio` над пакетом `http`?

- A) `http` быстрее, `Dio` — устарел
- B) Dio: interceptors, FormData, upload progress, timeout из коробки, базовый URL, retry, более удобный API
- C) Dio работает только на Android
- D) Нет преимуществ — они идентичны

<details>
<summary>Ответ</summary>

**B) Dio: interceptors, FormData, upload progress, timeout, базовый URL**

`Dio` — мощный HTTP клиент:

```dart
final dio = Dio(BaseOptions(
  baseUrl: 'https://api.example.com',
  connectTimeout: Duration(seconds: 5),
  receiveTimeout: Duration(seconds: 3),
));
final response = await dio.get('/users/1');
```

</details>

---

### Вопрос 4 🟡

Как добавить заголовок `Authorization` к каждому запросу через Dio?

- A) `dio.options.headers['Authorization'] = 'Bearer token'`
- B) Добавить `Interceptor` в `dio.interceptors`:
- C) `dio.setToken('Bearer token')`
- D) A и B оба корректны

<details>
<summary>Ответ</summary>

**D) A и B оба корректны**

`dio.options.headers` — глобально для всех запросов. Interceptor даёт больше гибкости:

```dart
dio.interceptors.add(InterceptorsWrapper(
  onRequest: (options, handler) {
    options.headers['Authorization'] = 'Bearer ${storage.token}';
    return handler.next(options);
  },
));
```

Interceptor лучше когда токен обновляется динамически.

</details>

---

### Вопрос 5 🟡

Как обработать ошибки HTTP (404, 500) через Dio?

- A) Dio не выбрасывает исключения — проверять `response.statusCode`
- B) `DioException` выбрасывается для ошибок — обработать через `catch (DioException e)` и проверить `e.type`, `e.response?.statusCode`
- C) `try/catch(HttpException e)`
- D) Через callback `dio.onError`

<details>
<summary>Ответ</summary>

**B) `DioException` — обработать через `catch`**

```dart
try {
  final response = await dio.get('/users/1');
} on DioException catch (e) {
  if (e.type == DioExceptionType.connectionTimeout) {
    // таймаут
  } else if (e.response?.statusCode == 404) {
    // не найдено
  } else if (e.type == DioExceptionType.receiveTimeout) {
    // медленный сервер
  }
}
```

</details>

---

### Вопрос 6 🟡

Как отправить POST запрос с JSON телом через Dio?

- A) `dio.post('/users', data: jsonEncode(user))`
- B) `dio.post('/users', data: user.toJson())` — Dio автоматически кодирует Map в JSON
- C) `dio.post('/users', body: user.toJson())`
- D) `dio.request('/users', method: 'POST', data: user)`

<details>
<summary>Ответ</summary>

**B) `dio.post('/users', data: user.toJson())` — Dio автоматически кодирует Map**

```dart
final response = await dio.post(
  '/users',
  data: {'name': 'Alice', 'email': 'alice@example.com'},
);
// Response: response.data (уже Map, если Content-Type: application/json)
```

Dio автоматически ставит `Content-Type: application/json` и декодирует ответ.

</details>

---

### Вопрос 7 🟡

Как загрузить файл через Dio с отслеживанием прогресса?

- A) `dio.upload(file, onProgress: ...)`
- B) `dio.post('/upload', data: FormData.fromMap({'file': MultipartFile.fromFile(path)}), onSendProgress: (sent, total) => ...)`
- C) `dio.postFile(file)`
- D) `MultipartRequest` — только через пакет `http`

<details>
<summary>Ответ</summary>

**B) `FormData` + `MultipartFile` + `onSendProgress` callback**

```dart
final formData = FormData.fromMap({
  'name': 'photo.jpg',
  'file': await MultipartFile.fromFile('./photo.jpg', filename: 'photo.jpg'),
});
await dio.post(
  '/upload',
  data: formData,
  onSendProgress: (sent, total) => print('${(sent/total*100).toInt()}%'),
);
```

</details>

---

### Вопрос 8 🔴

Как реализовать автоматическое обновление токена в Dio Interceptor?

- A) Обновлять токен вручную перед каждым запросом
- B) В `onError` interceptor: при 401 → обновить токен → повторить запрос через `handler.resolve(await dio.fetch(err.requestOptions))`
- C) `dio.options.tokenRefresher = () => refreshToken()`
- D) Нельзя — нужно использовать `retrofit` пакет

<details>
<summary>Ответ</summary>

**B) В `onError`: 401 → refresh → retry через `dio.fetch(originalRequest)`**

```dart
onError: (DioException error, handler) async {
  if (error.response?.statusCode == 401) {
    try {
      await authService.refreshToken();
      final opts = error.requestOptions;
      opts.headers['Authorization'] = 'Bearer ${authService.token}';
      final response = await dio.fetch(opts); // повторный запрос
      return handler.resolve(response);
    } catch (_) {
      authService.logout();
    }
  }
  return handler.next(error);
}
```

</details>

---

### Вопрос 9 🔴

Что такое `CancelToken` в Dio и когда его использовать?

- A) Токен для аутентификации
- B) Объект для отмены HTTP запроса — передать в `dio.get(url, cancelToken: token)` и вызвать `token.cancel()`
- C) Токен обновления JWT
- D) CSRF токен

<details>
<summary>Ответ</summary>

**B) Объект для отмены запроса**

```dart
final cancelToken = CancelToken();

// Запрос:
dio.get('/search', queryParameters: {'q': query}, cancelToken: cancelToken);

// Отмена (например при dispose или новом поиске):
cancelToken.cancel('Пользователь начал новый поиск');

// Обработка:
try { ... } on DioException catch (e) {
  if (CancelToken.isCancel(e)) return; // игнорировать
}
```

Полезно для: поиска (debounce + cancel), dispose виджета, смены экрана.

</details>

---

### Вопрос 10 🔴

Как тестировать HTTP запросы во Flutter без реальной сети?

- A) Тестировать только с реальным сервером
- B) Использовать `MockClient` из `package:http/testing.dart` или `DioAdapterMock`, или `http_mock_adapter` пакет
- C) `flutter test --no-network`
- D) Через `InMemoryServer`

<details>
<summary>Ответ</summary>

**B) `MockClient` для `http`, `http_mock_adapter` или `DioMock` для Dio**

```dart
// Для package:http:
final client = MockClient((request) async {
  if (request.url.path == '/users/1') {
    return http.Response('{"name": "Alice"}', 200);
  }
  return http.Response('Not Found', 404);
});

// Для Dio:
final dioAdapter = DioAdapter(dio: dio);
dioAdapter.onGet('/users/1', (server) => server.reply(200, {'name': 'Alice'}));
```

</details>
