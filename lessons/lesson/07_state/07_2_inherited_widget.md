# 7.2 InheritedWidget

## 1. Суть концепции

`InheritedWidget` — встроенный механизм Flutter для передачи данных вниз по дереву виджетов без явной передачи через конструкторы. Это основа Provider, Theme, MediaQuery, Navigator и других системных виджетов.

Понимание InheritedWidget помогает понять, как работает Provider под капотом.

---

## 2. Как работает

```
InheritedWidget (хранит данные)
    ├── Widget A
    │     └── Widget B
    │           └── Widget C ← вызывает Context.dependOnInheritedWidgetOfExactType<T>()
    └── Widget D
```

Виджет C получает данные без передачи через A и B. При изменении InheritedWidget перестраиваются только зависимые виджеты (те, кто вызвал `dependOn...`).

---

## 3. Создание InheritedWidget

```dart
// 1. Создать InheritedWidget
class CartInherited extends InheritedWidget {
  final List<Product> items;
  final VoidCallback onUpdate;

  const CartInherited({
    super.key,
    required this.items,
    required this.onUpdate,
    required super.child,
  });

  // Метод доступа для удобства
  static CartInherited of(BuildContext context) {
    final result = context.dependOnInheritedWidgetOfExactType<CartInherited>();
    assert(result != null, 'CartInherited не найден в дереве');
    return result!;
  }

  // Когда пересобирать зависимые виджеты?
  @override
  bool updateShouldNotify(CartInherited oldWidget) {
    return items != oldWidget.items; // пересобрать если items изменились
  }
}

// 2. Обёртка с состоянием (т.к. InheritedWidget сам неизменяем)
class CartProvider extends StatefulWidget {
  final Widget child;
  const CartProvider({super.key, required this.child});

  @override
  State<CartProvider> createState() => _CartProviderState();
}

class _CartProviderState extends State<CartProvider> {
  final List<Product> _items = [];

  void _update() => setState(() {});

  @override
  Widget build(BuildContext context) {
    return CartInherited(
      items: _items,
      onUpdate: _update,
      child: widget.child,
    );
  }
}

// 3. Размещение в дереве
void main() => runApp(
  CartProvider(
    child: MaterialApp(home: const HomeScreen()),
  ),
);

// 4. Получение данных в любом дочернем виджете
class CartIcon extends StatelessWidget {
  const CartIcon({super.key});

  @override
  Widget build(BuildContext context) {
    final cart = CartInherited.of(context); // подписывается на изменения
    return Badge(
      label: Text('${cart.items.length}'),
      child: const Icon(Icons.shopping_cart),
    );
  }
}
```

---

## 4. dependOn vs getElementFor

```dart
// dependOnInheritedWidgetOfExactType — ПОДПИСЫВАЕТСЯ на изменения
// Виджет будет перестраиваться при updateShouldNotify == true
final theme = context.dependOnInheritedWidgetOfExactType<CartInherited>();

// getInheritedWidgetOfExactType — только читает, НЕ подписывается
// Не вызывает rebuild при изменении
final theme = context.getInheritedWidgetOfExactType<CartInherited>();

// Используй второй вариант если виджет не должен перестраиваться при изменении
// (например, только записывает данные, не отображает)
```

---

## 5. updateShouldNotify — контроль перестроек

```dart
@override
bool updateShouldNotify(CartInherited oldWidget) {
  // true = все зависимые виджеты перестроятся
  // false = не перестраивать даже если InheritedWidget пересоздан

  // По значению
  return items.length != oldWidget.items.length;

  // По идентичности
  return !identical(items, oldWidget.items);

  // Всегда перестраивать
  return true;
}
```

---

## 6. Реальные примеры InheritedWidget во Flutter

```dart
// Все они используют InheritedWidget под капотом:
Theme.of(context)           // → _InheritedTheme
MediaQuery.of(context)      // → _MediaQueryFromView
Navigator.of(context)       // → _NavigatorObservation
DefaultTextStyle.of(context) // → DefaultTextStyle
Scaffold.of(context)        // → _ScaffoldScope
Form.of(context)            // → _FormScope
```

---

## 7. Почему Provider лучше InheritedWidget напрямую

|                      | InheritedWidget              | Provider                  |
| -------------------- | ---------------------------- | ------------------------- |
| Изменяемое состояние | Нужна обёртка StatefulWidget | Встроено (ChangeNotifier) |
| Множество типов      | Вкладывать друг в друга      | `MultiProvider`           |
| Lazy-загрузка        | Нет                          | `lazy: true`              |
| `dispose`            | Вручную                      | Автоматически             |
| Читаемость           | Громоздко                    | Компактно                 |

---

## 8. Практические рекомендации

1. **Не пиши InheritedWidget вручную** в продакшн-коде — используй Provider или Riverpod.
2. **Изучи InheritedWidget** для понимания того, как работает весь Flutter state management.
3. **`context.dependOnInheritedWidgetOfExactType`** — для подписки, **`getInheritedWidgetOfExactType`** — только для чтения.
4. **`updateShouldNotify`** — оптимизируй, возвращай `false` если данные не изменились.
5. **InheritedWidget уместен** для создания собственных SDK / библиотек виджетов.
