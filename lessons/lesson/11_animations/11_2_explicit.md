# 11.2 Explicit-анимации (явные)

## 1. Суть

Explicit-анимации дают полный контроль: запуск, пауза, повтор, реверс, синхронизация нескольких анимаций. Требуют `AnimationController` и `TickerProviderStateMixin`.

---

## 2. Основные компоненты

```dart
// Иерархия явной анимации:
// AnimationController (0.0 → 1.0) → Tween (маппинг) → Animation<T> → AnimatedBuilder
```

---

## 3. Базовый синтаксис

```dart
class MyAnimatedWidget extends StatefulWidget {
  const MyAnimatedWidget({super.key});

  @override
  State<MyAnimatedWidget> createState() => _MyAnimatedWidgetState();
}

class _MyAnimatedWidgetState extends State<MyAnimatedWidget>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double> _scaleAnimation;
  late final Animation<Color?> _colorAnimation;

  @override
  void initState() {
    super.initState();

    // Контроллер: источник значений 0.0 → 1.0
    _controller = AnimationController(
      duration: const Duration(milliseconds: 600),
      vsync: this, // TickerProvider
    );

    // Tween — маппинг 0.0→1.0 в 0.5→1.2
    _scaleAnimation = Tween<double>(begin: 0.5, end: 1.2)
        .animate(CurvedAnimation(parent: _controller, curve: Curves.elasticOut));

    // Цвет
    _colorAnimation = ColorTween(begin: Colors.blue, end: Colors.purple)
        .animate(CurvedAnimation(parent: _controller, curve: Curves.easeInOut));

    // Запустить анимацию
    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose(); // ОБЯЗАТЕЛЬНО! иначе утечка памяти
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller, // перестраивает при каждом изменении
      builder: (context, child) {
        return Transform.scale(
          scale: _scaleAnimation.value,
          child: Container(
            width: 100,
            height: 100,
            color: _colorAnimation.value,
            child: child, // статичная часть — не перестраивается
          ),
        );
      },
      child: const Center(child: Icon(Icons.star, color: Colors.white)),
    );
  }
}
```

---

## 4. Управление контроллером

```dart
_controller.forward();        // воспроизвести вперёд (0 → 1)
_controller.reverse();        // воспроизвести назад (1 → 0)
_controller.repeat();         // зациклить
_controller.repeat(reverse: true); // туда-обратно (ping-pong)
_controller.stop();           // остановить
_controller.reset();          // вернуть в 0
_controller.animateTo(0.5);   // анимировать к конкретному значению

// Текущее значение
print(_controller.value);     // 0.0 → 1.0
print(_scaleAnimation.value); // значение после Tween

// Статус
_controller.status; // AnimationStatus: forward, reverse, completed, dismissed

// Слушать завершение
_controller.addStatusListener((status) {
  if (status == AnimationStatus.completed) {
    _controller.reverse();
  }
});
```

---

## 5. Несколько анимаций — Interval

```dart
// Staggered (каскадные) анимации через Interval
class StaggeredAnimation extends StatefulWidget {
  const StaggeredAnimation({super.key});

  @override
  State<StaggeredAnimation> createState() => _StaggeredAnimationState();
}

class _StaggeredAnimationState extends State<StaggeredAnimation>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<Offset> _slide1;
  late final Animation<Offset> _slide2;
  late final Animation<Offset> _slide3;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: const Duration(milliseconds: 1200),
      vsync: this,
    );

    // Каждый элемент анимирует в своём временном окне
    _slide1 = Tween<Offset>(begin: const Offset(-1, 0), end: Offset.zero)
        .animate(CurvedAnimation(
          parent: _controller,
          curve: const Interval(0.0, 0.4, curve: Curves.easeOut),
        ));

    _slide2 = Tween<Offset>(begin: const Offset(-1, 0), end: Offset.zero)
        .animate(CurvedAnimation(
          parent: _controller,
          curve: const Interval(0.2, 0.6, curve: Curves.easeOut),
        ));

    _slide3 = Tween<Offset>(begin: const Offset(-1, 0), end: Offset.zero)
        .animate(CurvedAnimation(
          parent: _controller,
          curve: const Interval(0.4, 0.8, curve: Curves.easeOut),
        ));

    _controller.forward();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, _) {
        return Column(
          children: [
            SlideTransition(position: _slide1, child: const ListTile(title: Text('Элемент 1'))),
            SlideTransition(position: _slide2, child: const ListTile(title: Text('Элемент 2'))),
            SlideTransition(position: _slide3, child: const ListTile(title: Text('Элемент 3'))),
          ],
        );
      },
    );
  }
}
```

---

## 6. Готовые Transition-виджеты

```dart
// FadeTransition — прозрачность
FadeTransition(
  opacity: _fadeAnimation, // Animation<double>
  child: ...,
)

// SlideTransition — сдвиг
SlideTransition(
  position: Tween<Offset>(
    begin: const Offset(0, 1), // снизу
    end: Offset.zero,
  ).animate(_controller),
  child: ...,
)

// ScaleTransition — масштаб
ScaleTransition(
  scale: _scaleAnimation,
  child: ...,
)

// RotationTransition — вращение
RotationTransition(
  turns: _controller, // 0→1 = 360°
  child: const Icon(Icons.refresh),
)

// SizeTransition — изменение размера
SizeTransition(
  sizeFactor: _controller,
  axis: Axis.vertical,
  child: ...,
)
```

---

## 7. Под капотом

- `AnimationController` использует `Ticker` — callback от движка при каждом кадре (`vsync`).
- `vsync: this` с `TickerProviderStateMixin` — анимация останавливается когда виджет невидим (оптимизация батареи).
- `AnimatedBuilder` подписывается на `addListener` контроллера и вызывает `setState` при каждом изменении.
- `Interval(0.2, 0.6)` — нормализует значения контроллера в диапазон 0.0-1.0 для нужного временного окна.

---

## 8. Типичные ошибки

| Ошибка                            | Причина                                            | Решение                                                     |
| --------------------------------- | -------------------------------------------------- | ----------------------------------------------------------- |
| `_controller.dispose()` не вызван | Забыл переопределить `dispose()`                   | Всегда вызывай `_controller.dispose()` в `dispose()`        |
| `TickerDisposedException`         | Обратный вызов после dispose                       | Проверяй `mounted` или отменяй listeners в dispose          |
| Несколько контроллеров            | `SingleTickerProviderStateMixin` только для одного | Используй `TickerProviderStateMixin` для нескольких         |
| Анимация тормозит                 | Перестройка тяжёлых виджетов на каждом кадре       | Выноси статичные части в `child` аргумент `AnimatedBuilder` |

---

## 9. Рекомендации

1. **`SingleTickerProviderStateMixin`** — один контроллер, **`TickerProviderStateMixin`** — несколько.
2. **Всегда `dispose()`** контроллер — это источник утечек памяти №1 в анимациях.
3. **`AnimatedBuilder` + `child`** — статичные виджеты не перестраиваются на каждом кадре.
4. **Staggered через `Interval`** — один контроллер для каскадных анимаций.
5. **`addStatusListener`** — для запуска действий по завершению анимации.
