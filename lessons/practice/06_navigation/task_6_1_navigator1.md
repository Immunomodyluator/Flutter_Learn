# 6.1: Navigator 1.0 — переход на экран деталей рецепта

> Project: FitMenu | Глава 6 — Navigation

### 6.1: Императивная навигация через Navigator.push и pop

🎯 **Цель шага:** Реализовать переход с экрана списка рецептов на экран деталей рецепта с передачей данных через конструктор — используя классический `Navigator.push` и `MaterialPageRoute`.

---

📝 **Техническое задание:**

Реализуй экран деталей в `lib/features/recipes/screens/recipe_detail_screen.dart`.

**RecipeDetailScreen принимает:**
```dart
class RecipeDetailScreen extends StatelessWidget {
  const RecipeDetailScreen({super.key, required this.recipe});
  final Recipe recipe;
}
```

**Содержимое экрана:**
- `CustomScrollView` с `SliverAppBar(expandedHeight: 280, pinned: true)`
- Фото рецепта в `flexibleSpace`
- Кнопка "← Назад" (автоматически из AppBar) вызывает `Navigator.pop(context)`
- Секции: описание, КБЖУ (NutritionStatsBar из гл.4), список ингредиентов, шаги приготовления
- FAB: "Добавить в план питания" → `Navigator.pop(context, true)` (возвращает результат)

**Переход из RecipesScreen:**
```dart
// При нажатии на RecipeTile:
final added = await Navigator.push<bool>(
  context,
  MaterialPageRoute(
    builder: (_) => RecipeDetailScreen(recipe: recipe),
  ),
);
if (added == true) {
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(content: Text('${recipe.title} добавлен в план!')),
  );
}
```

**Анимация перехода:** добавь кастомную `PageRouteBuilder` с `FadeTransition` (опционально).

---

✅ **Критерии приёмки:**
- [ ] Переход на детали осуществляется через `Navigator.push` (не `Navigator.pushNamed`)
- [ ] Данные рецепта передаются через конструктор (не через глобальное состояние)
- [ ] FAB возвращает `true` через `Navigator.pop(context, true)`
- [ ] SnackBar появляется на списке после возврата с результатом `true`
- [ ] AppBar с кнопкой назад работает корректно
- [ ] Экран не вызывает утечку памяти (StatelessWidget, нет подписок без отписки)

---

💡 **Подсказка:** `Navigator.push<T>` — generic, T это тип возвращаемого значения. `await Navigator.push<bool>(...)` вернёт `null` если пользователь нажал системную кнопку назад (не FAB). Проверяй `if (mounted && added == true)` перед показом SnackBar в StatefulWidget.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/screens/recipe_detail_screen.dart

import 'package:flutter/material.dart';
// import 'package:fitmenu/features/recipes/widgets/recipe_card.dart';

class RecipeDetailScreen extends StatelessWidget {
  const RecipeDetailScreen({super.key, required this.recipe});

  final Recipe recipe;

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          SliverAppBar(
            expandedHeight: 280,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: Text(
                recipe.title,
                style: const TextStyle(fontSize: 14),
                maxLines: 1,
                overflow: TextOverflow.ellipsis,
              ),
              background: Image.network(
                recipe.imageUrl,
                fit: BoxFit.cover,
              ),
            ),
          ),

          SliverToBoxAdapter(
            child: Padding(
              padding: const EdgeInsets.all(16),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // КБЖУ
                  Row(
                    mainAxisAlignment: MainAxisAlignment.spaceAround,
                    children: [
                      _MacroChip(label: 'Калории', value: '${recipe.calories} ккал', color: Colors.orange),
                      _MacroChip(label: 'Время', value: '${recipe.cookingTimeMinutes} мин', color: Colors.blue),
                    ],
                  ),
                  const SizedBox(height: 16),

                  Text('Описание', style: Theme.of(context).textTheme.titleMedium),
                  const SizedBox(height: 8),
                  const Text(
                    'Вкусное и полезное блюдо, богатое белком и витаминами. '
                    'Подходит для правильного питания и диетического рациона.',
                  ),

                  const SizedBox(height: 16),
                  Text('Ингредиенты', style: Theme.of(context).textTheme.titleMedium),
                  const SizedBox(height: 8),
                  ...['Куриное филе 200г', 'Брокколи 150г', 'Оливковое масло 1 ст.л.']
                      .map((i) => Padding(
                            padding: const EdgeInsets.only(bottom: 4),
                            child: Row(
                              children: [
                                const Icon(Icons.circle, size: 6),
                                const SizedBox(width: 8),
                                Text(i),
                              ],
                            ),
                          )),

                  const SizedBox(height: 80), // место под FAB
                ],
              ),
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton.extended(
        onPressed: () => Navigator.pop(context, true),
        icon: const Icon(Icons.add),
        label: const Text('В план питания'),
      ),
    );
  }
}

class _MacroChip extends StatelessWidget {
  const _MacroChip({
    required this.label,
    required this.value,
    required this.color,
  });

  final String label;
  final String value;
  final Color color;

  @override
  Widget build(BuildContext context) {
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
      decoration: BoxDecoration(
        color: color.withOpacity(0.1),
        borderRadius: BorderRadius.circular(12),
        border: Border.all(color: color.withOpacity(0.3)),
      ),
      child: Column(
        children: [
          Text(value,
              style: TextStyle(
                  color: color, fontWeight: FontWeight.bold, fontSize: 16)),
          Text(label, style: Theme.of(context).textTheme.labelSmall),
        ],
      ),
    );
  }
}
```

</details>
