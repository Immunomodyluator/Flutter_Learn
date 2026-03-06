# Квиз: setState и InheritedWidget

**Тема:** 07.1 — setState  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что делает `setState(() { ... })`?

- A) Сохраняет состояние виджета в базе данных
- B) Помечает виджет как "грязный" и планирует перестройку в конце текущего кадра
- C) Немедленно перестраивает весь экран
- D) Создаёт новый экземпляр State

<details>
<summary>Ответ</summary>

**B) Помечает виджет как "грязный" и планирует перестройку в конце текущего кадра**

`setState` не вызывает `build()` синхронно — Flutter пакетирует обновления и вызывает `build` в следующем кадре. Изменения внутри замыкания `setState(() {...})` захватываются, хотя можно изменить переменные и вне — стрелочка нужна для читаемости и явности намерения.

</details>

---

### Вопрос 2 🟢

Когда использовать `setState`, а когда — внешний state management?

- A) `setState` — всегда, внешний SM — только для серверных данных
- B) `setState` — для локального UI state одного виджета; SM (Provider/Riverpod/BLoC) — для данных которые нужны нескольким виджетам или экранам
- C) `setState` устарел — всегда использовать Provider
- D) Нет разницы — выбор зависит от личных предпочтений

<details>
<summary>Ответ</summary>

**B) `setState` — для локального UI state; SM — для разделяемых данных**

Примеры для `setState`: состояние флага "показать пароль", выбранный индекс в `ToggleButton`, анимация раскрытия карточки. Для SM: данные пользователя, корзина покупок, темп тема, аутентификация.

</details>

---

### Вопрос 3 🟢

Что произойдёт если вызвать `setState` после `dispose()`?

- A) Ничего — setState игнорируется
- B) `FlutterError: setState() called after dispose()` — будет ошибка в debug режиме
- C) Автоматически создаётся новый State
- D) Исключение не выбрасывается, но перестройка не происходит

<details>
<summary>Ответ</summary>

**B) `FlutterError: setState() called after dispose()`**

Распространённая ошибка при асинхронных операциях. Решение: проверять `if (mounted) setState(...)`. `mounted` — геттер `State` возвращающий `true` пока виджет в дереве. В Flutter 3.x lint предупреждает о `use_build_context_synchronously`.

</details>

---

### Вопрос 4 🟡

Что такое `InheritedWidget` и для чего используется?

- A) Виджет унаследованный от другого виджета
- B) Механизм передачи данных вниз по дереву без явной передачи через конструкторы (prop drilling)
- C) Базовый класс для всех виджетов с состоянием
- D) Виджет для наследования темы

<details>
<summary>Ответ</summary>

**B) Механизм передачи данных вниз по дереву без prop drilling**

`InheritedWidget` — основа для `Theme`, `MediaQuery`, `Navigator`. Дочерние виджеты получают данные через `MyInheritedWidget.of(context)`. Автоматически перестраивает зависимые виджеты при изменении. Provider по сути — обёртка над `InheritedWidget`.

</details>

---

### Вопрос 5 🟡

Как реализовать `InheritedWidget` для счётчика?

- A) Сделать класс `extends InheritedWidget`, реализовать `updateShouldNotify`, статический метод `of(context)`
- B) `InheritedWidget(value: counter, child: ...)`
- C) `InheritedNotifier(notifier: counterNotifier, child: ...)`
- D) Напрямую нельзя — нужен `InheritedModel`

<details>
<summary>Ответ</summary>

**A) Extend `InheritedWidget`, реализовать `updateShouldNotify` и статический `of`**

```dart
class CounterInherited extends InheritedWidget {
  final int count;
  const CounterInherited({required this.count, required super.child});

  static CounterInherited of(BuildContext ctx) =>
      ctx.dependOnInheritedWidgetOfExactType<CounterInherited>()!;

  @override
  bool updateShouldNotify(CounterInherited old) => count != old.count;
}
```

</details>

---

### Вопрос 6 🟡

