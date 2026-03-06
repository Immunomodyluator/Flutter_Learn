# 1.2 Коллекции

## 1. Суть концепции

Относится к: **коллекции, обобщённые типы (generics)**.

Dart имеет три встроенных типа коллекций: `List` (упорядоченный список), `Map` (ключ-значение), `Set` (уникальные элементы). Все они **обобщённые** — тип элементов указывается в угловых скобках и проверяется компилятором.

Dart добавляет мощные вещи, которых нет в большинстве языков прямо в синтаксисе: **spread-оператор** и **collection if/for** — позволяют строить коллекции декларативно прямо в UI-коде.

---

## 2. Как используется во Flutter

- `List<Widget>` — список дочерних виджетов в `Column`, `Row`, `Stack`
- `Map<String, dynamic>` — десериализованный JSON
- `Set<String>` — уникальные ID выбранных элементов
- **Collection if/for** применяются прямо в `children: []` чтобы условно добавлять виджеты
- Spread `...` используется для объединения списков виджетов

---

## 3. Синтаксис и базовый пример

### List

```dart
// Изменяемый список с типом
List<String> fruits = ['apple', 'banana', 'cherry'];

// Сокращённая запись через литерал
var numbers = [1, 2, 3, 4, 5]; // List<int>

// Основные операции
fruits.add('mango');          // добавить в конец
fruits.insert(0, 'orange');   // вставить по индексу
fruits.remove('banana');      // удалить по значению
fruits.removeAt(0);           // удалить по индексу
print(fruits.length);         // длина
print(fruits[0]);             // доступ по индексу
print(fruits.contains('apple')); // проверка наличия

// Неизменяемый список
final immutable = List.unmodifiable(['a', 'b', 'c']);
const constList = ['x', 'y', 'z']; // тоже неизменяемый
```

### Map

```dart
// Явный тип
Map<String, int> scores = {'Alice': 95, 'Bob': 87};

// Через var
var user = {'name': 'Ivan', 'age': 28}; // Map<String, Object>

// Операции
scores['Charlie'] = 92;           // добавить/обновить
scores.remove('Bob');              // удалить
print(scores['Alice']);            // 95
print(scores['Unknown']);           // null (не ошибка!)
print(scores.containsKey('Alice')); // true
print(scores.keys);                // ('Alice', 'Charlie')
print(scores.values);              // (95, 92)
print(scores.entries);             // пары ключ-значение

// Безопасный доступ с дефолтом
int score = scores['Unknown'] ?? 0; // оператор ?? — если null, то 0
```

### Set

```dart
var tags = <String>{'flutter', 'dart', 'mobile'};
tags.add('flutter'); // уже есть — не добавится
print(tags.length);  // 3, не 4

// Операции над множествами
var a = {1, 2, 3, 4};
var b = {3, 4, 5, 6};
print(a.intersection(b)); // {3, 4}
print(a.union(b));         // {1, 2, 3, 4, 5, 6}
print(a.difference(b));    // {1, 2}
```

### Spread-оператор `...`

```dart
var first = [1, 2, 3];
var second = [4, 5, 6];
var combined = [...first, ...second]; // [1, 2, 3, 4, 5, 6]

List<int>? nullable;
var safe = [...?nullable, 7]; // ...? — безопасный spread для nullable
```

### Collection if/for

```dart
bool showAdmin = true;
var menuItems = [
  'Home',
  'Profile',
  if (showAdmin) 'Admin panel', // условно добавляем элемент
  'Settings',
];

var numbers = [1, 2, 3];
var doubled = [
  for (var n in numbers) n * 2, // [2, 4, 6]
];
```

---

## 4. Реальный пример из Flutter

```dart
class FilterableList extends StatelessWidget {
  final List<String> items;
  final bool showDividers;
  final String? pinnedItem; // null если не нужен

  const FilterableList({
    super.key,
    required this.items,
    this.showDividers = false,
    this.pinnedItem,
  });

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // collection if: добавляем закреплённый элемент только если он есть
        if (pinnedItem != null) ...[
          _buildItem(pinnedItem!, isPinned: true),
          const Divider(),
        ],

        // collection for: создаём виджеты из списка данных
        for (int i = 0; i < items.length; i++) ...[
          _buildItem(items[i]),
          // collection if внутри for: разделители между элементами, но не после последнего
          if (showDividers && i < items.length - 1) const Divider(),
        ],
      ],
    );
  }

  Widget _buildItem(String text, {bool isPinned = false}) {
    return ListTile(
      title: Text(text),
      leading: isPinned ? const Icon(Icons.push_pin) : null,
    );
  }
}
```

`...[]` (spread пустого/непустого списка) — стандартный паттерн во Flutter для вставки группы виджетов с условием.

---

## 5. Что происходит под капотом

### Generics в runtime

В отличие от Java (type erasure), Dart **сохраняет** generic-тип в runtime:

```dart
var list = <int>[1, 2, 3];
print(list is List<int>);    // true
print(list is List<String>); // false — проверяется в runtime!
```

### Изменяемость и структура памяти

- `List` — динамический массив (аналог `ArrayList` в Java). При переполнении ёмкости создаётся новый буфер с удвоенным размером.
- `Map` в Dart — связная хеш-таблица, **сохраняет порядок вставки** (в отличие от HashMap в Java).
- `Set` — тоже хеш-таблица, порядок не гарантирован.

### const коллекции

```dart
const list = [1, 2, 3]; // объект создаётся один раз при компиляции
// list.add(4); // ошибка: Unsupported operation: Cannot add to an unmodifiable list
```

---

## 6. Типичные ошибки

| Ошибка                                | Почему возникает                 | Правильно                                                 |
| ------------------------------------- | -------------------------------- | --------------------------------------------------------- |
| `map['key']` без проверки на null     | Возвращает `null` если ключа нет | `map['key'] ?? defaultValue` или `map.containsKey('key')` |
| Модификация `const` коллекции         | `const` — неизменяемая           | Использовать `final` вместо `const` если нужна мутация    |
| `List<dynamic>` вместо `List<Widget>` | Теряется типобезопасность        | Всегда указывать тип: `List<Widget>`                      |
| Использование `Set` как `List`        | У `Set` нет доступа по индексу   | Конвертировать: `set.toList()[index]`                     |
| Изменение списка во время итерации    | `ConcurrentModificationError`    | Итерироваться по копии: `list.toList().forEach(...)`      |

---

## 7. Практические рекомендации

1. **Collection if/for вместо `..addAll()`** — декларативный подход читабельнее, особенно в `children:` виджетов.
2. **`List.generate(n, (i) => ...)` для виджетов по количеству** — вместо `for` цикла императивного стиля.
3. **Не мутируй коллекции, пришедшие извне** — создавай копию через `[...original]` или `List.from(original)`.
4. **`where`, `map`, `expand`, `fold`** — методы коллекций в функциональном стиле, используются повсеместно:
   ```dart
   final visible = items.where((e) => e.isActive).toList();
   final titles = items.map((e) => e.title).toList();
   ```
5. **`const []`** для пустых коллекций-значений по умолчанию\*\* — экономит память:
   ```dart
   const ProductCard({this.tags = const []});
   ```
6. **`Map.fromIterables`** для создания Map из двух списков.
7. **`Set` для проверки принадлежности** — `set.contains(x)` работает за O(1), а `list.contains(x)` — за O(n).
