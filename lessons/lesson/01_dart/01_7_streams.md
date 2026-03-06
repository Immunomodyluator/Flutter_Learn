# 1.7 Streams

## 1. Суть концепции

Относится к: **async runtime, реактивное программирование**.

`Stream<T>` — поток данных: последовательность асинхронных событий во времени. В отличие от `Future` (одно значение → завершение), Stream может выдавать **много значений** со временем или вообще никогда не завершаться.

Аналогия: `Future` — доставка одного письма. `Stream` — подписка на газету: события приходят когда есть новости.

---

## 2. Как используется во Flutter

- **Firebase Firestore** — `stream()` для realtime обновлений документов
- **Firebase Auth** — `authStateChanges()` — поток изменений состояния авторизации
- **BLoC паттерн** — события и состояния передаются через StreamController
- **`StreamBuilder`** — виджет, перерисовывающийся при каждом новом event в потоке
- **Bluetooth, WebSocket, GPS** — постоянные потоки данных от устройства

---

## 3. Синтаксис и базовый пример

### Создание и подписка

```dart
// Простой поток из значений
final stream = Stream.fromIterable([1, 2, 3, 4, 5]);

// Подписка через listen
stream.listen(
  (data) => print('Data: $data'),
  onError: (e) => print('Error: $e'),
  onDone: () => print('Stream closed'),
);

// Поток с задержкой — отправляет event каждую секунду
final ticker = Stream.periodic(
  const Duration(seconds: 1),
  (i) => i, // (count) => значение
);
```

### async\* и yield — генератор потока

```dart
// async* — функция-генератор потока
Stream<int> countDown(int from) async* {
  for (int i = from; i >= 0; i--) {
    await Future.delayed(const Duration(seconds: 1));
    yield i; // отправить значение в поток
  }
}

// Использование
countDown(5).listen((n) => print(n)); // 5, 4, 3, 2, 1, 0
```

### await for — итерация по потоку

```dart
Future<void> processEvents() async {
  await for (final event in eventStream) {
    // обрабатываем каждый event
    print('Got: $event');
  }
  // код после: выполняется когда поток закрыт
}
```

### StreamController — создание кастомного потока

```dart
// Single-subscription (одна подписка)
final controller = StreamController<String>();

// Broadcast (несколько подписчиков)
final broadcastController = StreamController<String>.broadcast();

// Добавить событие в поток
controller.sink.add('Hello');
controller.sink.add('World');
controller.sink.addError(Exception('oops'));

// Закрыть поток
controller.close();

// Подписаться
controller.stream.listen((data) => print(data));
```

### Трансформации потока

```dart
final numbers = Stream.fromIterable([1, 2, 3, 4, 5, 6]);

numbers
    .where((n) => n.isEven)        // фильтр
    .map((n) => n * 2)             // трансформация
    .take(2)                        // взять первые 2
    .listen(print);                 // 4, 8

// debounce (через rxdart или вручную)
searchStream
    .debounceTime(const Duration(milliseconds: 300)) // rxdart
    .distinct()    // пропускать дублирующиеся значения
    .listen((query) => search(query));
```

---

## 4. Реальный пример из Flutter

```dart
// Realtime чат — сообщения из Firestore через StreamBuilder
class ChatScreen extends StatelessWidget {
  final String chatId;

  const ChatScreen({super.key, required this.chatId});

  Stream<List<Message>> _messagesStream() {
    return FirebaseFirestore.instance
        .collection('chats/$chatId/messages')
        .orderBy('timestamp', descending: true)
        .limit(50)
        .snapshots() // Stream<QuerySnapshot> — обновляется в реальном времени
        .map((snap) => snap.docs.map(Message.fromDoc).toList());
  }

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<List<Message>>(
      stream: _messagesStream(),
      builder: (context, snapshot) {
        // Состояния аналогичны FutureBuilder
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }

        if (snapshot.hasError) {
          return Center(child: Text('Ошибка: ${snapshot.error}'));
        }

        final messages = snapshot.data ?? [];

        if (messages.isEmpty) {
          return const Center(child: Text('Нет сообщений'));
        }

        return ListView.builder(
          reverse: true,
          itemCount: messages.length,
          itemBuilder: (context, i) => MessageBubble(message: messages[i]),
        );
      },
    );
  }
}
```

`StreamBuilder` не нужно отписываться вручную — он делает это автоматически при `dispose`.

---

## 5. Что происходит под капотом

### Single-subscription vs Broadcast

```dart
// Single-subscription: слушать можно ТОЛЬКО ОДИН РАЗ
final stream = someStream;
stream.listen(...); // OK
stream.listen(...); // StateError: Stream has already been listened to

// Broadcast: несколько слушателей
final broadcast = stream.asBroadcastStream();
broadcast.listen(...); // OK
broadcast.listen(...); // OK — оба получают события
```

`Stream` из Firestore — всегда broadcast-ready. `StreamController.broadcast()` создаёт broadcaster вручную.

### Отмена подписки (cancel)

```dart
class _MyWidgetState extends State<MyWidget> {
  StreamSubscription<Event>? _subscription;

  @override
  void initState() {
    super.initState();
    _subscription = eventStream.listen(_handleEvent);
  }

  @override
  void dispose() {
    _subscription?.cancel(); // обязательно! иначе утечка памяти
    super.dispose();
  }
}
```

`StreamBuilder` делает это автоматически. При ручной `listen` — всегда `cancel` в `dispose`.

### Backpressure

Dart Stream не имеет встроенного backpressure. Если producer быстрее consumer — события накапливаются в памяти. Для контроля используют `pause()`/`resume()` на `StreamSubscription`.

---

## 6. Типичные ошибки

| Ошибка                                            | Почему возникает                            | Правильно                                             |
| ------------------------------------------------- | ------------------------------------------- | ----------------------------------------------------- |
| Не отменять `StreamSubscription`                  | Утечка памяти, события после `dispose`      | `cancel()` в `dispose()`                              |
| `listen()` на single-subscription дважды          | `StateError` в runtime                      | Использовать `asBroadcastStream()` или один слушатель |
| Не закрывать `StreamController`                   | Утечка ресурсов                             | `controller.close()` в `dispose()`                    |
| `StreamBuilder` с новым Stream в каждом `build()` | Stream пересоздаётся, подписка сбрасывается | Хранить Stream в State, не создавать в `build()`      |
| Не обрабатывать `onError`                         | Необработанная ошибка в потоке = crash      | Добавить `onError` или `.handleError()`               |

---

## 7. Практические рекомендации

1. **`StreamBuilder` > ручной `listen`** в виджетах — автоматически управляет подпиской и `dispose`.
2. **Храни `Stream` в State или Provider**, не создавай в `build()` — аналогично `Future`.
3. **`StreamController.broadcast()` для шины событий** внутри приложения (шина событий, EventBus паттерн).
4. **`rxdart`** — если нужны debounce, throttle, combineLatest, switchMap — стандарт для сложных stream-операций.
5. **Закрывай StreamController** в `dispose()` — это важно, иначе ресурсы не освобождаются.
6. **`distinct()`** на поiske — не отправляй запрос если текст не изменился.
7. **BLoC паттерн построен на StreamController** — понимание Stream — ключ к пониманию Bloc/Cubit.
