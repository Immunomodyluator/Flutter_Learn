# Квиз: Custom Painter

**Тема:** 18.1 — Custom Painter  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое `CustomPainter` и как его использовать?

- A) Класс для рисования кастомных анимаций
- B) Абстрактный класс с методом `paint(Canvas, Size)`; использовать через `CustomPaint` виджет
- C) Замена `Canvas` для Flutter Web
- D) `CustomPainter` только для Canvas 2D

<details>
<summary>Ответ</summary>

**B) Абстрактный класс → `CustomPaint` виджет**

```dart
class CirclePainter extends CustomPainter {
  final Color color;
  final double strokeWidth;

  CirclePainter({required this.color, this.strokeWidth = 2.0});

  @override
  void paint(Canvas canvas, Size size) {
    final paint = Paint()
      ..color = color
      ..strokeWidth = strokeWidth
      ..style = PaintingStyle.stroke;

    final center = Offset(size.width / 2, size.height / 2);
    final radius = math.min(size.width, size.height) / 2 - strokeWidth;

    canvas.drawCircle(center, radius, paint);
  }

  @override
  bool shouldRepaint(CirclePainter oldDelegate) =>
      color != oldDelegate.color || strokeWidth != oldDelegate.strokeWidth;
}

// Использование:
CustomPaint(
  size: const Size(200, 200),
  painter: CirclePainter(color: Colors.blue),
  child: const Center(child: Text('Custom!')),
)
```

</details>

---

### Вопрос 2 🟢

Как работает `Paint` объект и какие его основные свойства?

- A) `Paint` — неизменяемый объект
- B) `Paint` — настройки кисти: `color`, `strokeWidth`, `style` (fill/stroke), `shader`, `blendMode`, `strokeCap`, `strokeJoin`
- C) Один `Paint` объект на всё приложение
- D) `Paint` наследует от `Widget`

<details>
<summary>Ответ</summary>

**B) Мутабельный объект с настройками рисования**

```dart
// Заливка:
final fillPaint = Paint()
  ..color = Colors.blue
  ..style = PaintingStyle.fill;

// Обводка:
final strokePaint = Paint()
  ..color = Colors.red
  ..style = PaintingStyle.stroke
  ..strokeWidth = 3.0
  ..strokeCap = StrokeCap.round  // round, butt, square
  ..strokeJoin = StrokeJoin.round; // round, miter, bevel

// Градиент:
final gradientPaint = Paint()
  ..shader = const LinearGradient(
    colors: [Colors.blue, Colors.purple],
  ).createShader(Rect.fromLTWH(0, 0, 200, 200));

// Blur:
final blurPaint = Paint()
  ..maskFilter = const MaskFilter.blur(BlurStyle.normal, 10);

// AntiAlias (по умолчанию true):
paint.isAntiAlias = true;
```

</details>

---

### Вопрос 3 🟢

Как рисовать основные фигуры на Canvas?

- A) `Canvas.draw(Shape.circle, paint)`
- B) `drawCircle`, `drawRect`, `drawLine`, `drawOval`, `drawPath`, `drawRRect` — методы Canvas
- C) Только через `Path.addShape()`
- D) `Shape.render(canvas)`

<details>
<summary>Ответ</summary>

**B) Методы `draw*` на Canvas**

```dart
@override
void paint(Canvas canvas, Size size) {
  final paint = Paint()..color = Colors.blue;

  // Круг:
  canvas.drawCircle(const Offset(100, 100), 50, paint);

  // Прямоугольник:
  canvas.drawRect(const Rect.fromLTWH(10, 10, 100, 60), paint);

  // Скруглённый прямоугольник:
  canvas.drawRRect(
    RRect.fromRectAndRadius(
      const Rect.fromLTWH(10, 10, 100, 60),
      const Radius.circular(12),
    ),
    paint,
  );

  // Линия:
  canvas.drawLine(const Offset(0, 0), const Offset(200, 200), paint);

  // Овал:
  canvas.drawOval(const Rect.fromLTWH(50, 50, 150, 80), paint);

  // Дуга:
  canvas.drawArc(
    const Rect.fromLTWH(50, 50, 100, 100),
    -math.pi / 2,  // startAngle
    math.pi,       // sweepAngle
    false,         // useCenter
    paint,
  );
}
```

</details>

---

### Вопрос 4 🟡

Как строить сложные фигуры с помощью `Path`?

- A) `Path` только для линий
- B) `Path.moveTo`, `lineTo`, `cubicTo`, `quadraticBezierTo`, `arcTo`, `close` — построение контура; `canvas.drawPath(path, paint)`
- C) `Path.create(shapes: [Circle, Rect])`
- D) `Path` недоступен в debug режиме

<details>
<summary>Ответ</summary>

**B) Методы Path для построения контура**

