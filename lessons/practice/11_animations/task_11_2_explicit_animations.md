# 11.2: Explicit Animations — пульсирующая иконка добавления

> Project: FitMenu | Глава 11 — Animations

### 11.2: AnimationController и Tween — пульсирующая FAB

🎯 **Цель шага:** Создать явную анимацию через `AnimationController` и `Tween` — кнопка добавления блюда пульсирует, привлекая внимание, когда план питания пустой.

---

📝 **Техническое задание:**

Реализуй `lib/features/meal_plan/widgets/pulsing_add_button.dart`.

**Анимация:**
- FAB пульсирует (масштаб 1.0 → 1.15 → 1.0) бесконечно
- Параллельно меняется тень (elevation)
- При нажатии пульсация останавливается, кнопка возвращается в нормальное состояние
- Анимация запускается только когда список пуст (`isEmpty: true`)

**Технические требования:**
```dart
class PulsingAddButton extends StatefulWidget {
  const PulsingAddButton({super.key, required this.onPressed, this.isEmpty = false});
  final VoidCallback onPressed;
  final bool isEmpty; // пульсировать только при пустом списке
}

class _PulsingAddButtonState extends State<PulsingAddButton>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  late Animation<double> _elevationAnimation;
  ...
}
```

**Настройка анимаций:**
```dart
_controller = AnimationController(
  vsync: this,
  duration: const Duration(milliseconds: 800),
);

_scaleAnimation = Tween<double>(begin: 1.0, end: 1.15)
    .animate(CurvedAnimation(parent: _controller, curve: Curves.easeInOut));

_elevationAnimation = Tween<double>(begin: 4, end: 12)
    .animate(CurvedAnimation(parent: _controller, curve: Curves.easeInOut));

// Бесконечная пульсация через reverse
_controller.addStatusListener((status) {
  if (status == AnimationStatus.completed) _controller.reverse();
  if (status == AnimationStatus.dismissed) _controller.forward();
});
```

**AnimatedBuilder для UI:**
```dart
AnimatedBuilder(
  animation: _controller,
  builder: (context, child) => Transform.scale(
    scale: _scaleAnimation.value,
    child: FloatingActionButton.extended(
      elevation: _elevationAnimation.value,
      onPressed: widget.onPressed,
      icon: const Icon(Icons.add),
      label: const Text('Добавить блюдо'),
    ),
  ),
)
```

---

✅ **Критерии приёмки:**
- [ ] `AnimationController` создаётся в `initState` и уничтожается в `dispose`
- [ ] `SingleTickerProviderStateMixin` используется (не `TickerProviderStateMixin`)
- [ ] Пульсация останавливается когда `isEmpty = false`
- [ ] `AnimatedBuilder` перестраивает только FAB, не весь виджет
- [ ] `controller.dispose()` вызывается в `dispose()`
- [ ] `_controller.repeat(reverse: true)` — альтернативный способ бесконечной анимации

---

💡 **Подсказка:** `_controller.repeat(reverse: true)` — краткий способ запустить пульсацию вместо ручного `addStatusListener`. `CurvedAnimation` оборачивает контроллер, применяя кривую без изменения самого Tween. `AnimatedBuilder` — эффективнее `setState` внутри `addListener`, так как перестраивает только поддерево.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_plan/widgets/pulsing_add_button.dart

import 'package:flutter/material.dart';

class PulsingAddButton extends StatefulWidget {
  const PulsingAddButton({
    super.key,
    required this.onPressed,
    this.isEmpty = false,
  });

  final VoidCallback onPressed;
  final bool isEmpty;

  @override
  State<PulsingAddButton> createState() => _PulsingAddButtonState();
}

class _PulsingAddButtonState extends State<PulsingAddButton>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  late Animation<double> _elevationAnimation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(milliseconds: 800),
    );

    final curved = CurvedAnimation(
      parent: _controller,
      curve: Curves.easeInOut,
    );

    _scaleAnimation     = Tween<double>(begin: 1.0, end: 1.15).animate(curved);
    _elevationAnimation = Tween<double>(begin: 4.0, end: 14.0).animate(curved);

    _updateAnimation();
  }

  @override
  void didUpdateWidget(PulsingAddButton old) {
    super.didUpdateWidget(old);
    if (old.isEmpty != widget.isEmpty) _updateAnimation();
  }

  void _updateAnimation() {
    if (widget.isEmpty) {
      _controller.repeat(reverse: true);
    } else {
      _controller.stop();
      _controller.animateTo(0); // вернуть в исходное положение
    }
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
      builder: (context, child) => Transform.scale(
        scale: _scaleAnimation.value,
        child: FloatingActionButton.extended(
          elevation: _elevationAnimation.value,
          onPressed: widget.onPressed,
          icon: const Icon(Icons.add),
          label: const Text('Добавить блюдо'),
        ),
      ),
    );
  }
}
```

</details>
