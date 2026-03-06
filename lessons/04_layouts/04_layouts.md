# 4. Лейауты и позиционирование

## 1. Суть концепции

Layout во Flutter — это процесс, при котором каждый виджет получает **ограничения** (constraints) от родителя, вычисляет свой размер и позиционирует дочерние виджеты. Ключевой принцип:

> **"Constraints go down. Sizes go up. Parent sets position."**

Родитель передаёт ограничения вниз → ребёнок сообщает свой размер вверх → родитель устанавливает позицию.

---

## 2. Типы ограничений (BoxConstraints)

```dart
// Tight — фиксированный размер (width == maxWidth, height == maxHeight)
// Loose — диапазон (0 <= width <= maxWidth)
// Unbounded — бесконечный (maxWidth == double.infinity)

// Просмотр через LayoutBuilder
LayoutBuilder(
  builder: (context, constraints) {
    print(constraints); // BoxConstraints(w=375.0, h=812.0)
    return Container(width: constraints.maxWidth);
  },
)
```

---

## 3. Column и Row — оси

```dart
// Column — вертикальная ось (main axis = vertical)
Column(
  mainAxisAlignment: MainAxisAlignment.spaceBetween, // вдоль главной оси
  crossAxisAlignment: CrossAxisAlignment.start,       // вдоль поперечной оси
  mainAxisSize: MainAxisSize.min,  // min = сжаться, max = расширить (по умолчанию)
  children: [...],
)

// Row — горизонтальная ось (main axis = horizontal)
Row(
  mainAxisAlignment: MainAxisAlignment.center,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [...],
)
```

| MainAxisAlignment | Поведение                          |
| ----------------- | ---------------------------------- |
| `start`           | У начала                           |
| `end`             | У конца                            |
| `center`          | По центру                          |
| `spaceBetween`    | Равные промежутки между элементами |
| `spaceAround`     | Промежутки + половины по краям     |
| `spaceEvenly`     | Равные промежутки везде            |

---

## 4. Flex — гибкое распределение пространства

```dart
// Expanded — занять всё оставшееся пространство
Row(
  children: [
    const SizedBox(width: 80, child: Text('Label')),
    Expanded(child: TextField()),  // займёт всё остальное
  ],
)

// Flexible — занять доступное, но не более собственного размера
Row(
  children: [
    Flexible(flex: 2, child: Container(color: Colors.red)),  // 2/3
    Flexible(flex: 1, child: Container(color: Colors.blue)), // 1/3
  ],
)

// Spacer — пустое пространство
Row(
  children: [
    Text('Слева'),
    const Spacer(),     // равно Expanded(child: SizedBox())
    Text('Справа'),
  ],
)
```

---

## 5. Stack и наложение

```dart
// Stack — виджеты поверх друг друга
Stack(
  alignment: Alignment.center,  // выравнивание непозиционированных детей
  children: [
    Image.network('https://...'),
    // Positioned — задаёт точную позицию
    Positioned(
      bottom: 16,
      right: 16,
      child: FloatingActionButton(onPressed: () {}, child: const Icon(Icons.edit)),
    ),
    // Без Positioned — позиционируется по alignment
    Container(color: Colors.black38, child: const Text('Overlay')),
  ],
)
```

---

## 6. Container — универсальный бокс

```dart
Container(
  width: 200,
  height: 100,
  margin: const EdgeInsets.all(16),
  padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
  decoration: BoxDecoration(
    color: Colors.blue.shade100,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.blue, width: 2),
    boxShadow: const [
      BoxShadow(color: Colors.black26, blurRadius: 8, offset: Offset(0, 4)),
    ],
    gradient: const LinearGradient(
      colors: [Colors.blue, Colors.purple],
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
    ),
  ),
  child: const Text('Контент'),
)
```

---

## 7. Типичные ошибки лейаута

| Ошибка                                    | Сообщение / Проблема    | Решение                                       |
| ----------------------------------------- | ----------------------- | --------------------------------------------- |
| Column в Column без ограничений           | `RenderFlex overflowed` | `Expanded` или `SingleChildScrollView`        |
| ListView в Column                         | Unbounded height        | `Expanded(child: ListView(...))`              |
| `Container(width: double.infinity)` в Row | RenderFlex overflow     | `Expanded(child: Container(...))`             |
| Stack без размера                         | Нулевой размер          | Задать `width`/`height` или `Positioned.fill` |

---

## 8. Практические рекомендации

1. **Читай сообщения об ошибках** — Flutter объясняет причину overflow и предлагает решения.
2. **`const SizedBox(height: 8)`** вместо `Padding` для простых отступов.
3. **`LayoutBuilder`** для адаптивного UI — даёт constraints родителя.
4. **`Expanded` vs `Flexible`**: Expanded = `Flexible(fit: FlexFit.tight)`, занимает всё; Flexible — занимает нужное.
5. **Не злоупотребляй `Container`** — чаще нужен `Padding`, `ColoredBox`, или `DecoratedBox`.
