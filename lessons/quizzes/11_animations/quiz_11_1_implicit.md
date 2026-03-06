# Квиз: Неявные анимации

**Тема:** 11.1 — Implicit Animations  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое неявные анимации и как они работают?

- A) Анимации с явным `AnimationController`
- B) Виджеты автоматически анимируют переход между значениями при изменении `setState` — не нужен `AnimationController`
- C) Анимации с использованием Rive или Lottie
- D) Неявные = нет анимации, мгновенное изменение

<details>
<summary>Ответ</summary>

**B) Автоматический переход между значениями — не нужен AnimationController**

```dart
class _MyState extends State<MyWidget> {
  double _width = 100;

  @override
  Widget build(BuildContext ctx) => Column(
    children: [
      AnimatedContainer(
        duration: const Duration(milliseconds: 300),
        curve: Curves.easeInOut,
        width: _width, // плавно меняется при setState
        height: 100,
        color: Colors.blue,
      ),
      ElevatedButton(
        onPressed: () => setState(() => _width = _width == 100 ? 200 : 100),
        child: Text('Animate'),
      ),
    ],
  );
}
```

</details>

---

### Вопрос 2 🟢

Какой виджет анимирует появление/исчезновение через `opacity`?

- A) `FadeWidget`
- B) `AnimatedOpacity` — анимирует `opacity` от 0.0 до 1.0
- C) `AnimatedContainer(opacity: ...)`
- D) `Visibility(animate: true, ...)`

<details>
<summary>Ответ</summary>

**B) `AnimatedOpacity`**

```dart
AnimatedOpacity(
  opacity: _isVisible ? 1.0 : 0.0,
  duration: const Duration(milliseconds: 400),
  curve: Curves.easeInOut,
  onEnd: () => print('Animation complete'),
  child: const Text('Hello!'),
)
```

Виджет остаётся в дереве даже при `opacity: 0` (в отличие от `Visibility`). Для заменяемых виджетов используйте `AnimatedSwitcher`.

</details>

---

### Вопрос 3 🟢

Как `AnimatedSwitcher` анимирует замену дочернего виджета?

- A) Работает как `AnimatedContainer` — плавно меняет размер
- B) При изменении дочернего виджета (изменился `key` или тип) — анимирует выход старого и вход нового
- C) `AnimatedSwitcher` = `IndexedStack` с анимацией
- D) Только для `Text` виджетов

<details>
<summary>Ответ</summary>

**B) Анимирует выход старого и вход нового при смене `key` или типа**

```dart
AnimatedSwitcher(
  duration: const Duration(milliseconds: 300),
  transitionBuilder: (child, animation) => FadeTransition(
    opacity: animation,
    child: child,
  ),
  child: Text(
    '$_count',
    key: ValueKey<int>(_count), // ВАЖНО: разные ключи = триггер анимации
    style: const TextStyle(fontSize: 40),
  ),
)
```

Без `key` — если тип не меняется, анимация не срабатывает!

</details>

---

### Вопрос 4 🟡

Как использовать `TweenAnimationBuilder` для кастомной неявной анимации?

- A) Нельзя — для кастомных нужен `AnimationController`
- B) `TweenAnimationBuilder<T>(tween: Tween(begin: from, end: to), builder: (ctx, value, child) => ...)` — анимирует любой тип с тween
- C) `CustomAnimationBuilder(from: val1, to: val2)`
- D) Только для `double` значений

<details>
<summary>Ответ</summary>

**B) `TweenAnimationBuilder<T>` — неявная анимация для произвольных типов**

