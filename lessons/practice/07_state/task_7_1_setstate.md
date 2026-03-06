# 7.1: setState — локальное состояние счётчика калорий

> Project: FitMenu | Глава 7 — State Management

### 7.1: Счётчик дневных калорий через setState

🎯 **Цель шага:** Реализовать виджет `CalorieCounter` — интерактивный счётчик съеденных калорий за день, который обновляется локально через `setState`. Понять когда `setState` достаточно, а когда нужен внешний стейт-менеджер.

---

📝 **Техническое задание:**

Реализуй виджет `lib/features/home/widgets/calorie_counter.dart`.

**UI:**
- Большой круговой индикатор прогресса (`CircularProgressIndicator` или кастомный)
- В центре: текущие калории / норма (например: "1240 / 2000")
- Под ним: три кнопки быстрого добавления: +100 ккал, +200 ккал, +500 ккал
- Кнопка "Сбросить" (иконка `Icons.refresh`)
- Цвет индикатора меняется: зелёный < 80%, жёлтый 80–100%, красный > 100%
- Анимированное изменение числа (используй `AnimatedSwitcher`)

**Логика:**
```dart
class CalorieCounter extends StatefulWidget {
  const CalorieCounter({super.key, required this.targetCalories});
  final int targetCalories;
}

class _CalorieCounterState extends State<CalorieCounter> {
  int _currentCalories = 0;

  void _add(int amount) => setState(() => _currentCalories += amount);
  void _reset() => setState(() => _currentCalories = 0);
}
```

**Цвет прогресса:**
```dart
Color get _progressColor {
  final ratio = _currentCalories / widget.targetCalories;
  if (ratio > 1.0) return Colors.red;
  if (ratio > 0.8) return Colors.orange;
  return Colors.green;
}
```

---

✅ **Критерии приёмки:**
- [ ] `setState` вызывается только внутри State-класса
- [ ] Цвет индикатора меняется при достижении 80% и 100%
- [ ] `AnimatedSwitcher` анимирует смену числа калорий
- [ ] Кнопка "Сбросить" обнуляет счётчик
- [ ] Прогресс не превышает 1.0 визуально (`.clamp(0.0, 1.0)`)
- [ ] Виджет корректно принимает `targetCalories` снаружи

---

💡 **Подсказка:** `CircularProgressIndicator` принимает `value` от 0.0 до 1.0 — не забудь `clamp`. `AnimatedSwitcher` работает с уникальными `key` у дочерних виджетов — используй `ValueKey(_currentCalories)` чтобы анимация срабатывала при каждом изменении.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/home/widgets/calorie_counter.dart

import 'package:flutter/material.dart';

class CalorieCounter extends StatefulWidget {
  const CalorieCounter({super.key, required this.targetCalories});

  final int targetCalories;

  @override
  State<CalorieCounter> createState() => _CalorieCounterState();
}

class _CalorieCounterState extends State<CalorieCounter> {
  int _currentCalories = 0;

  void _add(int amount) => setState(() => _currentCalories += amount);
  void _reset() => setState(() => _currentCalories = 0);

  double get _progress =>
      (_currentCalories / widget.targetCalories).clamp(0.0, 1.0);

  Color get _progressColor {
    final ratio = _currentCalories / widget.targetCalories;
    if (ratio > 1.0) return Colors.red;
    if (ratio > 0.8) return Colors.orange;
    return Colors.green;
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        // Круговой прогресс
        SizedBox(
          width: 160,
          height: 160,
          child: Stack(
            alignment: Alignment.center,
            children: [
              SizedBox.expand(
                child: CircularProgressIndicator(
                  value: _progress,
                  strokeWidth: 12,
                  color: _progressColor,
                  backgroundColor:
                      Theme.of(context).colorScheme.surfaceVariant,
                ),
              ),
              // Анимированное число
              AnimatedSwitcher(
                duration: const Duration(milliseconds: 300),
                transitionBuilder: (child, animation) =>
                    ScaleTransition(scale: animation, child: child),
                child: Column(
                  key: ValueKey(_currentCalories),
                  mainAxisSize: MainAxisSize.min,
                  children: [
                    Text(
                      '$_currentCalories',
                      style: Theme.of(context).textTheme.headlineMedium
                          ?.copyWith(
                              fontWeight: FontWeight.bold,
                              color: _progressColor),
                    ),
                    Text(
                      '/ ${widget.targetCalories} ккал',
                      style: Theme.of(context).textTheme.labelSmall,
                    ),
                  ],
                ),
              ),
            ],
          ),
        ),

        const SizedBox(height: 16),

        // Кнопки быстрого добавления
        Row(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [100, 200, 500]
              .map(
                (amount) => Padding(
                  padding: const EdgeInsets.symmetric(horizontal: 4),
                  child: OutlinedButton(
                    onPressed: () => _add(amount),
                    child: Text('+$amount'),
                  ),
                ),
              )
              .toList(),
        ),

        const SizedBox(height: 8),

        TextButton.icon(
          onPressed: _reset,
          icon: const Icon(Icons.refresh, size: 16),
          label: const Text('Сбросить'),
        ),
      ],
    );
  }
}
```

</details>
