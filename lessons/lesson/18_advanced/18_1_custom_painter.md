# 18.1 CustomPainter и Canvas

## 1. Суть

`CustomPainter` — способ рисовать произвольную 2D-графику в Flutter с помощью `Canvas` API. Используется для: графиков, диаграмм, кастомных форм, игр, кастомных индикаторов, сложной анимации.

---

## 2. Базовый синтаксис

```dart
class MyPainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 2
      ..style = PaintingStyle.stroke; // stroke или fill

    // Нарисовать линию
    canvas.drawLine(
      Offset(0, size.height / 2),
      Offset(size.width, size.height / 2),
      paint,
    );

    // Нарисовать круг
    canvas.drawCircle(
      Offset(size.width / 2, size.height / 2),
      50,
      paint..style = PaintingStyle.fill..color = Colors.red,
    );

    // Нарисовать прямоугольник
    canvas.drawRect(
      Rect.fromLTWH(10, 10, 100, 60),
      paint..color = Colors.green,
    );

    // Нарисовать скруглённый прямоугольник
    canvas.drawRRect(
      RRect.fromRectAndRadius(
        Rect.fromLTWH(10, 80, 100, 60),
        const Radius.circular(12),
      ),
      paint,
    );
  }

  @override
  bool shouldRepaint(MyPainter oldDelegate) => false;
}

// Использование
CustomPaint(
  size: const Size(300, 200),
  painter: MyPainter(),
  child: const Text('Поверх canvas'), // child отображается поверх
)
```

---

## 3. Основные операции Canvas

```dart
void paint(Canvas canvas, Size size) {
  final paint = Paint();

  // --- Трансформации ---
  canvas.save();                                    // сохранить состояние
  canvas.translate(100, 50);                         // сдвиг
  canvas.rotate(math.pi / 4);                        // поворот в радианах
  canvas.scale(2.0, 2.0);                            // масштаб
  canvas.restore();                                  // восстановить состояние

  // --- Фигуры ---
  canvas.drawLine(Offset.zero, Offset(size.width, size.height), paint);
  canvas.drawOval(Rect.fromLTWH(0, 0, 100, 50), paint);
  canvas.drawArc(
    Rect.fromCircle(center: Offset(100, 100), radius: 80),
    0,                // startAngle
    math.pi * 1.5,   // sweepAngle
    false,           // useCenter
    paint,
  );

  // --- Кривые (Path) ---
  final path = Path()
    ..moveTo(0, size.height)
    ..quadraticBezierTo(
      size.width / 2, 0,      // контрольная точка
      size.width, size.height, // конечная точка
    );
  canvas.drawPath(path, paint);

  // --- Текст ---
  final textPainter = TextPainter(
    text: const TextSpan(text: 'Hello', style: TextStyle(fontSize: 24)),
    textDirection: TextDirection.ltr,
  )..layout();
  textPainter.paint(canvas, Offset(50, 50));

  // --- Изображения ---
  // canvas.drawImage(image, offset, paint);   // ui.Image
  // canvas.drawImageRect(image, src, dst, paint);

  // --- Clip ---
  canvas.clipRect(Rect.fromLTWH(0, 0, size.width / 2, size.height));
  // Теперь рисование только в левой половине
}
```

---

## 4. Paint — настройки кисти

```dart
final paint = Paint()
  ..color = Colors.blue
  ..strokeWidth = 3
  ..style = PaintingStyle.stroke          // контур или заливка
  ..strokeCap = StrokeCap.round           // форма конца линии
  ..strokeJoin = StrokeJoin.round         // соединение линий
  ..isAntiAlias = true                    // сглаживание
  ..maskFilter = const MaskFilter.blur(   // тень/размытие
      BlurStyle.normal, 8.0)
  ..shader = const LinearGradient(        // градиент
      colors: [Colors.blue, Colors.purple],
    ).createShader(Rect.fromLTWH(0, 0, 100, 100));
```

---

## 5. Анимированный CustomPainter

