# Квиз: Явные анимации

**Тема:** 11.2 — Explicit Animations  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что нужно для использования `AnimationController`?

- A) Только `StatefulWidget`
- B) `StatefulWidget` + `SingleTickerProviderStateMixin` (или `TickerProviderStateMixin`) + обязательный `dispose()`
- C) Только `ConsumerWidget` с Riverpod
- D) `AnimationController` работает в любом виджете

<details>
<summary>Ответ</summary>

**B) `StatefulWidget` + `SingleTickerProviderStateMixin` + `dispose()`**

```dart
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this, // обязательно — предотвращает утечки памяти
      duration: const Duration(milliseconds: 500),
    );
  }

  @override
  void dispose() {
    _controller.dispose(); // ОБЯЗАТЕЛЬНО
    super.dispose();
  }
}
```

`TickerProviderStateMixin` — для нескольких контроллеров.

</details>

---

### Вопрос 2 🟢

Что делают методы `forward()`, `reverse()`, `repeat()`?

- A) `forward` — ускорить; `reverse` — замедлить; `repeat` — зациклить
- B) `forward()` — анимировать 0→1; `reverse()` — 1→0; `repeat()` — зациклить
- C) Управляют только `opacity`
- D) Всё одно и то же

<details>
<summary>Ответ</summary>

**B) `forward` 0→1, `reverse` 1→0, `repeat` — цикл**

```dart
// Запустить вперёд:
_controller.forward();

// В обратную сторону:
_controller.reverse();

// Сбросить в 0:
_controller.reset();

// Зациклить (ping-pong):
_controller.repeat(reverse: true);

// Конкретное значение:
_controller.animateTo(0.5, duration: const Duration(seconds: 1));

// Слушать статус:
_controller.addStatusListener((status) {
  if (status == AnimationStatus.completed) _controller.reverse();
});
```

</details>

---

### Вопрос 3 🟢

Как создать анимацию с `Tween` и `CurvedAnimation`?

- A) `Tween(begin: 0, end: 100).animate(_controller)`
- B) `Tween<double>(begin: 0, end: 100).animate(CurvedAnimation(parent: _controller, curve: Curves.easeInOut))`
- C) `_controller.tween(0, 100, curve: Curves.easeIn)`
- D) `AnimatedTween(0, 100, controller: _controller)`

<details>
<summary>Ответ</summary>

**B) `Tween.animate(CurvedAnimation(...))`**

```dart
late Animation<double> _sizeAnimation;
late Animation<Color?> _colorAnimation;

@override
void initState() {
  super.initState();
  _controller = AnimationController(vsync: this, duration: const Duration(ms: 600));

  final curved = CurvedAnimation(parent: _controller, curve: Curves.elasticOut);

  _sizeAnimation = Tween<double>(begin: 50, end: 200).animate(curved);
  _colorAnimation = ColorTween(begin: Colors.blue, end: Colors.red).animate(curved);
}
```

</details>

---

### Вопрос 4 🟡

Чем `AnimatedBuilder` отличается от `AnimatedWidget`?

- A) Они взаимозаменяемы
- B) `AnimatedBuilder` — builder-паттерн inline с `child` параметром для оптимизации; `AnimatedWidget` — базовый класс для создания переиспользуемых анимироаемых виджетов
- C) `AnimatedWidget` работает без `AnimationController`
- D) `AnimatedBuilder` только для работы с несколькими анимациями

<details>
<summary>Ответ</summary>

**B) `AnimatedBuilder` — inline; `AnimatedWidget` — для переиспользования**

```dart
// AnimatedBuilder — встроенный (child не пересобирается!):
AnimatedBuilder(
  animation: _animation,
  child: const Icon(Icons.star, size: 50), // кэшируется
  builder: (ctx, child) => Transform.scale(
    scale: _animation.value,
    child: child, // переиспользуется
  ),
)

// AnimatedWidget — переиспользуемый виджет:
class SpinWidget extends AnimatedWidget {
  const SpinWidget({required Animation<double> animation, super.key})
      : super(listenable: animation);

  @override
  Widget build(BuildContext ctx) {
    final animation = listenable as Animation<double>;
    return Transform.rotate(angle: animation.value * 2 * pi, child: ...);
  }
}
```

</details>

---

### Вопрос 5 🟡

Как создать staggered анимацию (последовательная анимация нескольких элементов)?

- A) `StaggeredAnimation` виджет
- B) Использовать `Interval` tween — каждый элемент анимируется в своём временном интервале [0.0, 1.0]
- C) Отдельные `AnimationController` для каждого элемента
- D) `AnimationGroup([anim1, anim2, anim3])`

<details>
<summary>Ответ</summary>

**B) `Interval` в `CurvedAnimation` — временные промежутки**

```dart
// Один контроллер для всей staggered анимации:
_controller = AnimationController(vsync: this, duration: const Duration(ms: 1500));

_opacityAnim = Tween(begin: 0.0, end: 1.0).animate(CurvedAnimation(
  parent: _controller,
  curve: const Interval(0.0, 0.3, curve: Curves.easeIn), // первые 30%
));
_slideAnim = Tween(begin: Offset(1, 0), end: Offset.zero).animate(CurvedAnimation(
  parent: _controller,
  curve: const Interval(0.2, 0.6, curve: Curves.easeOut), // 20%-60%
));
_scaleAnim = Tween(begin: 0.5, end: 1.0).animate(CurvedAnimation(
  parent: _controller,
  curve: const Interval(0.5, 1.0, curve: Curves.bounceOut), // 50%-100%
));

_controller.forward();
```

</details>

---

### Вопрос 6 🟡

