# Квиз: Асинхронность в Dart

**Тема:** 01.6 — Async/Await  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что возвращает функция, помеченная `async`?

- A) `void`
- B) `Future`
- C) `Stream`
- D) `dynamic`

<details>
<summary>Ответ</summary>

**B) `Future`**

`async` функция всегда возвращает `Future<T>`, где `T` — тип возвращаемого значения. Если функция возвращает `void` — это `Future<void>`. Это позволяет вызывающему коду использовать `await` или `.then()`.

</details>

---

### Вопрос 2 🟢

Что делает `await` перед `Future`?

- A) Запускает Future в отдельном потоке
- B) Приостанавливает выполнение текущей функции до завершения Future
- C) Отменяет Future
- D) Преобразует Future в синхронное значение навсегда

<details>
<summary>Ответ</summary>

**B) Приостанавливает выполнение текущей функции до завершения Future**

`await` не блокирует поток — он возвращает управление event loop до завершения Future, а затем продолжает выполнение. Это неблокирующая пауза: другие события могут обрабатываться пока Future выполняется.

</details>

---

### Вопрос 3 🟢

Как правильно обработать ошибку при использовании `await`?

- A) `.catchError()`
- B) `try { await ... } catch (e) { ... }`
- C) `if (await future == null)`
- D) `await future.onError()`

<details>
<summary>Ответ</summary>

**B) `try { await ... } catch (e) { ... }`**

С `async/await` можно использовать привычный `try/catch`. Это делает код более читаемым по сравнению с цепочками `.then().catchError()`. Оба варианта работают, но `try/catch` с `await` — идиоматичный Dart.

</details>

---

### Вопрос 4 🟡

Что такое Event Loop в Dart?

- A) Цикл `for` для обработки событий
- B) Однопоточный механизм обработки событий и микрозадач
- C) Многопоточный пул задач
- D) Системный процесс Android/iOS

<details>
<summary>Ответ</summary>

**B) Однопоточный механизм обработки событий и микрозадач**

Dart однопоточен. Event Loop обрабатывает две очереди: **Microtask Queue** (Priority: `Future.microtask`, `scheduleMicrotask`) — высокий приоритет, и **Event Queue** — I/O, таймеры, `Future.delayed`. Микрозадачи всегда выполняются перед событиями.

</details>

---

### Вопрос 5 🟡

Что выведет код?

```dart
void main() {
  print('1');
  Future(() => print('2'));
  Future.microtask(() => print('3'));
  print('4');
}
```

- A) `1 2 3 4`
- B) `1 4 2 3`
- C) `1 4 3 2`
- D) `1 3 4 2`

<details>
<summary>Ответ</summary>

**C) `1 4 3 2`**

Синхронный код выполняется первым: `1`, `4`. Затем microtask queue (приоритет выше): `3`. Затем event queue: `2`. `Future(() => ...)` помещается в event queue, `Future.microtask` — в microtask queue.

</details>

---

### Вопрос 6 🟡

Как выполнить несколько Future **параллельно** и дождаться всех?

- A) `await future1; await future2;`
- B) `await Future.wait([future1, future2])`
- C) `Future.all([future1, future2])`
- D) `async.parallel([future1, future2])`

<details>
<summary>Ответ</summary>

**B) `await Future.wait([future1, future2])`**

`Future.wait` запускает все Future одновременно и ждёт завершения всех. `await future1; await future2` — последовательное выполнение (медленнее). `Future.wait` возвращает `Future<List<T>>` с результатами в том же порядке.

</details>

---

### Вопрос 7 🟡

Что делает `Future.delayed`?

- A) Откладывает компиляцию функции
- B) Создаёт Future, который завершается через указанную Duration
- C) Повторяет Future с задержкой
- D) Добавляет timeout к существующему Future

<details>
<summary>Ответ</summary>

**B) Создаёт Future, который завершается через указанную Duration**

`await Future.delayed(Duration(seconds: 2))` — пауза на 2 секунды. Можно также передать callback: `Future.delayed(Duration(ms: 500), () => 'result')`. Не блокирует поток.

</details>

---

### Вопрос 8 🔴

Что произойдёт, если в `Future.wait` один из Future бросит исключение?

- A) Остальные Future отменяются, и общее исключение бросается сразу
- B) Все Future выполняются, затем бросается первое исключение
- C) `Future.wait` завершается с `null` для провальных Future
- D) Ничего — ошибки игнорируются

<details>
<summary>Ответ</summary>

**B) Все Future выполняются, затем бросается первое исключение**

По умолчанию `Future.wait` ждёт все Future, даже если некоторые завершились с ошибкой. Затем бросается первое исключение. Используйте `eagerError: true` для немедленного броска при первой ошибке. Параметр `cleanUp` позволяет освободить ресурсы успешных Future при ошибке.

</details>

---

### Вопрос 9 🔴

Что такое `Completer` в Dart?

- A) Инструмент для завершения приложения
- B) Объект для ручного управления завершением Future
- C) Для объединения нескольких Stream
- D) Менеджер памяти для async операций

<details>
<summary>Ответ</summary>

**B) Объект для ручного управления завершением Future**

```dart
final completer = Completer<String>();
// Передать кому-то: completer.future
// Завершить позже:
completer.complete('result');
// Или с ошибкой:
completer.completeError(Exception('fail'));
```

Используется при интеграции с callback-based API, которое нельзя напрямую `await`.

</details>

---

### Вопрос 10 🔴

Что выведет код?

```dart
Future<int> compute() async {
  return Future.value(42);
}
void main() async {
  final result = await compute();
  print(result.runtimeType);
}
```

- A) `Future<int>`
- B) `int`
- C) `dynamic`
- D) Ошибка компиляции

<details>
<summary>Ответ</summary>

**B) `int`**

`await` разворачивает `Future<T>` до `T`. `compute()` возвращает `Future<int>`, `await compute()` даёт `int`. Если `Future<T>` оборачивает ещё один `Future<T>` — `await` автоматически разворачивает несколько уровней (flattening). Поэтому `return Future.value(42)` в `async` функции эквивалентно `return 42`.

</details>
