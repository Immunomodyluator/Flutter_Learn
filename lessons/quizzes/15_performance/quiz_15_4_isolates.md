# Квиз: Isolates

**Тема:** 15.4 — Isolates & Background Computation  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое Isolate в Dart и зачем он нужен?

- A) Аналог Thread в Java с общей памятью
- B) Независимый поток выполнения с собственной кучей памяти — не блокирует UI thread при тяжёлых вычислениях
- C) Async/await — тот же Isolate
- D) Isolate — только для сетевых запросов

<details>
<summary>Ответ</summary>

**B) Независимый поток с собственной памятью**

```
Dart модель:
- UI Isolate: main isolate, обрабатывает события, строит виджеты
- Каждый Isolate: своя куча (heap), нет shared memory
- Общение: только через сообщения (SendPort/ReceivePort)

Когда использовать Isolate:
✅ Парсинг большого JSON (> 1 MB)
✅ Обработка изображений
✅ Шифрование/дешифрование
✅ Сложные вычисления (ML, алгоритмы)
✅ Чтение/запись больших файлов

Не нужен для:
❌ HTTP запросы (уже async, не блокируют)
❌ SharedPreferences (быстрые операции)
❌ Небольшой JSON (< 100 KB)
```

</details>

---

### Вопрос 2 🟢

Как запустить одноразовое вычисление в отдельном Isolate?

- A) `Thread.start(() => heavyWork())`
- B) `Isolate.run(() => heavyWork())` (Flutter 3+) или `compute(heavyWork, input)`
- C) `Future.microtask(() => heavyWork())`
- D) `async { await heavyWork() }`

<details>
<summary>Ответ</summary>

**B) `Isolate.run()` или `compute()`**

```dart
// Isolate.run (Dart 2.19 / Flutter 3.7+):
Future<List<Product>> parseProducts(String json) async {
  return Isolate.run(() {
    final data = jsonDecode(json) as List;
    return data.map(Product.fromJson).toList();
  });
}

// compute() — Flutter обёртка, аналогично:
Future<List<Product>> parseProducts(String json) {
  return compute(_parseInBackground, json);
}

// Функция должна быть top-level или static:
List<Product> _parseInBackground(String json) {
  final data = jsonDecode(json) as List;
  return data.map(Product.fromJson).toList();
}

// ВАЖНО: замыкания с захватом переменных НЕ работают в compute()!
// ❌ compute(() => doWork(localVar), input) — ошибка
// ✅ top-level функция или static метод
```

</details>

---

### Вопрос 3 🟢

Чем `Isolate.run()` отличается от `compute()`?

- A) Это одно и то же
- B) `Isolate.run()` — Dart SDK, принимает любое замыкание; `compute()` — Flutter, только top-level/static функции с одним параметром
- C) `compute()` доступен только на мобильных платформах
- D) `Isolate.run()` не возвращает результат

<details>
<summary>Ответ</summary>

**B) `Isolate.run` — более гибкий**

```dart
// Isolate.run — любое замыкание, без ограничений:
final result = await Isolate.run(() {
  // Может замкнуть переменные!
  final config = localConfig;  // ✅ работает
  return expensiveComputation(config);
});

// compute — только top-level/static + один параметр:
// ❌ compute(() => work(localVar), null)  // ошибка
// ✅ compute(_topLevelFn, singleParam)

// Для передачи нескольких параметров в compute — Record или Map:
Future<String> processData(List<String> items, String format) {
  return compute(_process, (items, format)); // record как параметр
}

String _process((List<String>, String) params) {
  final (items, format) = params;
  return items.map((i) => '$i.$format').join(',');
}
```

</details>

---

### Вопрос 4 🟡

Как реализовать двустороннее общение между Isolate-ами через `SendPort`/`ReceivePort`?

- A) Использовать `SharedMemory` пакет
- B) Создать `ReceivePort` в main isolate → передать `sendPort` в spawned isolate → обмен сообщениями через `send()`
- C) `Isolate.send()` статический метод
- D) Через `StreamController`

<details>
<summary>Ответ</summary>

**B) `ReceivePort` + `SendPort` двусторонняя коммуникация**

