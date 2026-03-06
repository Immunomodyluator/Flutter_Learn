# 18.1: CustomPainter — диаграмма калорий CalorieRingChart

> Project: FitMenu | Глава 18 — Advanced

### 18.1: CalorieRingChart — круговая диаграмма КБЖУ через CustomPainter

🎯 **Цель шага:** Создать виджет `CalorieRingChart` — кольцевую диаграмму прогресса КБЖУ (калории, белки, жиры, углеводы) используя `CustomPainter` с анимацией появления через `AnimationController`.

---

📝 **Техническое задание:**

**Данные для диаграммы:**
```dart
class NutritionRingData {
  const NutritionRingData({
    required this.calories,      // потреблено ккал
    required this.targetCalories,
    required this.protein,       // г
    required this.targetProtein,
    required this.fat,
    required this.targetFat,
    required this.carbs,
    required this.targetCarbs,
  });
  // ...
}
```

**CustomPainter:**
```dart
class _CalorieRingPainter extends CustomPainter {
  const _CalorieRingPainter({
    required this.data,
    required this.animationValue, // 0.0 → 1.0
  });

  final NutritionRingData data;
  final double animationValue;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final rings = [
      _Ring(color: Color(0xFF6750A4), progress: data.calories / data.targetCalories,  radius: size.width * 0.45, strokeWidth: 18),
      _Ring(color: Color(0xFF00897B), progress: data.protein  / data.targetProtein,   radius: size.width * 0.35, strokeWidth: 14),
      _Ring(color: Color(0xFFE53935), progress: data.fat      / data.targetFat,        radius: size.width * 0.25, strokeWidth: 14),
      _Ring(color: Color(0xFFFB8C00), progress: data.carbs    / data.targetCarbs,      radius: size.width * 0.15, strokeWidth: 14),
    ];

    for (final ring in rings) {
      // Нарисовать фоновую дугу (серая)
      canvas.drawArc(
        Rect.fromCircle(center: center, radius: ring.radius),
        -pi / 2, 2 * pi,
        false,
        Paint()
          ..color       = ring.color.withOpacity(0.15)
          ..style       = PaintingStyle.stroke
          ..strokeWidth = ring.strokeWidth
          ..strokeCap   = StrokeCap.round,
      );
      // Нарисовать прогресс
      canvas.drawArc(
        Rect.fromCircle(center: center, radius: ring.radius),
        -pi / 2,
        2 * pi * ring.progress.clamp(0, 1) * animationValue,
        false,
        Paint()
          ..color       = ring.color
          ..style       = PaintingStyle.stroke
          ..strokeWidth = ring.strokeWidth
          ..strokeCap   = StrokeCap.round,
      );
    }

    // Текст калорий в центре
    _drawCenterText(canvas, size, data.calories.toInt(), data.targetCalories.toInt());
  }

  @override
  bool shouldRepaint(_CalorieRingPainter oldDelegate) =>
      oldDelegate.animationValue != animationValue || oldDelegate.data != data;
}
```

**Виджет с анимацией:**
```dart
class CalorieRingChart extends StatefulWidget {
  const CalorieRingChart({super.key, required this.data, this.size = 200});
  final NutritionRingData data;
  final double size;
  @override State<CalorieRingChart> createState() => _CalorieRingChartState();
}
```

---

✅ **Критерии приёмки:**
- [ ] 4 концентрических кольца: калории, белки, жиры, углеводы
- [ ] Прогресс > 100% обрезается через `.clamp(0, 1)`
- [ ] Анимация появления от 0 до финального значения за 1200ms
- [ ] `shouldRepaint` возвращает `true` только при изменении данных
- [ ] В центре кольца отображается текст с количеством калорий
- [ ] Виджет адаптируется под любой переданный `size`

---

💡 **Подсказка:** `canvas.drawArc(rect, startAngle, sweepAngle, useCenter, paint)` — угол в радианах (-π/2 = верх). `TextPainter` рисует текст на canvas. `Paint()..strokeCap = StrokeCap.round` делает закруглённые концы дуги.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/dashboard/presentation/widgets/calorie_ring_chart.dart

