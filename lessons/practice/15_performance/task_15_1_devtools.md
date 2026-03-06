# 15.1: DevTools — профилирование RecipesList

> Project: FitMenu | Глава 15 — Performance

### 15.1: Профилирование и диагностика с Flutter DevTools

🎯 **Цель шага:** Использовать Flutter DevTools для профилирования экрана `RecipesListScreen`, выявить jank (пропуск кадров), утечки памяти и избыточные перестройки виджетов.

---

📝 **Техническое задание:**

**1. Включить диагностические флаги в `main.dart`:**
```dart
void main() {
  // Только в Debug-режиме
  if (kDebugMode) {
    debugPaintSizeEnabled   = false; // рисует границы виджетов
    debugRepaintRainbowEnabled = false; // подсвечивает перерисовки
    debugProfileBuildsEnabled  = false; // профилирует build()
  }
  runApp(const FitMenuApp());
}
```

**2. Найти и исправить 3 типичных проблемы:**

**Проблема А — лишний rebuild:**
```dart
// ПЛОХО: весь Consumer перестраивается при любом изменении
Consumer<RecipesNotifier>(
  builder: (context, notifier, child) => ListView.builder(
    itemCount: notifier.recipes.length,
    itemBuilder: (_, i) => RecipeCard(recipe: notifier.recipes[i]),
  ),
);

// ХОРОШО: child не перестраивается
Consumer<RecipesNotifier>(
  builder: (context, notifier, child) => ListView.builder(
    itemCount: notifier.recipes.length,
    itemBuilder: (_, i) => RecipeCard(recipe: notifier.recipes[i]),
  ),
  child: const _HeaderWidget(), // не зависит от notifier
);
```

**Проблема Б — утечка анимационного контроллера:**
```dart
class _RecipeCardState extends State<RecipeCard>
    with SingleTickerProviderStateMixin {
  late final AnimationController _controller;
  @override void initState()  { super.initState(); _controller = AnimationController(vsync: this, ...); }
  @override void dispose()    { _controller.dispose(); super.dispose(); } // ← ОБЯЗАТЕЛЬНО
}
```

**Проблема В — тяжёлый build в scrollable:**
```dart
// ПЛОХО: строим полный виджет в каждом кадре
itemBuilder: (_, i) => RecipeCard(recipe: recipes[i]);

// ХОРОШО: использовать ключ для переиспользования
itemBuilder: (_, i) => RecipeCard(key: ValueKey(recipes[i].id), recipe: recipes[i]);
```

**3. Задание:** открой DevTools → Performance → нажми "Record" → прокрути RecipesList 10 раз → Stop. Найди frames > 16ms и выясни причину.

---

✅ **Критерии приёмки:**
- [ ] `debugRepaintRainbowEnabled = true` временно включается для диагностики перерисовок
- [ ] Экран `RecipesListScreen` скроллится без "красных кадров" в DevTools Performance
- [ ] В Widget Rebuilds Inspector нет виджетов с rebuild count > 10 на один scroll
- [ ] Все `AnimationController` вызывают `dispose()`
- [ ] `ValueKey` используется в `ListView.builder`

---

💡 **Подсказка:** DevTools → Flutter Inspector → "Highlight Repaints" показывает перерисовки без кода. Performance overlay (`showPerformanceOverlay: true` в MaterialApp) виден прямо на устройстве. Желтый/красный слой = jank.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/main.dart

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

void main() {
  if (kProfileMode) {
    // Запуск в режиме профилирования: flutter run --profile
    runApp(const FitMenuApp());
  } else {
    runApp(const FitMenuApp());
  }
}

// lib/features/recipes/presentation/recipes_list_screen.dart

class RecipesListScreen extends StatelessWidget {
  const RecipesListScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Consumer<RecipesNotifier>(
      builder: (context, notifier, child) {
        if (notifier.isLoading) {
          return const Center(child: CircularProgressIndicator());
        }
        return CustomScrollView(
          slivers: [
            // Константный header не перестраивается
            child!, // ← передаётся снаружи
            SliverPadding(
              padding: const EdgeInsets.all(16),
              sliver: SliverList.builder(
                itemCount: notifier.recipes.length,
                itemBuilder: (context, index) {
                  final recipe = notifier.recipes[index];
                  // ValueKey гарантирует правильный порядок при обновлении
                  return RecipeTile(
                    key: ValueKey(recipe.id),
                    recipe: recipe,
                  );
                },
              ),
            ),
          ],
        );
      },
      // child НЕ перестраивается при изменении notifier
      child: const SliverAppBar(
        title: Text('Рецепты'),
        pinned: true,
        expandedHeight: 160,
      ),
    );
  }
}
```

</details>
