# 7.1 setState и подъём состояния

## 1. Суть концепции

`setState()` — простейший способ управления состоянием в Flutter. Вызов `setState()` помечает виджет как "грязный" (`dirty`) и планирует перестройку в следующем фрейме.

**Подъём состояния (State Lifting)** — паттерн, при котором общее состояние переносится к ближайшему общему предку виджетов, которым оно нужно.

---

## 2. Принцип работы setState

```dart
class CounterPage extends StatefulWidget {
  const CounterPage({super.key});

  @override
  State<CounterPage> createState() => _CounterPageState();
}

class _CounterPageState extends State<CounterPage> {
  int _count = 0;

  void _increment() {
    setState(() {
      // Всё изменение состояния — внутри setState
      _count++;
    });
    // setState помечает виджет как dirty
    // Flutter планирует вызов build() в следующем фрейме
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(child: Text('$_count', style: const TextStyle(fontSize: 48))),
      floatingActionButton: FloatingActionButton(
        onPressed: _increment,
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

---

## 3. Что происходит под капотом

```
setState(() { _count++; })
          ↓
  Dart: выполняет переданный callback (изменяет данные)
          ↓
  Element.markNeedsBuild() — помечает Element как dirty
          ↓
  BuildOwner добавляет в очередь dirty elements
          ↓
  Следующий фрейм: build() вызывается для грязных элементов
          ↓
  Widget tree сравнивается с предыдущим (reconciliation)
          ↓
  RenderObject обновляется только там, где изменился layout/paint
```

---

## 4. Правила setState

```dart
// ✅ Правильно: изменения внутри setState
setState(() {
  _items.add(newItem);
  _isLoading = false;
  _selectedIndex = 2;
});

// ✅ Допустимо: изменить данные ДО setState (для async)
_items = await api.fetch();
if (mounted) setState(() {}); // пустой setState — тоже работает

// ❌ Неправильно: изменить вне setState
_count++; // UI не обновится
setState(() {}); // так можно, но непонятно что изменилось

// ❌ Неправильно: async операция внутри setState
setState(() async {
  _items = await api.fetch(); // игнорируется — setState не ждёт Future
});
```

---

## 5. Подъём состояния (State Lifting)

**Проблема**: два виджета нуждаются в одних данных.

```dart
// ❌ Без подъёма — данные не синхронизированы
class FilterChip extends StatefulWidget { /* своё состояние */ }
class ProductList extends StatefulWidget { /* отдельное состояние */ }

// ✅ С подъёмом — родитель владеет данными, дети читают/уведомляют
class ProductScreen extends StatefulWidget {
  const ProductScreen({super.key});

  @override
  State<ProductScreen> createState() => _ProductScreenState();
}

class _ProductScreenState extends State<ProductScreen> {
  String _selectedCategory = 'Все';
  List<Product> get _filteredProducts =>
      _selectedCategory == 'Все'
          ? products
          : products.where((p) => p.category == _selectedCategory).toList();

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Дочерний виджет получает callback для изменения состояния
        CategoryFilter(
          selected: _selectedCategory,
          onChanged: (category) => setState(() => _selectedCategory = category),
        ),
        // Дочерний виджет получает уже отфильтрованные данные
        Expanded(
          child: ProductList(products: _filteredProducts),
        ),
      ],
    );
  }
}

// Дочерние виджеты — "тупые", без своего состояния
class CategoryFilter extends StatelessWidget {
  final String selected;
  final ValueChanged<String> onChanged;

  const CategoryFilter({super.key, required this.selected, required this.onChanged});

  @override
  Widget build(BuildContext context) {
    return SingleChildScrollView(
      scrollDirection: Axis.horizontal,
      child: Row(
        children: ['Все', 'Электроника', 'Одежда', 'Книги'].map((cat) =>
          Padding(
            padding: const EdgeInsets.only(right: 8),
            child: FilterChip(
              label: Text(cat),
              selected: selected == cat,
              onSelected: (_) => onChanged(cat),
            ),
          ),
        ).toList(),
      ),
    );
  }
}

class ProductList extends StatelessWidget {
  final List<Product> products;
  const ProductList({super.key, required this.products});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: products.length,
      itemBuilder: (_, i) => ProductCard(product: products[i]),
    );
  }
}
```

---

## 6. Callback patterns

```dart
// VoidCallback — без параметров
final VoidCallback onPressed;
ElevatedButton(onPressed: onPressed, ...)

// ValueChanged<T> — с одним параметром
final ValueChanged<String> onChanged;
onChanged('new value');

// Callback с несколькими параметрами
final void Function(String name, int age) onSubmit;
onSubmit('Ivan', 25);
```

---

## 7. Когда setState недостаточно

| Признак                                        | Решение             |
| ---------------------------------------------- | ------------------- |
| Одно состояние нужно в 5+ несвязанных виджетах | Provider / Riverpod |
| "Prop drilling" через 3+ уровней               | Provider / Riverpod |
| Состояние нужно вне дерева (в сервисе)         | Riverpod / Bloc     |
| Сложная логика с событиями                     | Bloc                |

---

## 8. Практические рекомендации

1. **setState достаточен** для локального состояния одного экрана.
2. **Изменяй ВСЕ связанные поля в одном setState** — один rebuild вместо нескольких.
3. **`if (!mounted) return`** перед setState после async операций.
4. **Поднимай состояние** к ближайшему общему предку — не к корню приложения.
5. **Callback через параметры** — не GlobalKey для общения между виджетами.
6. **Не вызывай setState в build()** — бесконечный цикл.
