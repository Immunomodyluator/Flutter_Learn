# 15.4 Isolates и compute()

## 1. Суть

Dart работает в **одном потоке** (event loop). Тяжёлые CPU-операции (парсинг большого JSON, шифрование, сжатие) блокируют UI-поток → джанки.

**Isolate** — отдельный поток Dart-кода, изолированный от основного (собственная память). Общение только через сообщения. `compute()` — удобная функция-обёртка для запуска изолята.

---

## 2. compute() — простой способ

```dart
import 'package:flutter/foundation.dart';

// Функция должна быть TOP-LEVEL или static (не лямбда, не метод инстанса)
List<Product> _parseProducts(String jsonString) {
  final json = jsonDecode(jsonString) as List<dynamic>;
  return json.map((e) => Product.fromJson(e as Map<String, dynamic>)).toList();
}

// Вызов в UI
Future<void> loadProducts() async {
  final jsonString = await http.get(Uri.parse('/api/products'));

  // Парсинг в отдельном изоляте, не блокирует UI
  final products = await compute(_parseProducts, jsonString.body);

  setState(() => _products = products);
}
```

---

## 3. Когда нужен compute()

```
< 16 мс  → выполняй в основном потоке
> 16 мс  → используй compute() или Isolate
```

Типичные кандидаты:

- Парсинг JSON > 1 МБ
- Шифрование/хэширование
- Сжатие/декомпрессия
- Обработка изображений (фильтры, ресайзинг)
- Сложные вычисления (ML inference, маршрутизация)
- Сортировка/фильтрация большого массива данных

---

## 4. Isolate.run() — Dart 2.19+

```dart
import 'dart:isolate';

// Более современная и удобная версия compute()
Future<List<Product>> loadProducts() async {
  final jsonString = await dio.get<String>('/api/products');

  return Isolate.run(() {
    // Код внутри выполняется в изоляте
    final json = jsonDecode(jsonString.data!) as List;
    return json.map((e) => Product.fromJson(e as Map<String, dynamic>)).toList();
  });
}
```

---

## 5. Ручное управление Isolate

```dart
import 'dart:isolate';

// Для долгих операций с потоком сообщений
class HeavyIsolate {
  late Isolate _isolate;
  late ReceivePort _receivePort;
  late SendPort _sendPort;

  Future<void> start() async {
    _receivePort = ReceivePort();
    _isolate = await Isolate.spawn(
      _isolateEntry,
      _receivePort.sendPort,
    );

    // Получаем SendPort изолята
    _sendPort = await _receivePort.first as SendPort;
  }

  // Функция запускается в изоляте (top-level!)
  static void _isolateEntry(SendPort mainSendPort) {
    final port = ReceivePort();
    mainSendPort.send(port.sendPort); // отправляем наш порт в main

    port.listen((message) {
      if (message is String) {
        final result = _processHeavyTask(message);
        mainSendPort.send(result);
      }
    });
  }

  static String _processHeavyTask(String input) {
    // ... тяжёлые вычисления ...
    return 'processed: $input';
  }

  Future<String> process(String data) async {
    final responsePort = ReceivePort();
    _sendPort.send({'data': data, 'replyTo': responsePort.sendPort});
    return await responsePort.first as String;
  }

  void dispose() {
    _isolate.kill();
    _receivePort.close();
  }
}
```

---

## 6. Практический пример — парсинг и обработка

```dart
// --- isolate_tasks.dart (top-level функции) ---
Map<String, dynamic> _processJsonTask(Map<String, dynamic> params) {
  final jsonString = params['json'] as String;
  final filter = params['filter'] as String;

  final allItems = (jsonDecode(jsonString) as List)
      .map((e) => Item.fromJson(e as Map<String, dynamic>))
      .toList();

  final filtered = allItems
      .where((item) => item.category == filter)
      .toList();

  // Данные должны быть сериализуемы (никаких Flutter-объектов!)
  return {
    'items': filtered.map((e) => e.toJson()).toList(),
    'count': filtered.length,
  };
}

// --- data_service.dart ---
class DataService {
  Future<List<Item>> fetchAndFilterItems(String filter) async {
    final response = await dio.get<String>('/api/items');

    final result = await compute(
      _processJsonTask,
      {'json': response.data!, 'filter': filter},
    );

    final items = (result['items'] as List)
        .map((e) => Item.fromJson(e as Map<String, dynamic>))
        .toList();

    return items;
  }
}
```

---

## 7. Ограничения Isolate

```dart
// ❌ Нельзя передавать в изолят:
// - Объекты с финальными внутренними указателями (SocketException и др.)
// - Замыкания (closures), классы с ними
// - Dart:ui объекты (Widget, Canvas, TextStyle)
// - FlutterEngine, методы платформы

// ✅ Можно передавать:
// - Примитивы (int, double, bool, String)
// - List, Map со сериализуемыми значениями
// - Uint8List, ByteData (изображения, файлы)
// - Записи (records) с примитивами
```

---

## 8. flutter_isolate для platform channels

`dart:isolate` не поддерживает Platform Channels. Для этого есть `flutter_isolate`:

```yaml
dependencies:
  flutter_isolate: ^2.0.4
```

```dart
import 'package:flutter_isolate/flutter_isolate.dart';

// top-level функция
@pragma('vm:entry-point')
void isolateMain(SendPort sendPort) {
  // Можно использовать Platform Channels здесь
}

final isolate = await FlutterIsolate.spawn(isolateMain, receivePort.sendPort);
```

---

## 9. Под капотом

- `compute(fn, message)` = `Isolate.run(() => fn(message))` (упрощённо).
- Изоляты не разделяют память — данные **копируются** при передаче.
- Передача большого объекта (`Uint8List`) происходит через **transfer** без копирования (TransferableTypedData).
- `Isolate.spawn` имеет overhead ~2-5 мс на создание — не создавай изолят для микрозадач.

---

## 10. Типичные ошибки

| Ошибка                                  | Причина                                | Решение                                     |
| --------------------------------------- | -------------------------------------- | ------------------------------------------- |
| `Illegal argument in isolate message`   | Передаётся несериализуемый объект      | Передавать только примитивы/JSON            |
| Метод экземпляра как entrypoint         | Изоляты принимают только top-level     | Пометить функцию как `static` или top-level |
| Изолят создаётся на каждый запрос       | Overhead на создание > полезная работа | Пул изолятов или `IsolateChannel`           |
| UI тормозит при большой передаче данных | Копирование большого Uint8List         | Использовать `TransferableTypedData`        |

---

## 11. Рекомендации

1. **`Isolate.run()`** — первый выбор для разовых задач (проще `compute()`).
2. **`compute()`** — совместим с более старыми версиями Flutter.
3. **Сериализуй данные** в/из изолята — JSON-строки или примитивы.
4. **Не создавай изолят для каждого запроса** — используй переиспользуемый пул для частых задач.
5. **Измеряй время** задачи в обычном потоке — если < 5 мс, изолят не нужен (overhead превысит выгоду).
