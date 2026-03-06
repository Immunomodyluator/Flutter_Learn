# 15.2 Оптимизация ребилдов

## 1. Суть

Каждый раз, когда `setState`, `notifyListeners` или `ref.watch` вызывает rebuild — Flutter перестраивает поддерево виджетов. Лишние ребилды = лишние миллисекунды = джанки.

**Цель**: перестраивать только виджеты, данные которых реально изменились.

---

## 2. const-виджеты

Самый простой способ — `const`. Если виджет помечен `const`, Flutter не создаёт новый экземпляр при rebuild родителя.

```dart
// ❌ Пересоздаётся при каждом rebuild родителя
child: Icon(Icons.star, color: Colors.yellow),

// ✅ Один экземпляр, никогда не пересоздаётся
child: const Icon(Icons.star, color: Colors.yellow),

// ❌ Весь Column пересоздаётся
Column(children: [Text('Title'), Spacer(), Text('Subtitle')])

// ✅ Text и Spacer — const
Column(children: [
  const Text('Title'),
  const Spacer(),
  const Text('Subtitle'),
])
```

**Правило**: добавляй `const` ко всем виджетам, которые не получают изменяемые данные.

---

## 3. Локальный setState

`setState` на корне → весь экран перестраивается.

```dart
// ❌ Состояние счётчика в корне — весь экран ребилдится
class HomeScreen extends StatefulWidget { ... }
class _HomeScreenState extends State<HomeScreen> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(children: [
      const HeavyWidget(),  // перестраивается тоже!
      Text('$_count'),
      ElevatedButton(onPressed: () => setState(() => _count++), child: ...),
    ]);
  }
}

// ✅ Выделить в отдельный StatefulWidget
class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});
  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;

  @override
  Widget build(BuildContext context) {
    return Row(children: [
      Text('$_count'),
      ElevatedButton(
        onPressed: () => setState(() => _count++),
        child: const Text('+'),
      ),
    ]);
  }
}

class HomeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(children: [
      const HeavyWidget(), // больше не перестраивается при смене счётчика
      const CounterWidget(),
    ]);
  }
}
```

---

## 4. Selector (Provider)

```dart
// ❌ Весь виджет перестраивается при любом изменении CartViewModel
class CartTotal extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final vm = context.watch<CartViewModel>(); // слушает ВСЁ
    return Text('Итого: ${vm.total}');
  }
}

// ✅ Перестраивается только когда total изменился
class CartTotal extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final total = context.select<CartViewModel, double>((vm) => vm.total);
    return Text('Итого: $total');
  }
}
```

---

## 5. Consumer (Provider) — сужение rebuild-зоны

```dart
// ❌ Весь большой виджет перестраивается
class ProductScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final vm = context.watch<ProductViewModel>();
    return Scaffold(
      appBar: AppBar(title: const Text('Товар')),   // лишний rebuild
      body: Column(children: [
        const ProductImages(),   // лишний rebuild (статичный контент)
        Text(vm.price),          // нужен только он
      ]),
    );
  }
}

// ✅ Consumer оборачивает только изменяемую часть
class ProductScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: const AppBar(title: Text('Товар')),  // никогда не ребилдится
      body: Column(children: [
        const ProductImages(),                        // никогда не ребилдится
        Consumer<ProductViewModel>(
          builder: (context, vm, child) => Text(vm.price),
        ),
      ]),
    );
  }
}
```

---

## 6. RepaintBoundary

Создаёт отдельный raster-слой. Flutter не перерисовывает этот слой, если виджет внутри не изменился.

```dart
// Хорошо для: бесконечных списков, сложных карточек, изображений
ListView.builder(
  itemBuilder: (context, index) => RepaintBoundary(
    child: ProductCard(item: items[index]),
  ),
)

// AnimatedWidget — не добавляй RepaintBoundary, он сам управляет перерисовкой
// Scroll position изменяется постоянно — RepaintBoundary внутри ListView
// автоматически, не нужно добавлять вручную
```

---

## 7. Riverpod — точечные подписки

```dart
// ❌ Весь экран перестраивается при изменении любого поля UserState
final userState = ref.watch(userProvider);

// ✅ Подписка только на имя
final userName = ref.watch(userProvider.select((s) => s.name));

// ✅ Подписка на наличие ошибки
final hasError = ref.watch(userProvider.select((s) => s.error != null));
```

---

## 8. ListView.builder vs ListView

```dart
// ❌ Создаёт все виджеты сразу (даже если их 1000)
ListView(
  children: items.map((e) => ItemTile(item: e)).toList(),
)

// ✅ Создаёт только видимые виджеты
ListView.builder(
  itemCount: items.length,
  itemBuilder: (context, index) => ItemTile(item: items[index]),
)

// ✅ Для разнородных списков
ListView.builder(
  itemCount: items.length,
  itemExtent: 72, // оптимизация: известна высота каждого элемента
  itemBuilder: (context, index) => ItemTile(item: items[index]),
)
```

---

## 9. Измерение ребилдов

```dart
// Включить счётчик в DevTools Profile
debugProfileBuildsEnabled = true; // подробно (медленно)

// Или быстрый способ: логировать в конструкторе
class MyWidget extends StatelessWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context) {
    if (kDebugMode) print('MyWidget rebuild');
    return const Text('Data');
  }
}
```

---

## 10. Типичные ошибки

| Ошибка                        | Симптом                                   | Решение                                   |
| ----------------------------- | ----------------------------------------- | ----------------------------------------- |
| `context.watch` в корне       | Весь экран ребилдится при любом изменении | `context.select` или `Consumer`           |
| Лямбды в `onPressed`          | Новый объект при каждом build             | Вынести в метод (незначительно, но стоит) |
| `ListView` для 100+ элементов | Тормозит прокрутка                        | `ListView.builder` + `itemExtent`         |
| `RepaintBoundary` везде       | Лишнее потребление памяти                 | Только для часто изменяемых секций        |

---

## 11. Рекомендации

1. **`const` — первое и самое важное** оружие против ребилдов.
2. **Локализуй состояние** — `StatefulWidget` как можно меньше и ближе к нужному месту.
3. **`context.select` / `Selector`** — подписывайся только на нужное поле.
4. **`ListView.builder`** для любых списков длиннее ~20 элементов.
5. **Измеряй DevTools** перед оптимизацией — не оптимизируй то, что не тормозит.
