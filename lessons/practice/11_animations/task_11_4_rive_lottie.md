# 11.4: Rive и Lottie — анимация достижения цели

> Project: FitMenu | Глава 11 — Animations

### 11.4: Lottie анимация при выполнении дневной нормы калорий

🎯 **Цель шага:** Показать Lottie-анимацию "🎉 Цель достигнута!" когда пользователь набирает дневную норму калорий — интегрировать готовую анимацию из LottieFiles в FitMenu.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  lottie: ^3.0.0
```

**Ресурс:** скачай бесплатную анимацию с lottiefiles.com (например: "success checkmark", "confetti", "trophy") и сохрани в `assets/animations/goal_achieved.json`.

```yaml
flutter:
  assets:
    - assets/animations/
```

Реализуй `lib/features/home/widgets/goal_achieved_overlay.dart`.

**GoalAchievedOverlay:**
```dart
class GoalAchievedOverlay extends StatefulWidget {
  const GoalAchievedOverlay({super.key, required this.onDismiss});
  final VoidCallback onDismiss;
}
```

**Поведение:**
- Полупрозрачный чёрный фон (`Colors.black54`)
- По центру: Lottie-анимация 200×200
- Под анимацией: "🎯 Цель достигнута!"
- Анимация играет один раз, затем вызывает `onDismiss`
- Появление/исчезновение: `AnimatedOpacity`

**Lottie контроллер:**
```dart
class _GoalAchievedOverlayState extends State<GoalAchievedOverlay>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
  }

  @override
  Widget build(BuildContext context) {
    return Lottie.asset(
      'assets/animations/goal_achieved.json',
      controller: _controller,
      onLoaded: (composition) {
        _controller
          ..duration = composition.duration
          ..forward().then((_) => widget.onDismiss());
      },
    );
  }
}
```

**Показ из родителя:**
```dart
if (calories >= targetCalories && !_goalShown) {
  setState(() => _goalShown = true);
  showDialog(
    context: context,
    barrierDismissible: true,
    builder: (_) => GoalAchievedOverlay(
      onDismiss: () => Navigator.pop(context),
    ),
  );
}
```

---

✅ **Критерии приёмки:**
- [ ] Lottie-файл находится в `assets/animations/` и зарегистрирован
- [ ] Анимация играет один раз (не зацикливается)
- [ ] `onLoaded` устанавливает длительность из composition
- [ ] После завершения анимации вызывается `onDismiss`
- [ ] `_controller.dispose()` вызывается в `dispose()`
- [ ] Оверлей можно закрыть нажатием на фон (`barrierDismissible: true`)

---

💡 **Подсказка:** `Lottie.network(url)` — загрузить анимацию по URL без добавления ассета (удобно для прототипирования). `Lottie.asset` с `controller` — управляемое воспроизведение. Без `controller` Lottie воспроизводится автоматически в цикле. `composition.duration` — реальная длительность анимации из JSON-файла.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/home/widgets/goal_achieved_overlay.dart

import 'package:flutter/material.dart';
import 'package:lottie/lottie.dart';

class GoalAchievedOverlay extends StatefulWidget {
  const GoalAchievedOverlay({super.key, required this.onDismiss});

  final VoidCallback onDismiss;

  @override
  State<GoalAchievedOverlay> createState() => _GoalAchievedOverlayState();
}

class _GoalAchievedOverlayState extends State<GoalAchievedOverlay>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  bool _visible = false;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this);
    // Появление с задержкой
    WidgetsBinding.instance.addPostFrameCallback((_) {
      if (mounted) setState(() => _visible = true);
    });
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return AnimatedOpacity(
      opacity: _visible ? 1.0 : 0.0,
      duration: const Duration(milliseconds: 300),
      child: Material(
        color: Colors.black54,
        child: Center(
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              // Lottie анимация
              Lottie.asset(
                'assets/animations/goal_achieved.json',
                controller: _controller,
                width: 200,
                height: 200,
                onLoaded: (composition) {
                  _controller.duration = composition.duration;
                  _controller.forward().then((_) {
                    if (mounted) widget.onDismiss();
                  });
                },
              ),

              const SizedBox(height: 16),

              // Текст
              Text(
                '🎯 Цель достигнута!',
                style: Theme.of(context).textTheme.headlineSmall?.copyWith(
                      color: Colors.white,
                      fontWeight: FontWeight.bold,
                    ),
              ),
              const SizedBox(height: 8),
              Text(
                'Отличная работа! Ты выполнил норму на сегодня.',
                style: Theme.of(context)
                    .textTheme
                    .bodyMedium
                    ?.copyWith(color: Colors.white70),
                textAlign: TextAlign.center,
              ),

              const SizedBox(height: 24),

              TextButton(
                onPressed: widget.onDismiss,
                child: const Text(
                  'Продолжить',
                  style: TextStyle(color: Colors.white),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

</details>
