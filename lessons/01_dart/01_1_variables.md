# 1.1 Переменные и типы данных

## 1. Суть концепции

Относится к: **типизация, система типов Dart**.

Dart — язык со **статической типизацией** и **sound null safety**. Компилятор знает тип каждой переменной на этапе компиляции, что позволяет ловить ошибки до запуска. При этом `var` позволяет не писать тип явно — компилятор выведет его сам (type inference).

Ключевое отличие от JS/Python: тип переменной зафиксирован после объявления, его нельзя изменить.

---

## 2. Как используется во Flutter

- Каждое поле виджета, состояние, данные из API — всё имеет конкретный тип
- `final` для полей виджетов (данные пришли через конструктор и не меняются)
- `const` для статических данных (цвета, стили, тексты — компилируется в константу)
- `late` для полей, которые инициализируются в `initState()`, а не в конструкторе

---

## 3. Синтаксис и базовый пример

```dart
// Явное указание типа
int count = 0;
double price = 9.99;
String name = 'Flutter';
bool isLoading = false;

// Type inference — тип выводится компилятором
var title = 'Hello'; // String
var items = [1, 2, 3]; // List<int>

// final — значение присваивается один раз, нельзя переприсвоить
final String userId = fetchUserId();
final createdAt = DateTime.now(); // тип выведен как DateTime

// const — значение известно на этапе компиляции
const int maxRetries = 3;
const Color primaryColor = Color(0xFF6200EE);

// late — инициализация отложена, но тип non-nullable
late String userName;
// userName используется позже, после инициализации
```

**`final` vs `const`:**

- `final` — значение вычисляется один раз в runtime (может быть результатом функции)
- `const` — значение известно **на этапе компиляции** (только литералы и другие const)

---

## 4. Реальный пример из Flutter

```dart
class ProductCard extends StatelessWidget {
  // final: данные приходят снаружи, не меняются внутри виджета
  final String title;
  final double price;
  final bool isAvailable;

  const ProductCard({
    super.key,
    required this.title,
    required this.price,
    this.isAvailable = true, // default value
  });

  @override
  Widget build(BuildContext context) {
    // var — локальная переменная, тип выводится
    var displayPrice = price > 0 ? '${price.toStringAsFixed(2)} ₽' : 'Бесплатно';

    return Card(
      child: ListTile(
        title: Text(title),
        subtitle: Text(displayPrice),
        trailing: Icon(
          isAvailable ? Icons.check_circle : Icons.cancel,
          color: isAvailable ? Colors.green : Colors.red,
        ),
      ),
    );
  }
}
```

Все поля виджета `final` — это стандарт Flutter. Виджеты иммутабельны, их данные не должны меняться после создания.

---

## 5. Что происходит под капотом

### Type inference

`var title = 'Hello'` — компилятор видит литерал `String` и фиксирует тип. После этого `title = 42` вызовет ошибку компиляции. Это **не** динамическая типизация.

### `const` и дерево виджетов

```dart
// Эти два объекта — ОДИН и тот же объект в памяти
const text1 = Text('Hello');
const text2 = Text('Hello');
print(identical(text1, text2)); // true
```

Flutter при перестройке дерева виджетов пропускает `const` виджеты — они уже существуют в памяти и не изменились.

### `dynamic` vs `Object?`

- `dynamic` — отключает **все** проверки типов, ошибки только в runtime
- `Object?` — базовый тип для всего, но типобезопасный, требует касты

```dart
dynamic x = 'hello';
x.nonExistentMethod(); // ошибка только в RUNTIME

Object? y = 'hello';
y.nonExistentMethod(); // ошибка компиляции — правильно!
(y as String).toUpperCase(); // нужен явный каст
```

---

## 6. Типичные ошибки

| Ошибка                           | Почему возникает                                 | Правильно                                                 |
| -------------------------------- | ------------------------------------------------ | --------------------------------------------------------- |
| `var x; x = 'hello';`            | Без инициализатора `var` даёт тип `dynamic`      | `String x; x = 'hello';` или `var x = 'hello';`           |
| Использование `dynamic` для JSON | Теряются все подсказки IDE                       | Создавать data-классы с `fromJson`                        |
| `const` объект с mutable полем   | `const` требует чтобы все поля тоже были `const` | Использовать `final` вместо `const`                       |
| Переприсвоение `final`           | `final` — только одно присвоение                 | Использовать обычную переменную                           |
| `late` без инициализации         | `LateInitializationError` в runtime              | Инициализировать в `initState()` до первого использования |

---

## 7. Практические рекомендации

1. **Всегда `final` для полей виджетов** — это стандарт Flutter, помогает компилятору оптимизировать дерево виджетов.
2. **`const` конструкторы для виджетов** — снижает нагрузку на рендеринг. Если все поля `final` и widget `const`, Flutter не пересоздаёт его при перестройке.
3. **Избегай `dynamic`** — единственный легитимный кейс: работа с внешними библиотеками без типов или десериализация сырого JSON (и то временно).
4. **`var` для локальных переменных** — читаемость без потери безопасности, тип всё равно выводится статически.
5. **`late` только как крайний вариант** — предпочти nullable тип (`String?`) или инициализацию значением по умолчанию.
6. **Числа**: `int` для целых, `double` для дробных. В Dart нет `float`, `long`, `short` — только эти два.
7. **`String` всегда с заглавной** — в Dart типы пишутся с большой буквы (`String`, `int`, `bool`, не `string`, `integer`).
