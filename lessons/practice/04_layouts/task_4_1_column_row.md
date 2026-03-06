# 4.1: Column и Row — вёрстка полосы КБЖУ

> Project: FitMenu | Глава 4 — Layouts

### 4.1: NutritionStatsBar с Column, Row, Expanded и Flexible

🎯 **Цель шага:** Создать виджет `NutritionStatsBar`, который отображает дневной баланс КБЖУ (калории, белки, жиры, углеводы) в виде горизонтальной полосы с иконками и подписями — используя `Row`, `Column`, `Expanded` и `Flexible`.

---

📝 **Техническое задание:**

Реализуй виджет `NutritionStatsBar` в файле `lib/features/home/widgets/nutrition_stats_bar.dart`.

**Требования к UI:**
- Полоса расположена горизонтально (`Row`)
- 4 блока: Калории / Белки / Жиры / Углеводы
- Каждый блок — `Column`: иконка → значение → подпись (г / ккал)
- Блоки занимают равное пространство (`Expanded`)
- Между блоками — вертикальный разделитель `VerticalDivider`
- Под полосой — тонкая линия прогресса (`LinearProgressIndicator`), показывающая % выполнения дневной нормы калорий

**Требования к данным:**
```dart
class NutritionStats {
  final double calories;     // текущие ккал
  final double targetCalories; // норма ккал
  final double protein;      // г
  final double fat;          // г
  final double carbs;        // г
}
```

- Виджет принимает `NutritionStats stats` через конструктор
- Прогресс = `stats.calories / stats.targetCalories` (clamp 0.0–1.0)

---

✅ **Критерии приёмки:**
- [ ] Все 4 блока отображаются в одну строку без overflow
- [ ] На узком экране (320px) текст не выходит за границы — используется `FittedBox` или `overflow: TextOverflow.ellipsis`
- [ ] `LinearProgressIndicator` показывает реальный прогресс (передай `value: stats.calories / stats.targetCalories`)
- [ ] Виджет является `StatelessWidget` (данные приходят снаружи)
- [ ] Код соответствует Effective Dart: именование, типизация, const-конструкторы

---

💡 **Подсказка:** Используй `Expanded` внутри `Row` для равного распределения пространства. Чтобы иконки и текст не переполнялись на маленьких экранах, оберни контент каждого блока в `FittedBox(fit: BoxFit.scaleDown)`. `VerticalDivider` требует ограниченной высоты — оберни `Row` в `IntrinsicHeight`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/home/widgets/nutrition_stats_bar.dart

import 'package:flutter/material.dart';

class NutritionStats {
  const NutritionStats({
    required this.calories,
    required this.targetCalories,
    required this.protein,
    required this.fat,
    required this.carbs,
  });

  final double calories;
  final double targetCalories;
  final double protein;
  final double fat;
  final double carbs;
}

class NutritionStatsBar extends StatelessWidget {
  const NutritionStatsBar({super.key, required this.stats});

  final NutritionStats stats;

  @override
  Widget build(BuildContext context) {
    final progress = (stats.calories / stats.targetCalories).clamp(0.0, 1.0);
    final colorScheme = Theme.of(context).colorScheme;

    return Card(
      margin: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      child: Padding(
        padding: const EdgeInsets.all(12),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            // Полоса из 4 блоков
            IntrinsicHeight(
              child: Row(
                children: [
                  _NutrientBlock(
                    icon: Icons.local_fire_department,
                    value: stats.calories.toStringAsFixed(0),
                    unit: 'ккал',
                    label: 'Калории',
                    color: Colors.orange,
                  ),
                  const VerticalDivider(width: 1),
                  _NutrientBlock(
                    icon: Icons.egg_alt,
                    value: stats.protein.toStringAsFixed(1),
                    unit: 'г',
                    label: 'Белки',
                    color: Colors.blue,
                  ),
                  const VerticalDivider(width: 1),
                  _NutrientBlock(
                    icon: Icons.water_drop,
                    value: stats.fat.toStringAsFixed(1),
                    unit: 'г',
                    label: 'Жиры',
                    color: Colors.yellow[800]!,
                  ),
                  const VerticalDivider(width: 1),
                  _NutrientBlock(
                    icon: Icons.grain,
                    value: stats.carbs.toStringAsFixed(1),
                    unit: 'г',
                    label: 'Углеводы',
                    color: Colors.green,
                  ),
                ],
              ),
            ),
            const SizedBox(height: 10),
            // Прогресс-бар калорий
            ClipRRect(
              borderRadius: BorderRadius.circular(4),
              child: LinearProgressIndicator(
                value: progress,
                minHeight: 6,
                backgroundColor: colorScheme.surfaceVariant,
                color: progress >= 1.0 ? Colors.red : colorScheme.primary,
              ),
            ),
            const SizedBox(height: 4),
            Text(
              '${stats.calories.toStringAsFixed(0)} / ${stats.targetCalories.toStringAsFixed(0)} ккал',
              style: Theme.of(context).textTheme.bodySmall,
            ),
          ],
        ),
      ),
    );
  }
}

class _NutrientBlock extends StatelessWidget {
  const _NutrientBlock({
    required this.icon,
    required this.value,
    required this.unit,
    required this.label,
    required this.color,
  });

  final IconData icon;
  final String value;
  final String unit;
  final String label;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return Expanded(
      child: FittedBox(
        fit: BoxFit.scaleDown,
        child: Padding(
          padding: const EdgeInsets.symmetric(horizontal: 4),
          child: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              Icon(icon, color: color, size: 20),
              const SizedBox(height: 2),
              Text(
                '$value $unit',
                style: Theme.of(context)
                    .textTheme
                    .titleSmall
                    ?.copyWith(fontWeight: FontWeight.bold),
              ),
              Text(
                label,
                style: Theme.of(context).textTheme.labelSmall,
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