import 'dart:math' show pi;
import 'package:flutter/material.dart';

class CalorieRingChart extends StatefulWidget {
  const CalorieRingChart({super.key, required this.data, this.size = 220});
  final NutritionRingData data;
  final double size;
  @override State<CalorieRingChart> createState() => _CalorieRingChartState();
}

class _CalorieRingChartState extends State<CalorieRingChart>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  late final Animation<double>   _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync:    this,
      duration: const Duration(milliseconds: 1200),
    );
    _animation = CurvedAnimation(
      parent: _controller, curve: Curves.easeOutCubic,
    );
    _controller.forward();
  }

  @override
  void didUpdateWidget(CalorieRingChart oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (oldWidget.data != widget.data) {
      _controller
        ..reset()
        ..forward();
    }
  }

  @override
  void dispose() { _controller.dispose(); super.dispose(); }

  @override
  Widget build(BuildContext context) => AnimatedBuilder(
    animation: _animation,
    builder:   (_, __) => CustomPaint(
      size:    Size(widget.size, widget.size),
      painter: _CalorieRingPainter(
        data:           widget.data,
        animationValue: _animation.value,
      ),
    ),
  );
}

// ─── Painter ────────────────────────────────────────────────────────────────

class _Ring {
  const _Ring({required this.color, required this.progress, required this.radius, required this.strokeWidth});
  final Color  color;
  final double progress;
  final double radius;
  final double strokeWidth;
}

class _CalorieRingPainter extends CustomPainter {
  const _CalorieRingPainter({required this.data, required this.animationValue});
  final NutritionRingData data;
  final double animationValue;

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final rings  = [
      _Ring(color: const Color(0xFF6750A4), progress: (data.calories / data.targetCalories).clamp(0, 1), radius: size.width * 0.44, strokeWidth: 18),
      _Ring(color: const Color(0xFF00897B), progress: (data.protein  / data.targetProtein ).clamp(0, 1), radius: size.width * 0.33, strokeWidth: 14),
      _Ring(color: const Color(0xFFE53935), progress: (data.fat      / data.targetFat     ).clamp(0, 1), radius: size.width * 0.22, strokeWidth: 14),
      _Ring(color: const Color(0xFFFB8C00), progress: (data.carbs    / data.targetCarbs   ).clamp(0, 1), radius: size.width * 0.11, strokeWidth: 14),
    ];

    for (final ring in rings) {
      final bgPaint = Paint()
        ..color       = ring.color.withOpacity(0.15)
        ..style       = PaintingStyle.stroke
        ..strokeWidth = ring.strokeWidth;
      final fgPaint = Paint()
        ..color       = ring.color
        ..style       = PaintingStyle.stroke
        ..strokeWidth = ring.strokeWidth
        ..strokeCap   = StrokeCap.round;

      final rect = Rect.fromCircle(center: center, radius: ring.radius);
      canvas.drawArc(rect, -pi / 2, 2 * pi, false, bgPaint);
      canvas.drawArc(rect, -pi / 2, 2 * pi * ring.progress * animationValue, false, fgPaint);
    }

    // Текст в центре
    final tp = TextPainter(
      text: TextSpan(children: [
        TextSpan(text: '${data.calories.toInt()}\n', style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold, color: Colors.black87)),
        TextSpan(text: 'из ${data.targetCalories.toInt()} ккал', style: const TextStyle(fontSize: 11, color: Colors.black54)),
      ]),
      textAlign:    TextAlign.center,
      textDirection: TextDirection.ltr,
    )..layout(maxWidth: size.width * 0.35);
    tp.paint(canvas, Offset(center.dx - tp.width / 2, center.dy - tp.height / 2));
  }

  @override
  bool shouldRepaint(_CalorieRingPainter old) =>
      old.animationValue != animationValue || old.data != data;
}
```

</details>
