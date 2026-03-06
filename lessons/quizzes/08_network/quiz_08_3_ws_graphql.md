# Квиз: WebSockets и GraphQL

**Тема:** 08.3 — WebSockets / GraphQL  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Чем WebSocket отличается от обычного HTTP запроса?

- A) WebSocket только для мобильных приложений
- B) WebSocket — двунаправленное персистентное соединение; HTTP — запрос-ответ и закрытие
- C) WebSocket быстрее, но менее надёжен
- D) WebSocket использует другой протокол (FTP)

<details>
<summary>Ответ</summary>

**B) WebSocket — двунаправленное персистентное соединение**

WebSocket: клиент и сервер могут отправлять данные в любой момент без нового запроса. Идеально для: чатов, real-time обновлений, live dashboards. HTTP polling — альтернатива, но менее эффективна.

</details>

---

### Вопрос 2 🟢

Как создать WebSocket соединение в Dart?

- A) `HttpClient().connectWs(url)`
- B) `WebSocket.connect(url)` из `dart:io`
- C) `WebSocketChannel.connect(Uri.parse(url))` из пакета `web_socket_channel`
- D) B и C оба корректны

<details>
<summary>Ответ</summary>

**D) B и C оба корректны**

`dart:io` `WebSocket` — только для нативных платформ. `web_socket_channel` — кросс-платформенный (Web + Mobile + Desktop):

```dart
final channel = WebSocketChannel.connect(Uri.parse('wss://echo.websocket.org'));
channel.stream.listen((message) => print(message));
channel.sink.add('Hello WebSocket!');
```

</details>

---

### Вопрос 3 🟢

Что такое GraphQL?

- A) База данных для мобильных приложений
- B) Язык запросов для API — клиент запрашивает ровно те поля которые нужны, не больше и не меньше
- C) Расширение REST API
- D) Пакет для Flutter для работы с базами данных

<details>
<summary>Ответ</summary>

**B) Язык запросов для API — клиент запрашивает только нужные поля**

GraphQL преимущества над REST:

- **No over-fetching**: только запрошенные поля
- **No under-fetching**: данные из нескольких ресурсов в одном запросе
- **Strongly typed**: схема определяет все типы
- **Subscriptions**: real-time через WebSocket

</details>

---

### Вопрос 4 🟡

Как отправить и получить данные через `web_socket_channel`?

- A) `channel.send()` и `channel.receive()`
- B) `channel.sink.add(data)` для отправки; `channel.stream.listen(callback)` для получения
- C) `channel.write(data)` и `channel.read()`
- D) `channel.emit(event, data)` и `channel.on(event, callback)`

<details>
<summary>Ответ</summary>

**B) `sink.add` для отправки, `stream.listen` для получения**

```dart
final channel = WebSocketChannel.connect(Uri.parse('wss://...'));

// Отправить:
channel.sink.add(jsonEncode({'type': 'message', 'text': 'Hello'}));

// Получить:
channel.stream.listen(
  (data) => handleMessage(jsonDecode(data)),
  onError: (error) => reconnect(),
  onDone: () => print('Connection closed'),
);

// Закрыть:
await channel.sink.close(status.goingAway);
```

</details>

---

### Вопрос 5 🟡

Как использовать GraphQL во Flutter с пакетом `graphql_flutter`?

- A) `GraphQL.query('''{ users { name } }''')`
- B) `GraphQLClient` + `Query`/`Mutation` виджеты или `client.query(QueryOptions(document: gql(...)))`
- C) `HttpClient.graphql(url, query: '...')`
- D) GraphQL использует REST API под капотом

<details>
<summary>Ответ</summary>

**B) `GraphQLClient` + виджеты или программный вызов**

```dart
final client = GraphQLClient(
  cache: GraphQLCache(),
  link: HttpLink('https://api.example.com/graphql'),
);

// Программно:
final result = await client.query(QueryOptions(
  document: gql(r'''query { users { id name } }'''),
));
if (!result.hasException) {
  final users = result.data!['users'];
}
```

</details>

---

### Вопрос 6 🟡

Что такое GraphQL Subscription и как реализовать его во Flutter?

- A) Подписка на уведомления о новых версиях пакетов
- B) Real-time механизм GraphQL через WebSocket — сервер push-ает данные при изменениях
- C) Аналог REST polling
- D) Подписка доступна только в GraphQL Premium