```dart
// Анимация цвета:
TweenAnimationBuilder<Color?>(
  tween: ColorTween(begin: Colors.blue, end: Colors.red),
  duration: const Duration(seconds: 1),
  builder: (ctx, color, child) => Container(
    width: 100, height: 100,
    color: color,
    child: child, // статичный child кэшируется — оптимизация
  ),
  child: const Icon(Icons.star), // не пересобирается при анимации
)

// Анимация Matrix4:
TweenAnimationBuilder<double>(
  tween: Tween(begin: 0.0, end: pi / 4),
  duration: const Duration(milliseconds: 500),
  builder: (ctx, angle, _) => Transform.rotate(angle: angle, child: Icon(Icons.settings)),
)
```

</details>

---

### Вопрос 5 🟡

Чем `AnimatedAlign` отличается от `AnimatedPositioned`?

- A) Они идентичны
- B) `AnimatedAlign` анимирует `Alignment` внутри обычного виджета; `AnimatedPositioned` — только дочерний элемент `Stack`, анимирует `top/left/right/bottom`
- C) `AnimatedAlign` — только для текста
- D) `AnimatedPositioned` требует `AnimationController`

<details>
<summary>Ответ</summary>

**B) `AnimatedAlign` → внутри обычного контейнера; `AnimatedPositioned` → внутри Stack**

```dart
// AnimatedAlign — любой виджет:
AnimatedAlign(
  alignment: _aligned ? Alignment.topLeft : Alignment.bottomRight,
  duration: const Duration(milliseconds: 500),
  child: const FlutterLogo(size: 50),
)

// AnimatedPositioned — только в Stack:
Stack(children: [
  AnimatedPositioned(
    left: _expanded ? 0 : 200,
    top: 50,
    duration: const Duration(milliseconds: 300),
    child: const FlutterLogo(size: 50),
  ),
])
```

</details>

---

### Вопрос 6 🟡

Как реализовать анимацию появления списка элементов (staggered animation) без `AnimationController`?

- A) Невозможно без explicit animations
- B) `TweenAnimationBuilder` с `delay` через `Future.delayed` + `AnimatedOpacity` для каждого элемента
- C) `AnimatedList` + `insertItem()`
- D) `ListView.builder` автоматически анимирует элементы

<details>
<summary>Ответ</summary>

**C) `AnimatedList` — для динамического добавления/удаления с анимацией**

```dart
// Staggered через задержку:
class _StaggeredList extends StatefulWidget { ... }

class _State extends State<_StaggeredList> {
  final _key = GlobalKey<AnimatedListState>();
  final _items = <String>[];

  void _addItem(String item) {
    _items.add(item);
    _key.currentState?.insertItem(_items.length - 1);
  }

  @override
  Widget build(ctx) => AnimatedList(
    key: _key,
    initialItemCount: _items.length,
    itemBuilder: (ctx, index, animation) => FadeTransition(
      opacity: animation,
      child: SizeTransition(
        sizeFactor: animation,
        child: ListTile(title: Text(_items[index])),
      ),
    ),
  );
}
```

</details>

---

### Вопрос 7 🟡

Что делает `AnimatedDefaultTextStyle` и когда его использовать?

- A) Анимирует fontFamily текста
- B) Анимирует изменения `TextStyle` (size, color, weight) для всех дочерних `Text` виджетов
- C) Только для `fontSize` анимации
- D) Эквивалент `AnimatedContainer` для текста

<details>
<summary>Ответ</summary>

**B) Анимирует `TextStyle` для всех дочерних `Text` виджетов**

```dart
AnimatedDefaultTextStyle(
  style: _isLarge
    ? const TextStyle(fontSize: 32, color: Colors.blue, fontWeight: FontWeight.bold)
    : const TextStyle(fontSize: 16, color: Colors.black, fontWeight: FontWeight.normal),
  duration: const Duration(milliseconds: 300),
  curve: Curves.elasticOut,
  child: Column(children: [
    const Text('Header'), // оба анимируются:
    const Text('Subtitle'),
  ]),
)
```

</details>

---

### Вопрос 8 🔴

Как обработать `AnimatedContainer` когда нужно анимировать `BoxDecoration` с несколькими свойствами?

