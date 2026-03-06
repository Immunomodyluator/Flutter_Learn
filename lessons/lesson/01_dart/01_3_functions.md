# 1.3 Функции и стрелочные функции

## 1. Суть концепции

Относится к: **функции, функциональное программирование, замыкания**.

В Dart функции — **объекты первого класса**: их можно присваивать переменным, передавать как аргументы и возвращать из других функций. Это основа для колбэков в Flutter (`onPressed`, `onChanged`, `builder`).

Стрелочная функция `=>` — синтаксический сахар для функций с одним выражением в теле.

---

## 2. Как используется во Flutter

- **Колбэки событий**: `onPressed: () => doSomething()`
- **Builder-паттерн**: `builder: (context, snapshot) { ... }`
- **Трансформация данных**: `.map((e) => Widget(...)).toList()`
- **Передача логики в виджет**: кнопка получает колбэк и вызывает его — модель отделена от UI
- **Typedef** — именование типов функций для читаемых API

---

## 3. Синтаксис и базовый пример

### Обычная и стрелочная функции

```dart
// Обычная функция
int add(int a, int b) {
  return a + b;
}

// Стрелочная — эквивалент, только для одного выражения
int add(int a, int b) => a + b;

// Возвращаемый тип можно опустить (вывод типа)
double area(double r) => 3.14 * r * r;
```

### Параметры: позиционные, именованные, optional

```dart
// Позиционные (порядок важен)
String greet(String name, int age) => 'Hi, $name! Age: $age';
greet('Ivan', 28);

// Именованные (порядок не важен, по умолчанию optional)
Widget buildButton({required String label, VoidCallback? onTap}) {
  return TextButton(onPressed: onTap, child: Text(label));
}
buildButton(label: 'OK', onTap: () => print('tapped'));

// Значения по умолчанию
String format(String text, {bool uppercase = false}) =>
    uppercase ? text.toUpperCase() : text;

// Optional позиционные (в квадратных скобках)
String hello([String name = 'World']) => 'Hello, $name!';
hello();        // 'Hello, World!'
hello('Dart');  // 'Hello, Dart!'
```

### Функции как объекты

```dart
// Присваивание функции переменной
int Function(int, int) operation = add;
print(operation(3, 4)); // 7

// Передача функции как аргумент
List<int> numbers = [1, 2, 3, 4, 5];
List<int> evens = numbers.where((n) => n.isEven).toList(); // [2, 4]

// Возврат функции из функции
Function multiplier(int factor) => (int x) => x * factor;
var double_ = multiplier(2);
print(double_(5)); // 10
```

### Анонимные функции и замыкания

```dart
// Анонимная функция (лямбда)
var greet = (String name) => 'Hello, $name';
print(greet('Flutter'));

// Замыкание — захватывает переменные из внешней области
int counter = 0;
var increment = () => counter++;
increment();
increment();
print(counter); // 2 — функция "захватила" counter
```

### typedef

```dart
// Именование типа функции
typedef OnItemSelected = void Function(int index, String value);

class MyList extends StatelessWidget {
  final OnItemSelected onSelected; // читабельнее чем void Function(int, String)
  const MyList({super.key, required this.onSelected});
  // ...
}
```

---

## 4. Реальный пример из Flutter

```dart
// Переиспользуемая кнопка с колбэком
class ActionButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed; // VoidCallback = void Function()
  final bool isLoading;

  const ActionButton({
    super.key,
    required this.label,
    required this.onPressed,
    this.isLoading = false,
  });

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      // Стрелочная функция: если загрузка — передаём null (кнопка заблокирована)
      onPressed: isLoading ? null : onPressed,
      child: isLoading
          ? const SizedBox(
              width: 16,
              height: 16,
              child: CircularProgressIndicator(strokeWidth: 2),
            )
          : Text(label),
    );
  }
}

// Использование:
class LoginScreen extends StatelessWidget {
  const LoginScreen({super.key});

  Future<void> _handleLogin() async {
    // логика логина
  }

  @override
  Widget build(BuildContext context) {
    return ActionButton(
      label: 'Войти',
      onPressed: _handleLogin, // передаём метод как объект без ()
    );
  }
}
```

Обрати внимание: `onPressed: _handleLogin` — без скобок. Скобки вызывают функцию немедленно, без скобок — передают как значение.

---

## 5. Что происходит под капотом

### Замыкания и heap

Когда функция захватывает переменную из внешей области (`counter` в примере выше), Dart перемещает эту переменную в **heap** (не в стек). Это позволяет функции обращаться к ней и после того, как внешняя функция завершилась.

### `VoidCallback` и `Function` типы

```dart
VoidCallback         // = void Function()
ValueChanged<T>      // = void Function(T value)
ValueGetter<T>       // = T Function()
ValueSetter<T>       // = void Function(T value)
```

Это `typedef`-псевдонимы из Flutter SDK. Используй их вместо длинных сигнатур.

### Tear-off

```dart
// Вместо: onPressed: () => _handleLogin()
// Можно:
onPressed: _handleLogin // tear-off — создаёт closure без явного лямбды
```

Dart создаёт closure автоматически. Разницы в поведении нет, но tear-off короче.

---

## 6. Типичные ошибки

| Ошибка                                         | Почему возникает                                   | Правильно                                                     |
| ---------------------------------------------- | -------------------------------------------------- | ------------------------------------------------------------- |
| `onPressed: doSomething()`                     | Функция вызывается сразу при построении виджета    | `onPressed: doSomething` или `onPressed: () => doSomething()` |
| Большой блок кода в `onPressed: () { ... }`    | Логика попадает в UI слой                          | Вынести в отдельный метод или сервис                          |
| Не указан тип у функции-параметра              | `Function` слишком широкий тип, теряются подсказки | Указывай точный тип: `VoidCallback`, `ValueChanged<String>`   |
| Замыкание захватывает изменяемый счётчик цикла | В async коде захватывается последнее значение `i`  | Используй `for-in` или создавай копию: `final idx = i;`       |

---

## 7. Практические рекомендации

1. **Стрелка `=>` для однострочников** — стандарт стиля в Flutter-коде, делает `build()` читаемым.
2. **Именованные параметры везде в публичных API виджетов** — `required` делает API самодокументирующимся.
3. **`VoidCallback` / `ValueChanged<T>` вместо `Function`** — типобезопасность и читаемость.
4. **Колбэки с именем `onXxx`** — соглашение Flutter: `onPressed`, `onChanged`, `onTap`, `onSubmitted`.
5. **Не передавай анонимную функцию в `const` виджет** — анонимная функция создаётся заново при каждом `build`, это нарушает `const`.
6. **`typedef` для сложных сигнатур** — если функция принимает более 2 аргументов, именуй её тип.
7. **Tear-off вместо явного лямбды** там, где сигнатуры совпадают:
   ```dart
   items.map(MyWidget.new).toList() // конструктор как функция
   ```
