# 4.1 Column и Row

## 1. Суть концепции

`Column` и `Row` — основные flex-виджеты для линейного расположения дочерних элементов. Column — вертикально, Row — горизонтально. Основаны на Flexbox-модели (аналог CSS Flexbox).

---

## 2. Оси

```
Row:
  → Main Axis (горизонталь)
  ↕ Cross Axis (вертикаль)

Column:
  ↕ Main Axis (вертикаль)
  → Cross Axis (горизонталь)
```

---

## 3. MainAxisAlignment и CrossAxisAlignment

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.center,     // цент по вертикали
  crossAxisAlignment: CrossAxisAlignment.stretch,  // растянуть по горизонтали
  children: [
    Container(height: 50, color: Colors.red),
    Container(height: 80, color: Colors.green),
    Container(height: 60, color: Colors.blue),
  ],
)

Row(
  mainAxisAlignment: MainAxisAlignment.spaceBetween,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    const Icon(Icons.home),
    const Text('Главная'),
    const Icon(Icons.arrow_forward),
  ],
)
```

---

## 4. mainAxisSize

```dart
// MainAxisSize.max (по умолчанию) — занять весь доступный размер по главной оси
Column(
  mainAxisSize: MainAxisSize.max,  // Column займёт всю высоту
  children: [...],
)

// MainAxisSize.min — занять минимально необходимый размер
Column(
  mainAxisSize: MainAxisSize.min,  // Column сожмётся по содержимому
  children: [
    const Text('Строка 1'),
    const Text('Строка 2'),
  ],
)
// Полезно, например, для BottomSheet
```

---

## 5. Expanded и Flexible

```dart
// Expanded — ребёнок занимает ВЕСЬ оставшийся размер
Column(
  children: [
    Container(height: 100, color: Colors.red),
    Expanded(                          // займёт всё оставшееся
      child: Container(color: Colors.green),
    ),
    Container(height: 100, color: Colors.blue),
  ],
)

// Flexible с flex — пропорциональное распределение
Row(
  children: [
    Flexible(flex: 1, child: Container(color: Colors.red)),   // 1/4
    Flexible(flex: 3, child: Container(color: Colors.green)), // 3/4
  ],
)

// Несколько Expanded
Row(
  children: [
    Expanded(flex: 2, child: Container(color: Colors.red)),   // 2/3
    Expanded(flex: 1, child: Container(color: Colors.blue)),  // 1/3
  ],
)
```

---

## 6. Реальный пример: экран профиля

```dart
class ProfileScreen extends StatelessWidget {
  const ProfileScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Шапка профиля
        const Padding(
          padding: EdgeInsets.all(16),
          child: Row(
            children: [
              CircleAvatar(radius: 40, child: Icon(Icons.person, size: 40)),
              SizedBox(width: 16),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text('Иван Иванов', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
                    SizedBox(height: 4),
                    Text('ivan@example.com', style: TextStyle(color: Colors.grey)),
                  ],
                ),
              ),
              IconButton(icon: Icon(Icons.edit), onPressed: null),
            ],
          ),
        ),

        // Статистика
        const Padding(
          padding: EdgeInsets.symmetric(horizontal: 16),
          child: Row(
            children: [
              _StatCard(label: 'Публикации', value: '42'),
              SizedBox(width: 8),
              _StatCard(label: 'Подписчики', value: '1.2K'),
              SizedBox(width: 8),
              _StatCard(label: 'Подписан', value: '156'),
            ],
          ),
        ),

        // Список постов
        Expanded(
          child: ListView.builder(
            itemCount: 20,
            itemBuilder: (_, i) => ListTile(title: Text('Пост #$i')),
          ),
        ),
      ],
    );
  }
}

class _StatCard extends StatelessWidget {
  final String label;
  final String value;
  const _StatCard({required this.label, required this.value});

  @override
  Widget build(BuildContext context) {
    return Expanded(
      child: Card(
        child: Padding(
          padding: const EdgeInsets.all(12),
          child: Column(
            children: [
              Text(value, style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold)),
              Text(label, style: const TextStyle(color: Colors.grey, fontSize: 12)),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## 7. Типичные ошибки

| Ошибка                                            | Ошибка в рантайме                              | Решение                                             |
| ------------------------------------------------- | ---------------------------------------------- | --------------------------------------------------- |
| `ListView` в `Column`                             | `Vertical viewport was given unbounded height` | `Expanded(child: ListView(...))`                    |
| `Column` внутри `Column` с неограниченной высотой | `RenderFlex overflow`                          | Добавить `Expanded` или `SingleChildScrollView`     |
| `Row` без `Expanded` для текста                   | Overflow по горизонтали                        | `Expanded(child: Text(...))`                        |
| `Column` в `SingleChildScrollView` с `Expanded`   | Expanded не работает без ограниченной высоты   | Убрать `Expanded`, использовать `mainAxisSize: min` |

---

## 8. Практические рекомендации

1. **`const SizedBox(width: N)` / `SizedBox(height: N)`** для отступов между элементами.
2. **`CrossAxisAlignment.stretch`** в Column чтобы кнопки занимали всю ширину.
3. **`MainAxisSize.min`** в Column/Row внутри BottomSheet для корректного размера.
4. **`Expanded` обязателен** перед `ListView`, `GridView` в Column/Row.
5. **Проверяй overflow** — Flutter показывает чёрно-жёлтые полоски при переполнении.