Как реализовать `FadeTransition` + `SlideTransition` одновременно?

- A) Нельзя — только одна transition за раз
- B) Оборнуть один `*Transition` в другой — flutter применяет оба трансформа к дочернему виджету
- C) `AnimationGroup(fade: ..., slide: ...)`
- D) Использовать `AnimatedContainer` с несколькими параметрами

<details>
<summary>Ответ</summary>

**B) Вложить `*Transition` виджеты друг в друга**

```dart
late Animation<double> _opacity;
late Animation<Offset> _position;

// Один AnimationController — несколько Tween'ов:
_opacity = Tween<double>(begin: 0.0, end: 1.0).animate(_controller);
_position = Tween<Offset>(
  begin: const Offset(0, 0.5),
  end: Offset.zero,
).animate(CurvedAnimation(parent: _controller, curve: Curves.easeOut));

// Применить оба:
FadeTransition(
  opacity: _opacity,
  child: SlideTransition(
    position: _position,
    child: const Card(child: Text('Появляюсь плавно!')),
  ),
)
```

</details>

---

### Вопрос 7 🟡

Как правильно добавить `listener` к `AnimationController` и избежать утечек памяти?

- A) `controller.addListener(() { setState(() {}); })` — и всё
- B) Хранить ссылку на listener-функцию для возможности `removeListener`, либо использовать `AnimatedBuilder`/`AnimationBuilder`
- C) Listener автоматически удаляется при `dispose()`
- D) Listener нельзя удалить после добавления

<details>
<summary>Ответ</summary>

**B) Хранить ссылку или использовать `AnimatedBuilder` вместо listener + setState**

```dart
// ПЛОХО — нельзя удалить анонимный listener:
_controller.addListener(() { setState(() {}); }); // утечка при пересборке

// ХОРОШО — именованный listener:
void _onAnimation() => setState(() {});

@override
void initState() {
  super.initState();
  _controller.addListener(_onAnimation);
}

@override
void dispose() {
  _controller.removeListener(_onAnimation);
  _controller.dispose();
  super.dispose();
}

// ЛУЧШИЙ СПОСОБ — AnimatedBuilder вместо listener:
AnimatedBuilder(
  animation: _controller,
  builder: (ctx, child) => Container(width: _controller.value * 200),
)
```

</details>

---

### Вопрос 8 🔴

Как реализовать физическую анимацию с `SpringSimulation`?

- A) `SpringAnimation(mass: 1, stiffness: 100)`
- B) `AnimationController.animateWith(SpringSimulation(...))` — физическая симуляция вместо кривой
- C) `Curves.spring` — встроена в Curves
- D) Физические анимации только через Rive

<details>
<summary>Ответ</summary>

**B) `_controller.animateWith(SpringSimulation(...))`**

```dart
void _startSpring() {
  final spring = SpringDescription(
    mass: 1,
    stiffness: 500,
    damping: 25,
  );

  final simulation = SpringSimulation(
    spring,
    _controller.value, // начальное положение
    1.0,               // целевое положение
    0.0,               // начальная скорость
  );

  _controller.animateWith(simulation);
}

// Также доступны:
// FrictionSimulation — замедление
// GravitySimulation — гравитация
// ScrollSpringSimulation — как в scroll physics
```

</details>

---

### Вопрос 9 🔴

Как использовать `AnimationController` вне `State` (например в ViewModel)?

- A) Нельзя — только в State
- B) Передать `TickerProvider vsync` извне; или использовать `SingleTickerProviderStateMixin` в специальных State-only виджетах-обёртках
- C) `AnimationController(vsync: TickerProvider.global())`
- D) `AnimationController.detached()` — работает без vsync

<details>
<summary>Ответ</summary>

**B) Передать `TickerProvider` или использовать `TickerProviderStateMixin` в виджете**

```dart
// ViewModel принимает vsync:
class AnimationViewModel {
  late AnimationController controller;

  void init(TickerProvider vsync) {
    controller = AnimationController(
      vsync: vsync,
      duration: const Duration(milliseconds: 300),
    );
  }

  void dispose() => controller.dispose();
}

// В Widget:
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
  final _vm = AnimationViewModel();

  @override
  void initState() {
    super.initState();
    _vm.init(this); // передать vsync
  }

  @override
  void dispose() {
    _vm.dispose();
    super.dispose();
  }
}
```

</details>

---

### Вопрос 10 🔴

Как реализовать анимацию основанную на жестах (`GestureDetector` + `AnimationController`)?

- A) `GestureDetector` автоматически анимирует
- B) `onPanUpdate` → `controller.value = clamp(...)`, `onPanEnd` → `fling()` или `animateTo()` — привязать position к value контроллера
- C) `DragAnimation` виджет
- D) Только через `Draggable` виджет

<details>
<summary>Ответ</summary>

**B) Привязать `controller.value` к позиции жеста в `onPanUpdate`**

```dart
class _DraggableCard extends StatefulWidget { ... }

class _State extends State<_DraggableCard> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, value: 0.5);
  }

  @override
  Widget build(ctx) => GestureDetector(
    onPanUpdate: (details) {
      _controller.value += details.delta.dx / 300; // нормализовать к [0, 1]
    },
    onPanEnd: (details) {
      final velocity = details.velocity.pixelsPerSecond.dx / 300;
      _controller.fling(velocity: velocity); // физический fling
    },
    child: AnimatedBuilder(
      animation: _controller,
      builder: (ctx, child) => Transform.translate(
        offset: Offset((_controller.value - 0.5) * 300, 0),
        child: child,
      ),
      child: Card(child: Text('Перетащи!')),
    ),
  );
}
```

</details>