```dart
class AnimatedCirclePainter extends CustomPainter {
  final double progress; // 0.0 → 1.0
  final Color color;

  const AnimatedCirclePainter({required this.progress, required this.color});

  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = size.shortestSide / 2 - 10;

    // Фон
    canvas.drawCircle(
      center,
      radius,
      Paint()..color = Colors.grey.shade200,
    );

    // Прогресс
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -math.pi / 2,           // начало сверху
      2 * math.pi * progress, // дуга пропорционально прогрессу
      false,
      Paint()
        ..color = color
        ..strokeWidth = 8
        ..style = PaintingStyle.stroke
        ..strokeCap = StrokeCap.round,
    );
  }

  @override
  bool shouldRepaint(AnimatedCirclePainter old) =>
      old.progress != progress || old.color != color;
}

// Виджет
class CircularProgressIndicatorCustom extends StatefulWidget {
  const CircularProgressIndicatorCustom({super.key});

  @override
  State<CircularProgressIndicatorCustom> createState() => _State();
}

class _State extends State<CircularProgressIndicatorCustom>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 2),
    )..repeat();
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
      builder: (context, _) => CustomPaint(
        size: const Size(120, 120),
        painter: AnimatedCirclePainter(
          progress: _controller.value,
          color: HSVColor.fromAHSV(1, _controller.value * 360, 1, 1).toColor(),
        ),
      ),
    );
  }
}
```

---

## 6. shouldRepaint — оптимизация

```dart
@override
bool shouldRepaint(MyPainter oldDelegate) {
  // false — никогда не перерисовывать (статичная графика)
  // true — всегда перерисовывать
  // Правильно: сравнивать значимые поля:
  return oldDelegate.progress != progress ||
         oldDelegate.color != color;
}
```

---

## 7. RepaintBoundary для Canvas-виджетов

```dart
// Изолировать CustomPainter от перерисовки родителя
RepaintBoundary(
  child: CustomPaint(
    painter: ComplexChartPainter(data: chartData),
  ),
)
```

---

## 8. Реальный пример — LineChart

```dart
class LineChartPainter extends CustomPainter {
  final List<double> values; // нормализованные 0.0..1.0

  const LineChartPainter({required this.values});

  @override
  void paint(Canvas canvas, Size size) {
    if (values.isEmpty) return;

    final linePaint = Paint()
      ..color = Colors.blue
      ..strokeWidth = 2
      ..style = PaintingStyle.stroke
      ..strokeCap = StrokeCap.round
      ..strokeJoin = StrokeJoin.round;

    final fillPaint = Paint()
      ..shader = LinearGradient(
          begin: Alignment.topCenter,
          end: Alignment.bottomCenter,
          colors: [Colors.blue.withOpacity(0.3), Colors.transparent],
        ).createShader(Rect.fromLTWH(0, 0, size.width, size.height))
      ..style = PaintingStyle.fill;

    final path = Path();
    final fillPath = Path();

    for (int i = 0; i < values.length; i++) {
      final x = size.width * i / (values.length - 1);
      final y = size.height * (1 - values[i]);

      if (i == 0) {
        path.moveTo(x, y);
        fillPath.moveTo(x, size.height);
        fillPath.lineTo(x, y);
      } else {
        path.lineTo(x, y);
        fillPath.lineTo(x, y);
      }
    }

    fillPath
      ..lineTo(size.width, size.height)
      ..close();

    canvas.drawPath(fillPath, fillPaint);
    canvas.drawPath(path, linePaint);
  }

  @override
  bool shouldRepaint(LineChartPainter old) => old.values != values;
}
```

---

## 9. Типичные ошибки

| Ошибка                  | Причина                                       | Решение                                                  |
| ----------------------- | --------------------------------------------- | -------------------------------------------------------- |
| Постоянный перерисовка  | `shouldRepaint` всегда `true`                 | Сравнивать только значимые поля                          |
| Некорректные координаты | Система координат canvas — левый верхний угол | Ориентироваться на `size` параметр                       |
| Текст вне bounds        | Нет layout() перед paint()                    | Всегда `textPainter.layout()` до `textPainter.paint()`   |
| Медленный CustomPainter | Сложные операции каждый кадр                  | Кешировать через `picture`, использовать RepaintBoundary |

---

## 10. Рекомендации

1. **`shouldRepaint` правильно** — отсутствие оптимизации здесь убивает fps.
2. **`canvas.save()/restore()`** — для трансформаций в отдельных секциях.
3. **`Path`** для сложных форм — кривые Безье, произвольные контуры.
4. **`clipPath`** для масок и сложных форм.
5. **Готовые библиотеки** — `fl_chart`, `syncfusion_flutter_charts` для сложных графиков.