```dart
@override
void paint(Canvas canvas, Size size) {
  final path = Path();

  // Треугольник:
  path.moveTo(size.width / 2, 0);       // вершина
  path.lineTo(size.width, size.height); // правый угол
  path.lineTo(0, size.height);          // левый угол
  path.close();                          // замкнуть

  // Кривая Безье (кубическая):
  final curvePath = Path()
    ..moveTo(0, size.height / 2)
    ..cubicTo(
      size.width * 0.25, 0,              // контрольная точка 1
      size.width * 0.75, size.height,    // контрольная точка 2
      size.width, size.height / 2,       // конечная точка
    );

  // Произвольная форма:
  final starPath = _createStarPath(size);

  canvas.drawPath(path, Paint()..color = Colors.blue);
  canvas.drawPath(curvePath, Paint()
    ..color = Colors.red
    ..style = PaintingStyle.stroke
    ..strokeWidth = 3);

  // Операции с путями:
  final combined = Path.combine(
    PathOperation.union, path, curvePath,
  );
}
```

</details>

---

### Вопрос 5 🟡

Как правильно реализовать `shouldRepaint`?

- A) Всегда возвращать `true`
- B) Сравнивать все параметры, влияющие на рисование; возвращать `true` только при изменениях — иначе лишние перерисовки
- C) Всегда возвращать `false`
- D) `shouldRepaint` вызывается только при смене темы

<details>
<summary>Ответ</summary>

**B) Сравнивать только значимые параметры**

```dart
class ProgressPainter extends CustomPainter {
  final double progress; // 0.0 - 1.0
  final Color activeColor;
  final Color backgroundColor;
  final double strokeWidth;

  const ProgressPainter({
    required this.progress,
    this.activeColor = Colors.blue,
    this.backgroundColor = Colors.grey,
    this.strokeWidth = 8.0,
  });

  @override
  void paint(Canvas canvas, Size size) {
    // ... рисование прогресс бара
  }

  // Корректная реализация:
  @override
  bool shouldRepaint(ProgressPainter oldDelegate) {
    return progress != oldDelegate.progress ||
           activeColor != oldDelegate.activeColor ||
           backgroundColor != oldDelegate.backgroundColor ||
           strokeWidth != oldDelegate.strokeWidth;
  }
}

// ПЛОХО: всегда true — перерисовывается при любом setState в дереве:
@override
bool shouldRepaint(_) => true;

// ПЛОХО: всегда false — не обновляется никогда:
@override
bool shouldRepaint(_) => false;
```

</details>

---

### Вопрос 6 🟡

Как трансформировать Canvas (translate, rotate, scale)?

- A) Трансформации применяются к виджету, не Canvas
- B) `canvas.save()`/`restore()` для изоляции; `canvas.translate()`, `canvas.rotate()`, `canvas.scale()` для трансформаций
- C) `canvas.transform(Matrix4.identity())`
- D) Трансформации только через виджеты Transform

<details>
<summary>Ответ</summary>

**B) `save/restore` + методы трансформации**

```dart
@override
void paint(Canvas canvas, Size size) {
  // Нарисовать объект, повёрнутый на 45 градусов:
  canvas.save(); // сохранить текущее состояние

  // Перенести начало координат в центр:
  canvas.translate(size.width / 2, size.height / 2);
  // Повернуть:
  canvas.rotate(math.pi / 4); // 45 градусов
  // Масштабировать:
  canvas.scale(0.8, 0.8);

  // Нарисовать от нового начала координат:
  canvas.drawRect(
    Rect.fromCenter(center: Offset.zero, width: 100, height: 60),
    Paint()..color = Colors.blue,
  );

  canvas.restore(); // восстановить состояние

  // saveLayer — для смешивания/эффектов (дороже):
  canvas.saveLayer(
    Rect.fromLTWH(0, 0, size.width, size.height),
    Paint()..blendMode = BlendMode.multiply,
  );
  // ... рисование
  canvas.restore();
}
```

</details>

---

### Вопрос 7 🟡

Как нарисовать текст на Canvas?

- A) `canvas.drawText('Hello', paint)`
- B) `TextPainter` — настройка + `layout()` + `paint(canvas, offset)`
- C) `Text('Hello').paint(canvas)`
- D) `canvas.drawString('Hello', x, y)`

<details>
<summary>Ответ</summary>

**B) `TextPainter.layout()` + `paint()`**

```dart
@override
void paint(Canvas canvas, Size size) {
  // Настроить TextPainter:
  final textPainter = TextPainter(
    text: TextSpan(
      text: '42%',
      style: TextStyle(
        color: Colors.white,
        fontSize: 24,
        fontWeight: FontWeight.bold,
      ),
    ),
    textDirection: TextDirection.ltr,
  );

  // Рассчитать layout:
  textPainter.layout(
    minWidth: 0,
    maxWidth: size.width,
  );

  // Нарисовать по центру:
  final offset = Offset(
    (size.width - textPainter.width) / 2,
    (size.height - textPainter.height) / 2,
  );
  textPainter.paint(canvas, offset);
}
```