```dart
Future<void> startWorkerIsolate() async {
  // Порт для получения сообщений от isolate:
  final receivePort = ReceivePort();

  // Запустить isolate, передав sendPort:
  await Isolate.spawn(_workerIsolate, receivePort.sendPort);

  // Слушать сообщения:
  receivePort.listen((message) {
    if (message is SendPort) {
      // Получили порт для отправки задач:
      final workerSendPort = message;
      workerSendPort.send({'task': 'process', 'data': [1, 2, 3]});
    } else if (message is Map) {
      print('Результат: ${message['result']}');
    }
  });
}

void _workerIsolate(SendPort mainSendPort) {
  final workerReceivePort = ReceivePort();
  // Отправить свой SendPort для получения задач:
  mainSendPort.send(workerReceivePort.sendPort);

  workerReceivePort.listen((message) {
    final task = message as Map;
    // Обработать задачу:
    final result = task['data'].reduce((a, b) => a + b);
    mainSendPort.send({'result': result});
  });
}
```

</details>

---

### Вопрос 5 🟡

Что такое Isolate Groups в Flutter 3+ и чем они полезны?

- A) Группа связанных Isolate-ов с общим GC
- B) Isolate внутри одной группы могут делить иммутабельные объекты без копирования — быстрее передача больших данных
- C) Только в Flutter Web
- D) Isolate Groups — экспериментальная функция, не использовать в production

<details>
<summary>Ответ</summary>

**B) Общие иммутабельные объекты без копирования**

```dart
// До Isolate Groups — данные КОПИРУЮТСЯ при передаче:
// Передача 10 MB данных = 10 MB копирования = долго

// С Isolate Groups (Flutter 3+, Dart 2.15+):
// Иммутабельные объекты передаются по ссылке — без копирования!

// Isolate.run() автоматически использует isolate groups:
final bigData = List.generate(1000000, (i) => i); // 8 MB

// Передаётся по ссылке (если immutable) — быстро:
final result = await Isolate.run(() {
  return bigData.fold(0, (a, b) => a + b); // bigData не копируется!
});

// Ключевое условие: данные должны быть:
// - Примитивы (int, double, bool, String)
// - Иммутабельные (final поля, frozen объекты)
// - Transferable через SendPort (TransferableTypedData)
```

</details>

---

### Вопрос 6 🟡

Как реализовать worker pool для параллельной обработки задач?

- A) `IsolatePool` встроен в Flutter
- B) Создать несколько `Isolate` через `Isolate.spawn`; распределять задачи через `SendPort`; использовать пакет `worker_manager`
- C) `Isolate.pool(count: 4, () => work())`
- D) Worker pool не нужен — `compute()` сам распределяет

<details>
<summary>Ответ</summary>

**B) Ручной pool или пакет `worker_manager`**

```dart
// Простой Worker Pool:
class IsolatePool {
  final int size;
  final _workers = <_Worker>[];
  int _nextWorker = 0;

  IsolatePool(this.size);

  Future<void> initialize() async {
    for (var i = 0; i < size; i++) {
      final worker = _Worker();
      await worker.spawn();
      _workers.add(worker);
    }
  }

  Future<T> run<T>(Future<T> Function() task) {
    final worker = _workers[_nextWorker++ % size];
    return worker.execute(task);
  }

  void dispose() => _workers.forEach((w) => w.kill());
}

// Пакет worker_manager:
// WorkerManager.instance.execute(task, priority: Priority.high)

// Параллельная обработка изображений:
final pool = IsolatePool(Platform.numberOfProcessors);
await pool.initialize();

final results = await Future.wait(
  images.map((img) => pool.run(() => processImage(img))),
);
```

</details>

---

### Вопрос 7 🟡

Как передавать большие объёмы данных между Isolate-ами без копирования?

- A) Любые данные передаются без копирования
- B) `TransferableTypedData` — передаёт `ByteBuffer` без копирования; иммутабельные объекты в isolate groups тоже без копирования
- C) Только через файловую систему
- D) `Isolate.sendRaw()`

<details>
<summary>Ответ</summary>

**B) `TransferableTypedData` для ByteBuffer**

```dart
// Стандартный Uint8List — КОПИРУЕТСЯ при передаче:
final bytes = Uint8List(10 * 1024 * 1024); // 10 MB
sendPort.send(bytes); // ← копирует 10 MB!

// TransferableTypedData — передаётся без копирования:
final transferable = TransferableTypedData.fromList([bytes]);
sendPort.send(transferable); // ← НЕ копирует!

// В принимающем isolate — восстановить:
receivePort.listen((message) {
  if (message is TransferableTypedData) {
    final bytes = message.materialize().asUint8List();
    // Работать с bytes...
  }
});

// ВАЖНО: TransferableTypedData можно использовать ОДИН РАЗ!
// После materialize() — исходный объект становится недействительным.

// Пример: передача сырых данных изображения:
final imageBytes = await file.readAsBytes();
final transferable = TransferableTypedData.fromList([imageBytes]);
workerSendPort.send({'type': 'decode', 'data': transferable});
```