- A) `AnimatedContainer` не поддерживает `decoration`
- B) `AnimatedContainer` поддерживает `decoration` — Flutter интерполирует `BoxDecoration` включая цвет, borderRadius, boxShadow; НО нельзя одновременно задавать `color` и `decoration`
- C) Использовать `TweenAnimationBuilder<BoxDecoration>`
- D) Каждое свойство декорации анимировать отдельно

<details>
<summary>Ответ</summary>

**B) `AnimatedContainer` поддерживает `decoration` — нельзя комбинировать с `color`**

```dart
AnimatedContainer(
  duration: const Duration(milliseconds: 400),
  curve: Curves.easeInOutCubic,
  width: _selected ? 200 : 100,
  height: _selected ? 200 : 100,
  decoration: BoxDecoration(
    // НЕЛЬЗЯ: и color и decoration вместе!
    // color: Colors.blue, ← убрать
    color: _selected ? Colors.purple : Colors.blue, // здесь OK - внутри decoration
    borderRadius: BorderRadius.circular(_selected ? 40 : 8),
    boxShadow: _selected ? [
      BoxShadow(color: Colors.black26, blurRadius: 20, spreadRadius: 5),
    ] : [],
  ),
  child: const Icon(Icons.star, color: Colors.white),
)
```

</details>

---

### Вопрос 9 🔴

Как создать собственный `ImplicitlyAnimatedWidget`?

- A) Нельзя — только предустановленные Animated\* виджеты
- B) Наследовать `ImplicitlyAnimatedWidget` + `AnimatedWidgetBaseState` с переопределением `forEachTween()`
- C) Использовать `StatefulWidget` + `AnimationController`
- D) `AnimatedBuilder` + `setState`

<details>
<summary>Ответ</summary>

**B) `ImplicitlyAnimatedWidget` + `AnimatedWidgetBaseState.forEachTween()`**

```dart
class AnimatedScale extends ImplicitlyAnimatedWidget {
  final double scale;
  final Widget child;

  const AnimatedScale({
    required this.scale,
    required this.child,
    required Duration duration,
    Curve curve = Curves.linear,
    super.key,
  }) : super(duration: duration, curve: curve);

  @override
  AnimatedWidgetBaseState<AnimatedScale> createState() => _AnimatedScaleState();
}

class _AnimatedScaleState extends AnimatedWidgetBaseState<AnimatedScale> {
  Tween<double>? _scale;

  @override
  void forEachTween(TweenVisitor<dynamic> visitor) {
    _scale = visitor(_scale, widget.scale,
        (v) => Tween<double>(begin: v as double)) as Tween<double>?;
  }

  @override
  Widget build(BuildContext ctx) => Transform.scale(
    scale: _scale!.evaluate(animation),
    child: widget.child,
  );
}
```

</details>

---

### Вопрос 10 🔴

Как отменить текущую анимацию `AnimatedSwitcher` и мгновенно показать новый виджет?

- A) `AnimatedSwitcher` нельзя переключить без анимации
- B) Установить `duration: Duration.zero` или использовать `key` с уникальным значением и `reverseDuration: Duration.zero`
- C) `controller.skip()`
- D) Оборнуть в `IgnorePointer`

<details>
<summary>Ответ</summary>

**B) `duration: Duration.zero` — мгновенная смена; или условный Duration**

```dart
AnimatedSwitcher(
  duration: _skipAnimation ? Duration.zero : const Duration(milliseconds: 300),
  child: SomeWidget(key: ValueKey(_currentId)),
)

// Другой подход — условный виджет без AnimatedSwitcher:
if (_skipAnimation)
  SomeWidget(key: ValueKey(_currentId))
else
  AnimatedSwitcher(
    duration: const Duration(milliseconds: 300),
    child: SomeWidget(key: ValueKey(_currentId)),
  )
```

Практика: хранить `_skipAnimation = true` при первом показе данных (loading → content без анимации).

</details>
