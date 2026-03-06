# Квиз: Streams в Dart

**Тема:** 01.7 — Streams  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Чем `Stream` отличается от `Future`?

- A) `Future` быстрее
- B) `Stream` может испускать несколько значений во времени, `Future` — одно
- C) `Stream` синхронный, `Future` — асинхронный
- D) `Future` поддерживает отмену, `Stream` — нет

<details>
<summary>Ответ</summary>

**B) `Stream` может испускать несколько значений во времени, `Future` — одно**

`Future<T>` — одно значение (или ошибка) в будущем. `Stream<T>` — последовательность значений во времени: может испускать 0, 1 или много событий, и завершиться (или нет). Аналог: `Future` — HTTP запрос, `Stream` — WebSocket.

</details>

---

### Вопрос 2 🟢

Как подписаться на Stream?

- A) `stream.get()`
- B) `stream.listen((data) => ...)`
- C) `await stream`
- D) `stream.subscribe(() => ...)`

<details>
<summary>Ответ</summary>

**B) `stream.listen((data) => ...)`**

`listen()` подписывается на stream и возвращает `StreamSubscription`. Можно также использовать `await for (var item in stream) { }` в async функции — это идиоматичный Dart.

</details>

---

### Вопрос 3 🟢

Что такое broadcast Stream?

- A) Stream с высоким приоритетом
- B) Stream, на который могут подписаться несколько слушателей одновременно
- C) Stream который транслирует данные по сети
- D) Stream с буферизацией всех событий

<details>
<summary>Ответ</summary>

**B) Stream, на который могут подписаться несколько слушателей одновременно**

По умолчанию Stream — single-subscription (только один `listen()`). `asBroadcastStream()` или `StreamController.broadcast()` создают broadcast Stream с несколькими подписчиками. Новые подписчики не получают прошлые события.

</details>

---

### Вопрос 4 🟡

Что делает `StreamController`?

- A) Ограничивает скорость Stream
- B) Позволяет вручную управлять испусканием событий в Stream
- C) Конвертирует Future в Stream
- D) Отменяет Stream при ошибке

<details>
<summary>Ответ</summary>

**B) Позволяет вручную управлять испусканием событий в Stream**

```dart
final controller = StreamController<int>();
controller.stream; // Stream для подписки
controller.sink.add(42); // испустить событие
controller.sink.addError(Exception()); // испустить ошибку
controller.close(); // завершить stream
```

`StreamController` — основа для создания собственных потоков данных.

</details>

---

### Вопрос 5 🟡

Как трансформировать значения Stream?

- A) `stream.transform()`
- B) `stream.map((e) => ...)`
- C) Нельзя — Stream неизменяем
- D) `stream.convert((e) => ...)`

<details>
<summary>Ответ</summary>

**B) `stream.map((e) => ...)`**

`Stream` поддерживает методы аналогичные Iterable: `map`, `where`, `expand`, `take`, `skip`, `distinct`. Они возвращают новый Stream, не изменяя исходный. `transform()` принимает `StreamTransformer` для сложных трансформаций.

</details>

---

### Вопрос 6 🟡

Что произойдёт, если не отменить `StreamSubscription` при удалении виджета?

- A) Ничего, Stream автоматически очищается
- B) Утечка памяти и возможные вызовы setState на удалённый виджет
- C) Ошибка компиляции
- D) Stream автоматически замораживается

<details>
<summary>Ответ</summary>

**B) Утечка памяти и возможные вызовы setState на удалённом виджете**

Подписка удерживает ссылку на объект. После удаления виджета подписка продолжает работать и может вызывать `setState()` на уже удалённом `State`. Нужно хранить `StreamSubscription` и вызывать `subscription.cancel()` в `dispose()`.

</details>

---

### Вопрос 7 🟡

Что такое `async*` и `yield`?

- A) `async*` — более быстрый аналог `async`
- B) Генераторы: `async*` создаёт `Stream`, `yield` испускает значение
- C) `async*` — для параллельного выполнения
- D) `yield` — аналог `return` для `Future`

<details>
<summary>Ответ</summary>

**B) Генераторы: `async*` создаёт `Stream`, `yield` испускает значение**

```dart
Stream<int> countStream(int max) async* {
  for (int i = 0; i <= max; i++) {
    await Future.delayed(Duration(seconds: 1));
    yield i; // испустить i в stream
  }
}
```

`sync*` + `yield` создаёт `Iterable` синхронно. `yield*` разворачивает другой Stream/Iterable.

</details>

---

### Вопрос 8 🔴

Как объединить два Stream в один последовательный поток?

- A) `Stream.merge(s1, s2)`
- B) `StreamGroup.merge([s1, s2])` (пакет async)
- C) `s1.followedBy(s2)`
- D) `s1.concat(s2)`

<details>
<summary>Ответ</summary>

**B) `StreamGroup.merge([s1, s2])` (пакет async)**

В стандартной библиотеке нет встроенного merge. Пакет `async` предоставляет `StreamGroup.merge()`. Альтернатива: `rx_dart` с `MergeStream`. `followedBy` есть у `Iterable`, но не у `Stream`.

</details>

---

### Вопрос 9 🔴

Что выведет код?

```dart
void main() async {
  final stream = Stream.fromIterable([1, 2, 3]);
  final sum = await stream.reduce((a, b) => a + b);
  print(sum);
}
```

- A) `[1, 2, 3]`
- B) `6`
- C) `3`
- D) Ошибка: нельзя `await` Stream

<details>
<summary>Ответ</summary>

**B) `6`**

`reduce()` применяет функцию к элементам stream и возвращает `Future<T>` с финальным значением. `1+2=3`, `3+3=6`. `await` ждёт завершения stream и получает аккумулированный результат. Аналогично `fold()`, но без начального значения.

</details>

---

### Вопрос 10 🔴

В чём разница между `onData`, `onError` и `onDone` у `StreamSubscription`?

- A) Это методы для трёх разных типов Stream
- B) `onData` — событие данных, `onError` — событие ошибки, `onDone` — сигнал о закрытии Stream
- C) `onDone` вызывается только при ошибке
- D) `onError` отменяет подписку автоматически

<details>
<summary>Ответ</summary>

**B) `onData` — событие данных, `onError` — событие ошибки, `onDone` — сигнал о закрытии Stream**

```dart
stream.listen(
  (data) => print('Data: $data'),       // onData
  onError: (e) => print('Error: $e'),   // при ошибке, не завершает если cancelOnError: false
  onDone: () => print('Stream closed'), // stream завершён (controller.close())
  cancelOnError: false,                 // продолжить после ошибки
);
```

</details>