</details>

---

### Вопрос 8 🔴

Как обрабатывать ошибки из Isolate?

- A) Ошибки в Isolate игнорируются
- B) `Isolate.run()` пробрасывает исключения; для `Isolate.spawn()` — `errorPort` параметр или `addErrorListener()`
- C) `try/catch` в main isolate поймает ошибки
- D) Только через `Zone.runGuarded()`

<details>
<summary>Ответ</summary>

**B) `Isolate.run()` пробрасывает; `spawn()` — errorPort**

```dart
// Isolate.run — ошибки пробрасываются напрямую:
try {
  final result = await Isolate.run(() {
    throw FormatException('Некорректные данные');
  });
} on FormatException catch (e) {
  print('Ошибка парсинга: $e'); // ✅ поймано
}

// Isolate.spawn — нужен errorPort:
Future<void> spawnWithErrorHandling() async {
  final receivePort = ReceivePort();
  final errorPort = ReceivePort();
  final exitPort = ReceivePort();

  await Isolate.spawn(
    _worker,
    receivePort.sendPort,
    onError: errorPort.sendPort,   // ошибки
    onExit: exitPort.sendPort,     // завершение
    errorsAreFatal: false,         // не убивать isolate при ошибке
  );

  errorPort.listen((error) {
    final errorList = error as List;
    print('Ошибка isolate: ${errorList[0]}');
    print('Stack: ${errorList[1]}');
  });
}
```

</details>

---

### Вопрос 9 🔴

Как использовать `FlutterIsolate` или background isolate channels для работы с Flutter плагинами из Isolate?

- A) Все Flutter плагины работают из любого Isolate
- B) `BackgroundIsolateBinaryMessenger.ensureInitialized(token)` — позволяет вызывать platform channels из фонового Isolate (Flutter 3.7+)
- C) `FlutterIsolate.spawn()` из `flutter_isolate` пакета единственный способ
- D) Плагины нельзя использовать из Isolate

<details>
<summary>Ответ</summary>

**B) `BackgroundIsolateBinaryMessenger` (Flutter 3.7+)**

```dart
// Получить RootIsolateToken в main isolate:
final rootToken = RootIsolateToken.instance!;

// Передать в фоновый Isolate:
await Isolate.spawn(_backgroundTask, rootToken);

void _backgroundTask(RootIsolateToken token) async {
  // Инициализировать binary messenger для фонового Isolate:
  BackgroundIsolateBinaryMessenger.ensureInitialized(token);
  WidgetsFlutterBinding.ensureInitialized();

  // Теперь можно использовать platform channels:
  final prefs = await SharedPreferences.getInstance(); // ✅
  final dbPath = await getDatabasesPath();              // ✅
  final docs = await getApplicationDocumentsDirectory(); // ✅

  // Делать тяжёлую работу с доступом к нативным API:
  final db = await openDatabase(dbPath + '/app.db');
  final rows = await db.query('large_table');
  // обработать rows...
}
```

</details>

---

### Вопрос 10 🔴

Как правильно организовать background processing для сложного приложения?

- A) Один глобальный Isolate для всего
- B) Выделить специальные Isolate для тяжёлых задач (image processing, DB operations); использовать `Isolate.run()` для одноразовых; Worker Pool для потока задач
- C) Переносить всю логику на сервер
- D) Использовать `Future.wait()` — это достаточно

<details>
<summary>Ответ</summary>

**B) Специализированные Isolate по задачам**

```dart
// Архитектура background processing:

// 1. Image Isolate — обработка изображений:
class ImageProcessor {
  Future<Uint8List> compress(Uint8List image, int quality) =>
      Isolate.run(() => _compressInIsolate((image, quality)));
}

// 2. Database Isolate — тяжёлые запросы:
class BackgroundDatabase {
  late Isolate _dbIsolate;
  late SendPort _sendPort;

  Future<void> initialize() async {
    final token = RootIsolateToken.instance!;
    await Isolate.spawn(_dbWorker, (token, _receivePort.sendPort));
  }
}

// 3. Crypto Isolate — шифрование:
Future<String> encryptData(String data, String key) =>
    Isolate.run(() => AES.encrypt(data, key).base64);

// 4. Batch processor — обработка очереди:
final pool = IsolatePool(Platform.numberOfProcessors - 1);

// Принципы:
// - UI isolate только для UI логики
// - Передача данных через TransferableTypedData
// - Graceful shutdown: Isolate.kill()
// - Мониторинг: Timeline.startSync() для профилирования
```

</details>
