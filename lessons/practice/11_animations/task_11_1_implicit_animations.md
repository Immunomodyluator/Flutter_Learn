# 11.1: Implicit Animations — анимированная полоса калорий

> Project: FitMenu | Глава 11 — Animations

### 11.1: AnimatedCalorieBar с неявными анимациями

🎯 **Цель шага:** Создать анимированную полосу прогресса калорий через неявные анимации Flutter (`AnimatedContainer`, `AnimatedDefaultTextStyle`, `TweenAnimationBuilder`) — без AnimationController.

---

📝 **Техническое задание:**

Реализуй `lib/features/home/widgets/animated_calorie_bar.dart`.

**Требования:**
- Горизонтальная полоса прогресса калорий (0% → 100%)
- **Анимация цвета**: зелёный → жёлтый → красный при достижении 80% и 100%
- **Анимация ширины**: плавное заполнение при изменении калорий
- **Анимированное число**: счётчик плавно меняется через `TweenAnimationBuilder<double>`
- Длительность анимации: 600мс, кривая: `Curves.easeInOut`

**Виджеты для использования:**
```dart
// Анимация ширины и цвета фона:
AnimatedContainer(
  duration: const Duration(milliseconds: 600),
  curve: Curves.easeInOut,
  width: maxWidth * progress,
  decoration: BoxDecoration(color: _colorForProgress(progress)),
)

// Анимированный текст:
TweenAnimationBuilder<double>(
  tween: Tween(begin: 0, end: currentCalories.toDouble()),
  duration: const Duration(milliseconds: 600),
  builder: (context, value, _) => Text('${value.toInt()} ккал'),
)

// Анимация стиля текста:
AnimatedDefaultTextStyle(
  duration: const Duration(milliseconds: 300),
  style: TextStyle(color: _colorForProgress(progress), fontWeight: ...),
  child: Text('...'),
)
```

**Компонент AnimatedCalorieBar:**
```dart
class AnimatedCalorieBar extends StatelessWidget {
  const AnimatedCalorieBar({
    super.key,
    required this.current,
    required this.target,
  });
  final int current;
  final int target;
}
```

---

✅ **Критерии приёмки:**
- [ ] `AnimatedContainer` плавно анимирует ширину при изменении `current`
- [ ] Цвет полосы меняется анимированно (не резко)
- [ ] `TweenAnimationBuilder` анимирует числовое значение
- [ ] Нет `AnimationController` — только неявные анимации
- [ ] Виджет `StatelessWidget` (вся анимация — в `Animated*` виджетах)
- [ ] Кривая `Curves.easeInOut` применена ко всем анимациям

---

💡 **Подсказка:** Неявные анимации (`Animated*`) запускаются автоматически при изменении переданного значения. `LayoutBuilder` + `AnimatedContainer` с `width: constraints.maxWidth * progress` — классика для прогресс-баров. `TweenAnimationBuilder` позволяет анимировать любой тип через `Tween<T>`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/home/widgets/animated_calorie_bar.dart

import 'package:flutter/material.dart';

class AnimatedCalorieBar extends StatelessWidget {
  const AnimatedCalorieBar({
    super.key,
    required this.current,
    required this.target,
  });

  final int current;
  final int target;

  double get _progress => (current / target).clamp(0.0, 1.0);

  Color _colorForProgress(double p) {
    if (p >= 1.0) return Colors.red;
    if (p >= 0.8) return Colors.orange;
    return Colors.green;
  }

  @override
  Widget build(BuildContext context) {
    final color = _colorForProgress(_progress);

    return Column(
      crossAxisAlignment: CrossAxisAlignment.start,
      children: [
        // Числа с анимацией
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            TweenAnimationBuilder<double>(
              tween: Tween(begin: 0, end: current.toDouble()),
              duration: const Duration(milliseconds: 600),
              curve: Curves.easeInOut,
              builder: (_, value, __) => AnimatedDefaultTextStyle(
                duration: const Duration(milliseconds: 300),
                style: TextStyle(
                  color: color,
                  fontWeight: FontWeight.bold,
                  fontSize: 16,
                ),
                child: Text('${value.toInt()} ккал'),
              ),
            ),
            Text(
              '/ $target ккал',
              style: Theme.of(context).textTheme.bodySmall,
            ),
          ],
        ),

        const SizedBox(height: 8),

        // Полоса прогресса
        ClipRRect(
          borderRadius: BorderRadius.circular(6),
          child: LayoutBuilder(
            builder: (context, constraints) => Stack(
              children: [
                // Фон
                Container(
                  height: 12,
                  width: constraints.maxWidth,
                  decoration: BoxDecoration(
                    color: Theme.of(context).colorScheme.surfaceVariant,
                  ),
                ),
                // Прогресс
                AnimatedContainer(
                  duration: const Duration(milliseconds: 600),
                  curve: Curves.easeInOut,
                  height: 12,
                  width: constraints.maxWidth * _progress,
                  decoration: BoxDecoration(
                    color: color,
                    borderRadius: BorderRadius.circular(6),
                  ),
                ),
              ],
            ),
          ),
        ),
      ],
    );
  }
}
```

</details>
