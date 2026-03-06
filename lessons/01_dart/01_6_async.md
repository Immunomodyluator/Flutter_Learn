# 1.6 Асинхронность: Future и async/await

## 1. Суть концепции

Относится к: **async runtime, Event Loop, конкурентность**.

Dart однопоточный, но умеет неблокирующим образом ждать операций I/O (сеть, файлы, база данных). Механизм — `Future<T>`: объект, который представляет значение, которое **появится в будущем**.

`async/await` — синтаксический сахар над `Future`, делающий асинхронный код читаемым как синхронный. Под капотом это всё тот же Event Loop — не многопоточность.

---

## 2. Как используется во Flutter

- Загрузка данных из API: `Future<List<Product>> fetchProducts()`
- `FutureBuilder` — виджет, который перерисовывается когда Future завершается
- `initState()` не может быть async → запускаем Future отдельным методом
- `Navigator.push()` возвращает `Future<T?>` — можно дождаться результата с экрана
- Работа с базой данных, файлами, SharedPreferences — всё async

---

## 3. Синтаксис и базовый пример

```dart
// Future — обещание значения типа T
Future<String> fetchUsername() async {
  // await — ждёт завершения Future, не блокируя Event Loop
  await Future.delayed(const Duration(seconds: 1)); // имитация запроса
  return 'Ivan'; // тип String, обёрнутый в Future<String> автоматически
}

// Вызов async функции
void main() async {
  print('Start');
  final name = await fetchUsername(); // ждём результата
  print('Hello, $name');             // 'Hello, Ivan'
  print('End');
}
// Выведет: Start → (1 сек) → Hello, Ivan → End
```

### Обработка ошибок

```dart
Future<String> riskyOperation() async {
  throw Exception('Something went wrong');
}

// Через try/catch
Future<void> main() async {
  try {
    final result = await riskyOperation();
    print(result);
  } catch (e) {
    print('Error: $e');
  } finally {
    print('Cleanup');
  }
}

// Через .catchError() (функциональный стиль, реже)
riskyOperation()
    .then((value) => print(value))
    .catchError((e) => print('Error: $e'));
```

### Future.wait — параллельное выполнение

```dart
// Последовательно (медленно): 3 сек
final user = await fetchUser();
final orders = await fetchOrders();

// Параллельно (быстро): 1 сек (максимум из времён)
final results = await Future.wait([
  fetchUser(),    // Future<User>
  fetchOrders(),  // Future<List<Order>>
]);
final user = results[0] as User;
final orders = results[1] as List<Order>;
```

### Состояния Future

```dart
// Future может быть в одном из трёх состояний:
// 1. Pending — ещё выполняется
// 2. Completed with value — успешно завершён
// 3. Completed with error — завершён с ошибкой
```

---

## 4. Реальный пример из Flutter

```dart
class ProductsScreen extends StatefulWidget {
  const ProductsScreen({super.key});

  @override
  State<ProductsScreen> createState() => _ProductsScreenState();
}

class _ProductsScreenState extends State<ProductsScreen> {
  late Future<List<Product>> _productsFuture;

  @override
  void initState() {
    super.initState();
    // initState не может быть async, поэтому сохраняем Future
    _productsFuture = _loadProducts();
  }

  Future<List<Product>> _loadProducts() async {
    try {
      final response = await ApiService.fetchProducts();
      return response;
    } catch (e) {
      // Бросаем дальше — FutureBuilder покажет error state
      rethrow;
    }
  }

  Future<void> _refresh() async {
    setState(() {
      _productsFuture = _loadProducts(); // пересоздаём Future → FutureBuilder перезапустится
    });
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<List<Product>>(
      future: _productsFuture,
      builder: (context, snapshot) {
        // snapshot.connectionState — текущее состояние Future
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Center(child: CircularProgressIndicator());
        }

        // snapshot.hasError — Future завершился ошибкой
        if (snapshot.hasError) {
          return Center(
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                Text('Ошибка: ${snapshot.error}'),
                ElevatedButton(
                  onPressed: _refresh,
                  child: const Text('Повторить'),
                ),
              ],
            ),
          );
        }

        // snapshot.data — результат Future
        final products = snapshot.data!; // безопасно: hasData прошёл
        return RefreshIndicator(
          onRefresh: _refresh,
          child: ListView.builder(
            itemCount: products.length,
            itemBuilder: (context, index) => ProductCard(product: products[index]),
          ),
        );
      },
    );
  }
}
```

---

## 5. Что происходит под капотом

### Event Loop и async/await

```
main isolate:
  1. Запускается синхронный код
  2. await → регистрирует continuation в microtask queue
  3. Event Loop продолжает выполнение других задач
  4. Future завершается → continuation помещается в очередь
  5. Microtask queue обрабатывается → код после await выполняется
```

`await` не создаёт поток. Он говорит Event Loop: "когда Future завершится — продолжи выполнение здесь". UI не блокируется.

### async функции возвращают Future неявно

```dart
Future<int> getValue() async {
  return 42; // Dart оборачивает 42 в Future.value(42) автоматически
}

// Эквивалентно:
Future<int> getValue() {
  return Future.value(42);
}
```

### Microtask vs Event queue

```dart
// Microtask queue — выше приоритет (выполняется между событиями)
scheduleMicrotask(() => print('microtask'));

// Event queue — обычные события (таймеры, I/O)
Future(() => print('event'));

print('sync');
// Вывод: sync → microtask → event
```

---

## 6. Типичные ошибки

| Ошибка                                | Почему возникает                                            | Правильно                                                         |
| ------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------------------------- |
| `setState()` после `dispose()`        | async операция завершилась после закрытия экрана            | Проверять `mounted` перед `setState`                              |
| Не обрабатывать ошибки Future         | Необработанный Future error = crash                         | Всегда `try/catch` или `.catchError()`                            |
| `await` внутри `initState()` напрямую | `initState` не `async`                                      | Вынести в отдельный `async` метод                                 |
| `Future.wait` без обработки ошибок    | Одна ошибка — все результаты теряются                       | `Future.wait(futures, eagerError: false)` или обрабатывать каждый |
| Пересоздавать Future в `build()`      | `FutureBuilder` перезапускает Future при каждой перерисовке | Сохранять Future в `State`, не создавать в `build`                |

### mounted проверка

```dart
Future<void> _loadData() async {
  final data = await fetchData();

  // Виджет мог быть dispose'd пока шёл запрос
  if (!mounted) return; // early return

  setState(() => _data = data);
}
```

---

## 7. Практические рекомендации

1. **Сохраняй Future в State** — никогда не создавай Future прямо в `build()`, это перезапускает его при каждой перерисовке.
2. **Всегда проверяй `mounted`** перед `setState()` в async методах — защита от crash после `dispose()`.
3. **`Future.wait` для параллельных независимых запросов** — существенно ускоряет загрузку.
4. **`unawaited()` для fire-and-forget** — аналитика, логи, которые не нужно ждать:
   ```dart
   unawaited(analytics.logEvent('screen_view'));
   ```
5. **`try/catch/finally` вместо `.catchError()`** — читабельнее, работает со всеми типами исключений.
6. **Timeout для сетевых запросов**:
   ```dart
   await fetchData().timeout(
     const Duration(seconds: 10),
     onTimeout: () => throw TimeoutException('Request timed out'),
   );
   ```
7. **`FutureBuilder` для UI** — не храни `isLoading`, `error`, `data` как отдельные поля — используй `FutureBuilder` или state management.
