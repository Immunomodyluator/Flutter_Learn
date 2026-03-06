# 2.3: Hot Reload и Hot Restart

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 2 — Настройка окружения Flutter

---

### 2.3: Hot Reload и Hot Restart

🎯 **Цель шага:** Понять разницу между Hot Reload (`r`) и Hot Restart (`R`) через практический эксперимент: менять UI без потери счётчика блюд и наблюдать жизненный цикл через `debugPrint`.

📝 **Техническое задание:**

Создай `MealCounterScreen` (`StatefulWidget`):

1. Состояние: `int _mealCount = 0`, `String _lastMeal = 'Нет'`
2. Кнопки добавления трёх типов блюд (завтрак/обед/ужин)
3. Добавь `debugPrint` в `initState`, `didUpdateWidget`, `dispose`

Проведи эксперименты и запиши результаты в `README.md`:
- **Эксперимент A:** Измени цвет кнопки → `r` → счётчик сохранился?
- **Эксперимент B:** Измени строку в `initState` → `r` → `initState` вызвался?
- **Эксперимент C:** `R` → счётчик сбросился в 0?

✅ **Критерии приёмки:**
- [ ] Hot Reload (`r`) сохраняет значение `_mealCount`
- [ ] Изменение цвета кнопки применяется без перезапуска
- [ ] `initState` НЕ вызывается при Hot Reload (только при Hot Restart / первом запуске)
- [ ] Hot Restart (`R`) сбрасывает `_mealCount = 0`
- [ ] В `README.md` есть таблица с результатами экспериментов

💡 **Подсказка:** Hot Reload обновляет только метод `build`. Всё что в `initState`, `static`-переменных или `main()` — требует Hot Restart. Добавление нового пакета → `flutter pub get` + Hot Restart.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_log/screens/meal_counter_screen.dart
import 'package:flutter/material.dart';

class MealCounterScreen extends StatefulWidget {
  const MealCounterScreen({super.key});

  @override
  State<MealCounterScreen> createState() => _MealCounterScreenState();
}

class _MealCounterScreenState extends State<MealCounterScreen> {
  int _mealCount = 0;
  String _lastMeal = 'Нет';

  @override
  void initState() {
    super.initState();
    // Попробуй изменить эту строку и нажми 'r' — не сработает (нужен 'R')
    debugPrint('🟢 initState: _mealCount=$_mealCount');
  }

  @override
  void didUpdateWidget(MealCounterScreen oldWidget) {
    super.didUpdateWidget(oldWidget);
    debugPrint('🔄 didUpdateWidget');
  }

  @override
  void dispose() {
    debugPrint('🔴 dispose');
    super.dispose();
  }

  void _add(String mealName) {
    setState(() {
      _mealCount++;
      _lastMeal = mealName;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        // Измени заголовок и нажми 'r' — счётчик сохранится!
        title: const Text('Дневник питания'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Блюд сегодня: $_mealCount',
                style: Theme.of(context).textTheme.headlineMedium),
            Text('Последнее: $_lastMeal',
                style: const TextStyle(color: Colors.grey)),
            const SizedBox(height: 32),
            // Измени цвет на Colors.orange и нажми 'r'
            FilledButton.icon(
              style: FilledButton.styleFrom(
                  backgroundColor: Colors.green), // ← меняй
              onPressed: () => _add('Завтрак'),
              icon: const Icon(Icons.breakfast_dining),
              label: const Text('Завтрак'),
            ),
            const SizedBox(height: 8),
            FilledButton.icon(
              onPressed: () => _add('Обед'),
              icon: const Icon(Icons.lunch_dining),
              label: const Text('Обед'),
            ),
            const SizedBox(height: 8),
            FilledButton.icon(
              style: FilledButton.styleFrom(
                  backgroundColor: Colors.indigo),
              onPressed: () => _add('Ужин'),
              icon: const Icon(Icons.dinner_dining),
              label: const Text('Ужин'),
            ),
          ],
        ),
      ),
    );
  }
}
```

```markdown
## Hot Reload vs Hot Restart — результаты экспериментов

| Изменение | Hot Reload `r` | Hot Restart `R` |
|-----------|---------------|-----------------|
| Цвет кнопки | ✅ Применяется, счётчик сохраняется | ✅ Применяется, счётчик = 0 |
| Текст AppBar title | ✅ Применяется, счётчик сохраняется | ✅ Применяется, счётчик = 0 |
| Строка в initState | ❌ НЕ применяется (initState не вызывается) | ✅ Применяется |
| Новый пакет в pubspec | ❌ НЕ работает | ✅ После `flutter pub get` |
| Изменение в main() | ❌ НЕ применяется | ✅ Применяется |

### Вывод
- **Hot Reload** = перекомпилировать изменённые файлы + обновить build(). Состояние сохраняется.
- **Hot Restart** = полный перезапуск dart VM. Состояние сбрасывается.
```

</details>