<details>
<summary>Ответ</summary>

**B) Real-time через WebSocket — сервер отправляет данные при изменениях**

```dart
final subscriptionDocument = gql(r'''
  subscription OnMessageAdded {
    messageAdded { id text author }
  }
''');

final subscription = client.subscribe(SubscriptionOptions(
  document: subscriptionDocument,
));

subscription.listen((result) {
  final message = result.data!['messageAdded'];
  // обновить UI
});
```

</details>

---

### Вопрос 7 🟡

Как переподключиться к WebSocket при обрыве связи?

- A) `web_socket_channel` автоматически переподключается
- B) Обработать `onDone`/`onError` в stream listener и реализовать логику retry с exponential backoff
- C) `channel.reconnect()`
- D) Через `WebSocket.reconnectPolicy`

<details>
<summary>Ответ</summary>

**B) Обработать `onDone`/`onError` и реализовать retry**

```dart
Future<void> connect() async {
  channel = WebSocketChannel.connect(uri);
  channel.stream.listen(
    onMessage,
    onError: (e) => scheduleReconnect(),
    onDone: () => scheduleReconnect(),
  );
}

void scheduleReconnect() {
  Future.delayed(Duration(seconds: min(_retryCount++ * 2, 30)), connect);
}
```

</details>

---

### Вопрос 8 🔴

Как кэшировать GraphQL запросы для работы офлайн?

- A) GraphQL не поддерживает кэширование
- B) `GraphQLCache` с `InMemoryStore` или `HiveStore` (persistent) — `graphql_flutter` хранит нормализованный кэш
- C) Вручную через `SharedPreferences`
- D) Только через database layer

<details>
<summary>Ответ</summary>

**B) `GraphQLCache` с `InMemoryStore` или `HiveStore`**

```dart
final store = await HiveStore.open(); // persistent кэш
final cache = GraphQLCache(store: store);

final client = GraphQLClient(
  cache: cache,
  link: HttpLink(...),
);

// Политика:
client.query(QueryOptions(
  fetchPolicy: FetchPolicy.cacheFirst, // кэш → сеть
  // FetchPolicy.cacheOnly — только кэш (офлайн)
  // FetchPolicy.networkOnly — только сеть
));
```

</details>

---

### Вопрос 9 🔴

Как реализовать heartbeat (keep-alive) для WebSocket соединения?

- A) WebSocket встроенно поддерживает ping/pong — ничего не нужно
- B) `Timer.periodic` + отправка ping-сообщения — если нет ответа за timeout → reconnect
- C) `channel.ping()` метод
- D) HTTP keep-alive автоматически работает для WebSocket

<details>
<summary>Ответ</summary>

**B) `Timer.periodic` + ping + timeout для detect disconnect**

```dart
Timer? _heartbeat;

void startHeartbeat() {
  _heartbeat = Timer.periodic(Duration(seconds: 30), (_) {
    channel.sink.add(jsonEncode({'type': 'ping'}));
    _pongTimer = Timer(Duration(seconds: 10), () {
      // нет pong через 10 сек → reconnect
      reconnect();
    });
  });
}

void handleMessage(dynamic data) {
  final msg = jsonDecode(data);
  if (msg['type'] == 'pong') _pongTimer?.cancel();
}
```

</details>

---

### Вопрос 10 🔴

Как организовать типизацию GraphQL запросов в Dart?

- A) Только через ручное написание `fromJson`
- B) `graphql_codegen` пакет генерирует Dart типы из GraphQL schema + `.graphql` файлов операций
- C) Использовать `dynamic` → GraphQL не поддерживает типизацию
- D) `Artemis` пакет (устарел в 2024)

<details>
<summary>Ответ</summary>

**B) `graphql_codegen` генерирует Dart типы из GraphQL schema**

Workflow:

1. Скачать schema: `introspect` endpoint → `schema.graphql`
2. Написать `.graphql` файлы с операциями
3. `graphql_codegen` генерирует: `GetUsersQuery`, `GetUsersQueryResult`, type-safe переменные
4. `client.query(GetUsersQuery.document)` → типизированный результат

Также популярен `ferry` пакет — полный GraphQL клиент с кодогенерацией.

</details>