Что такое `InheritedNotifier`?

- A) Аналог `InheritedWidget` для `ChangeNotifier`
- B) `InheritedWidget` специализированный для `Listenable` — автоматически перестраивает потомков при уведомлении
- C) Базовый класс для Provider
- D) Нотификатор для системных событий

<details>
<summary>Ответ</summary>

**B) `InheritedWidget` для `Listenable` — автоматически перестраивает при уведомлении**

```dart
class CounterNotifier extends ChangeNotifier {
  int _count = 0;
  int get count => _count;
  void increment() { _count++; notifyListeners(); }
}

class CounterWidget extends InheritedNotifier<CounterNotifier> {
  const CounterWidget({required CounterNotifier notifier, required super.child})
    : super(notifier: notifier);

  static CounterNotifier of(BuildContext ctx) =>
    ctx.dependOnInheritedWidgetOfExactType<CounterWidget>()!.notifier!;
}
```

</details>

---

### Вопрос 7 🟡

Чем `context.dependOnInheritedWidgetOfExactType` отличается от `context.getInheritedWidgetOfExactType`?

- A) Они идентичны
- B) `dependOn...` — подписывает виджет на обновления (rebuild при изменении); `getInherited...` — только читает без подписки
- C) `getInherited...` — асинхронный вариант
- D) `dependOn...` только для `InheritedModel`

<details>
<summary>Ответ</summary>

**B) `dependOn...` подписывает на обновления; `getInherited...` только читает**

`dependOnInheritedWidgetOfExactType` — O(1), регистрирует зависимость → rebuild. Использовать в `build()`. `getInheritedWidgetOfExactType` — без регистрации, данные не обновятся. Использовать в `initState()` или когда перестройка не нужна (редко).

</details>

---

### Вопрос 8 🔴

Как избежать лишних перестроек при использовании `InheritedWidget` с большим объектом?

- A) `InheritedWidget` не вызывает лишних перестроек
- B) `InheritedModel` — позволяет подписываться на отдельные аспекты (поля) данных
- C) Разделить данные на несколько маленьких `InheritedWidget`
- D) B и C оба являются правильными подходами

<details>
<summary>Ответ</summary>

**D) B и C оба являются правильными подходами**

`InheritedModel` позволяет: `context.dependOnInheritedWidgetOfExactType<MyModel>(aspect: 'counter')` — перестраивается только при изменении `counter` аспекта. Разделение на множество маленьких `InheritedWidget` — паттерн Provider под капотом (отдельный Provider для каждого куска данных).

</details>

---

### Вопрос 9 🔴

Что такое `BuildContext` с точки зрения `InheritedWidget`?

- A) Просто ссылка на виджет
- B) Handle на позицию в дереве виджетов — `Element` — через который происходит поиск `InheritedWidget` в родительской цепочке
- C) Контейнер для передачи данных
- D) Идентификатор виджета

<details>
<summary>Ответ</summary>

**B) `BuildContext` — это `Element` — позиция в дереве, через которую ищутся `InheritedWidget`**

`context.dependOnInheritedWidgetOfExactType<T>()` поднимается по дереву `Element`-ов вверх до нахождения `T`. Поэтому `InheritedWidget` должен быть **предком** потребителя. Если `InheritedWidget` ниже в дереве — `of(context)` вернёт null.

</details>

---

### Вопрос 10 🔴

Почему `setState` внутри `build()` — антипаттерн?

- A) Это вызовет бесконечный цикл перестроек
- B) `setState` в `build()` — ошибка компиляции
- C) Это работает корректно
- D) `build()` вызывается только один раз

<details>
<summary>Ответ</summary>

**A) Это вызовет бесконечный цикл перестроек**

`build()` → `setState()` → `build()` → `setState()` → ... до `StackOverflow`. Flutter выбросит: `setState() or markNeedsBuild() called during build`. Изменение состояния в `build` допустимо только в `post-frame callback`: `WidgetsBinding.addPostFrameCallback((_) => setState(...))`.

</details>
