# 08.1 Пакет `http` — базовые HTTP-запросы

## 1. Суть

Пакет `http` — официальная минималистичная библиотека от команды Dart для выполнения HTTP-запросов. Подходит для простых REST API без сложной логики.

```yaml
# pubspec.yaml
dependencies:
  http: ^1.2.0
```

---

## 2. Как используется во Flutter

Используется в слое данных (Repository / DataSource). Все методы возвращают `Future<Response>`.

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';
```

---

## 3. Основной синтаксис

```dart
// GET запрос
final response = await http.get(Uri.parse('https://api.example.com/users'));

// POST запрос с телом
final response = await http.post(
  Uri.parse('https://api.example.com/users'),
  headers: {'Content-Type': 'application/json'},
  body: jsonEncode({'name': 'Ivan', 'email': 'ivan@example.com'}),
);

// PUT, PATCH, DELETE
await http.put(Uri.parse(url), headers: headers, body: body);
await http.patch(Uri.parse(url), headers: headers, body: body);
await http.delete(Uri.parse(url), headers: headers);

// Объект Response
response.statusCode  // int: 200, 404, 500 ...
response.body        // String
response.bodyBytes   // Uint8List
response.headers     // Map<String, String>
```

---

## 4. Реальный пример — получение списка постов

```dart
import 'package:http/http.dart' as http;
import 'dart:convert';

class Post {
  final int id;
  final String title;
  final String body;

  const Post({required this.id, required this.title, required this.body});

  factory Post.fromJson(Map<String, dynamic> json) => Post(
        id: json['id'] as int,
        title: json['title'] as String,
        body: json['body'] as String,
      );
}

// --- Repository ---
class PostRepository {
  static const String _baseUrl = 'https://jsonplaceholder.typicode.com';

  Future<List<Post>> getPosts() async {
    final response = await http.get(Uri.parse('$_baseUrl/posts'));

    if (response.statusCode == 200) {
      final List<dynamic> data = jsonDecode(response.body) as List<dynamic>;
      return data.map((json) => Post.fromJson(json as Map<String, dynamic>)).toList();
    }
    throw HttpException('Ошибка ${response.statusCode}: ${response.body}');
  }

  Future<Post> createPost(String title, String body) async {
    final response = await http.post(
      Uri.parse('$_baseUrl/posts'),
      headers: const {'Content-Type': 'application/json; charset=UTF-8'},
      body: jsonEncode({'title': title, 'body': body, 'userId': 1}),
    );

    if (response.statusCode == 201) {
      return Post.fromJson(jsonDecode(response.body) as Map<String, dynamic>);
    }
    throw HttpException('Не удалось создать пост: ${response.statusCode}');
  }
}

// --- Widget ---
class PostListScreen extends StatefulWidget {
  const PostListScreen({super.key});

  @override
  State<PostListScreen> createState() => _PostListScreenState();
}

class _PostListScreenState extends State<PostListScreen> {
  late Future<List<Post>> _postsFuture;
  final _repository = PostRepository();

  @override
  void initState() {
    super.initState();
    _postsFuture = _repository.getPosts();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Посты')),
      body: FutureBuilder<List<Post>>(
        future: _postsFuture,
        builder: (context, snapshot) {
          if (snapshot.connectionState == ConnectionState.waiting) {
            return const Center(child: CircularProgressIndicator());
          }
          if (snapshot.hasError) {
            return Center(child: Text('Ошибка: ${snapshot.error}'));
          }
          final posts = snapshot.data!;
          return ListView.builder(
            itemCount: posts.length,
            itemBuilder: (context, index) {
              final post = posts[index];
              return ListTile(
                title: Text(post.title),
                subtitle: Text(post.body, maxLines: 2, overflow: TextOverflow.ellipsis),
              );
            },
          );
        },
      ),
    );
  }
}
```

---

## 5. Авторизованные запросы с Bearer токеном

```dart
class AuthenticatedClient {
  final http.Client _inner;
  final String _token;

  const AuthenticatedClient(this._inner, this._token);

  Future<http.Response> get(Uri url) => _inner.get(
        url,
        headers: {'Authorization': 'Bearer $_token'},
      );
}

// Использование
final client = AuthenticatedClient(http.Client(), userToken);
final response = await client.get(Uri.parse('$baseUrl/me'));
```

---

## 6. Под капотом

- `http.get(url)` — синтаксический сахар над `http.Client().get(url)`.
- По умолчанию создаётся новый `Client` на каждый вызов — неэффективно при многих запросах.
- Для продакшна создавай `http.Client()` один раз и передавай через DI.
- Не поддерживает интерцепторы, ретраи, CancelToken — для этого нужен `Dio`.

---

## 7. Типичные ошибки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `SocketException` | Нет интернета | Проверяй connectivity перед запросом |
| `FormatException` в `jsonDecode` | Сервер вернул HTML (ошибка nginx) | Проверяй `statusCode` перед decode |
| `statusCode == 200` но пустые данные | Не передан `Accept: application/json` | Добавь заголовок headers |
| Утечка памяти | `Client()` создаётся и не закрывается | Вызывай `client.close()` или DI синглтон |
| `setState` после dispose | `await` завершился после ухода со страницы | Проверяй `mounted` после каждого `await` |

---

## 8. Рекомендации

1. **Всегда проверяй `statusCode`** перед обработкой тела ответа.
2. **Не создавай `Client()` в widget-методах** — используй Repository с синглтон-клиентом.
3. **Используй `Uri.parse()`** — никогда не передавай строку напрямую.
4. **Декодируй с явным приведением типов**: `jsonDecode(body) as Map<String, dynamic>`.
5. **Для продакшна переходи на Dio** — больше возможностей из коробки.
6. **Проверяй `mounted`** после любого `await` перед вызовом `setState`.