</details>

---

### Вопрос 8 🔴

Как реализовать анимированный `CustomPainter`?

- A) `AnimatedCustomPainter` встроенный виджет
- B) Передать `AnimationController` как параметр + `repaint` в `super(repaint: animation)` — автоматический `repaint` при каждом кадре
- C) `Timer.periodic` в `paint()` методе
- D) `setState()` вызывает `shouldRepaint()` автоматически

<details>
<summary>Ответ</summary>

**B) `repaint:` параметр в `super()` для автоматических перерисовок**

```dart
class WavePainter extends CustomPainter {
  final Animation<double> animation;

  WavePainter({required this.animation})
      : super(repaint: animation); // ← ключевой момент!

  @override
  void paint(Canvas canvas, Size size) {
    final t = animation.value; // 0.0 → 1.0

    final path = Path();
    path.moveTo(0, size.height / 2);

    for (double x = 0; x <= size.width; x++) {
      final y = size.height / 2 +
          30 * math.sin((x / size.width * 2 * math.pi) + t * 2 * math.pi);
      path.lineTo(x, y);
    }

    canvas.drawPath(path, Paint()
      ..color = Colors.blue
      ..style = PaintingStyle.stroke
      ..strokeWidth = 2);
  }

  @override
  bool shouldRepaint(WavePainter old) => false;
  // false — потому что repaint: animation уже вызывает перерисовку
}

// Использование:
class WaveWidget extends StatefulWidget { ... }
class _WaveWidgetState extends State<WaveWidget>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this,
        duration: const Duration(seconds: 2))..repeat();
  }

  @override
  Widget build(BuildContext context) => CustomPaint(
    painter: WavePainter(animation: _controller),
  );
}
```

</details>

---

### Вопрос 9 🔴

Как реализовать hit testing для кастомного виджета?

- A) `CustomPainter` не может обрабатывать нажатия
- B) Переопределить `hitTest(Offset position)` → `true` если касание попадает в фигуру; или использовать `GestureDetector` поверх
- C) Все нажатия автоматически перехватываются по bounding box
- D) `canvas.addHitRegion(path)`

<details>
<summary>Ответ</summary>

**B) `hitTest()` метод CustomPainter**

```dart
class InteractiveCirclePainter extends CustomPainter {
  final List<CircleData> circles;
  final void Function(int index) onCircleTapped;

  InteractiveCirclePainter({required this.circles, required this.onCircleTapped});

  @override
  void paint(Canvas canvas, Size size) {
    for (final circle in circles) {
      canvas.drawCircle(circle.center, circle.radius,
          Paint()..color = circle.color);
    }
  }

  // Определить попал ли tap в фигуру:
  @override
  bool hitTest(Offset position) {
    for (int i = 0; i < circles.length; i++) {
      final circle = circles[i];
      final distance = (position - circle.center).distance;
      if (distance <= circle.radius) {
        onCircleTapped(i);
        return true;  // поглотить событие
      }
    }
    return false;  // не поглощать — передать дальше
  }

  @override
  bool shouldRepaint(_) => true;
}
```

</details>

---

### Вопрос 10 🔴

Как использовать `ImageShader` и `PictureRecorder` в CustomPainter?

- A) Эти классы недоступны в Flutter
- B) `ImageShader` — заливка изображением через `Paint.shader`; `PictureRecorder` — запись команд рисования для переиспользования или кэширования
- C) `ImageShader` только для WebGL
- D) `PictureRecorder` только в тестах

<details>
<summary>Ответ</summary>

**B) `ImageShader` для заливки; `PictureRecorder` для кэширования**

```dart
// ImageShader — заливка текстурой:
@override
void paint(Canvas canvas, Size size) {
  if (_image == null) return;

  final shader = ImageShader(
    _image!,
    TileMode.repeated,
    TileMode.repeated,
    Matrix4.identity().storage,
  );

  canvas.drawRect(
    Offset.zero & size,
    Paint()..shader = shader,
  );
}

// PictureRecorder — кэширование сложного рисования:
class CachedComplexPainter extends CustomPainter {
  Picture? _cachedPicture;
  Size? _cachedSize;

  @override
  void paint(Canvas canvas, Size size) {
    if (_cachedPicture == null || _cachedSize != size) {
      final recorder = PictureRecorder();
      final recordCanvas = Canvas(recorder);

      // Дорогостоящие операции — записать один раз:
      _drawComplexBackground(recordCanvas, size);

      _cachedPicture = recorder.endRecording();
      _cachedSize = size;
    }

    canvas.drawPicture(_cachedPicture!);
    // Динамическая часть — рисовать каждый кадр:
    _drawDynamicElements(canvas, size);
  }

  @override
  bool shouldRepaint(_) => true;
}
```

</details>
