# 4.3 Container и SizedBox

## 1. Суть концепции

`Container` — многофункциональный виджет: объединяет отступы, декорацию, выравнивание, трансформацию и ограничения размера. `SizedBox` — минималистичный контейнер только для задания размера или отступов.

**Правило**: используй наименее мощный виджет для задачи.

---

## 2. Container — все возможности

```dart
Container(
  // Размеры
  width: 200,
  height: 100,
  constraints: const BoxConstraints(
    minWidth: 100,
    maxWidth: 300,
    minHeight: 50,
    maxHeight: 200,
  ),

  // Отступы
  margin: const EdgeInsets.all(16),
  padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),

  // Выравнивание содержимого
  alignment: Alignment.center,

  // Цвет (не использовать вместе с decoration)
  // color: Colors.blue,

  // Декорация (заменяет color)
  decoration: BoxDecoration(
    color: Colors.blue.shade100,
    borderRadius: BorderRadius.circular(12),
    border: Border.all(color: Colors.blue, width: 2),
    boxShadow: const [
      BoxShadow(
        color: Colors.black26,
        blurRadius: 8,
        spreadRadius: 2,
        offset: Offset(0, 4),
      ),
    ],
    image: const DecorationImage(
      image: NetworkImage('https://picsum.photos/200'),
      fit: BoxFit.cover,
    ),
  ),

  // Трансформация
  transform: Matrix4.rotationZ(0.05),
  transformAlignment: Alignment.center,

  child: const Text('Контент'),
)
```

---

## 3. Частые паттерны с Container

```dart
// Округлённая карточка
Container(
  decoration: BoxDecoration(
    color: Colors.white,
    borderRadius: BorderRadius.circular(16),
    boxShadow: [BoxShadow(color: Colors.black12, blurRadius: 8)],
  ),
  child: ...,
)

// Градиентный фон
Container(
  decoration: const BoxDecoration(
    gradient: LinearGradient(
      colors: [Color(0xFF6B4EFF), Color(0xFFFF4E9A)],
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
    ),
  ),
  child: ...,
)

// Круг
Container(
  width: 60,
  height: 60,
  decoration: const BoxDecoration(
    color: Colors.blue,
    shape: BoxShape.circle,
  ),
  child: const Icon(Icons.person, color: Colors.white),
)
```

---

## 4. SizedBox — минималистичный контейнер

```dart
// Фиксированный размер
SizedBox(width: 200, height: 50, child: ElevatedButton(...))

// Заполнить доступное пространство (в Expanded)
const SizedBox.expand(child: Text('Заполнить'))

// Пустой отступ (предпочтительно для разделения элементов)
const SizedBox(height: 16)  // вертикальный отступ
const SizedBox(width: 8)    // горизонтальный отступ

// Скрыть виджет с сохранением места
SizedBox.shrink(child: AnimatedWidget())  // 0x0

// Заданный размер без содержимого
const SizedBox(height: 48)  // кнопка-заглушка по высоте
```

---

## 5. Когда что использовать

| Задача                   | Виджет                                      |
| ------------------------ | ------------------------------------------- |
| Цвет фона                | `ColoredBox(color: ..., child: ...)`        |
| Отступ                   | `Padding(padding: ..., child: ...)`         |
| Выравнивание             | `Align(alignment: ..., child: ...)`         |
| Размер                   | `SizedBox(width: ..., height: ...)`         |
| Скругление + тень + цвет | `Container(decoration: BoxDecoration(...))` |
| Несколько свойств сразу  | `Container`                                 |

```dart
// ❌ Избыточно
Container(
  padding: const EdgeInsets.all(16),
  child: Text('Hello'),
)

// ✅ Достаточно
Padding(
  padding: const EdgeInsets.all(16),
  child: Text('Hello'),
)
```

---

## 6. Реальный пример: кастомная кнопка

```dart
class GradientButton extends StatelessWidget {
  final String label;
  final VoidCallback onPressed;

  const GradientButton({super.key, required this.label, required this.onPressed});

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onPressed,
      child: Container(
        height: 52,
        decoration: BoxDecoration(
          gradient: const LinearGradient(
            colors: [Color(0xFF6B4EFF), Color(0xFFAB5EFF)],
          ),
          borderRadius: BorderRadius.circular(26),
          boxShadow: const [
            BoxShadow(
              color: Color(0x556B4EFF),
              blurRadius: 12,
              offset: Offset(0, 6),
            ),
          ],
        ),
        child: Center(
          child: Text(
            label,
            style: const TextStyle(
              color: Colors.white,
              fontWeight: FontWeight.bold,
              fontSize: 16,
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## 7. Типичные ошибки

| Ошибка                                                              | Проблема                          | Решение                                                |
| ------------------------------------------------------------------- | --------------------------------- | ------------------------------------------------------ |
| `color` + `decoration` в одном Container                            | Ошибка компиляции                 | Цвет указывать только в `BoxDecoration.color`          |
| Container без child и без constraints                               | Займёт всё доступное пространство | Задать явный размер или `mainAxisSize: min` в родителе |
| `BoxDecoration` без `borderRadius` для `shape: rectangle` (default) | Прямые углы                       | Добавить `borderRadius`                                |
| Глубокое вложение Container                                         | Ухудшает читаемость               | Извлекать в именованные виджеты                        |

---

## 8. Практические рекомендации

1. **`SizedBox` для пустых отступов** — быстрее и понятнее чем `Padding` без child.
2. **Не используй `color:` и `decoration:` вместе** — будет ошибка.
3. **`ColoredBox` быстрее `Container`** для простой заливки цветом.
4. **Именуй сложные контейнеры** как отдельные виджеты — `AvatarBorder`, не безымянный Container.
5. **`BoxConstraints.tight(size)`** для точного размера через constraints.
