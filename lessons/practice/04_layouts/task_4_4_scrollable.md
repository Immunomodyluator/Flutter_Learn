# 4.4: Прокручиваемые виджеты — лента рецептов с SliverAppBar

> Project: FitMenu | Глава 4 — Layouts

### 4.4: RecipesScreen с ListView.separated, GridView и SliverAppBar

🎯 **Цель шага:** Построить главный экран рецептов FitMenu с прокручиваемым заголовком (SliverAppBar), горизонтальной лентой "Рекомендованных" (ListView горизонтальный) и вертикальным списком всех рецептов (ListView.separated) — отработав Sliver-архитектуру и эффективный рендеринг длинных списков.

---

📝 **Техническое задание:**

Реализуй экран в файле `lib/features/recipes/screens/recipes_screen.dart`.

**Структура экрана:**
```
CustomScrollView
  ├── SliverAppBar (expandedHeight: 200, pinned: true, flexibleSpace: фото-заголовок)
  ├── SliverToBoxAdapter → Section "Рекомендованные" + горизонтальный ListView (RecipeCard)
  └── SliverList → вертикальный список RecipeTile с Divider
```

**SliverAppBar:**
- `expandedHeight: 200`
- `pinned: true` — остаётся видимым при прокрутке
- `flexibleSpace: FlexibleSpaceBar(title: Text("Рецепты"), background: Image.network(...))`
- `actions: [IconButton(Icons.search, ...)]`

**Горизонтальная лента:**
- `ListView.builder` с `scrollDirection: Axis.horizontal`
- Высота зафиксирована: `SizedBox(height: 220)`
- Использует `RecipeCard` из задания 4.2

**Вертикальный список (SliverList):**
- `SliverList.separated` (Flutter 3.7+) или `SliverList(delegate: SliverChildBuilderDelegate(...))`
- Использует `RecipeTile` из задания 4.3
- Разделитель: `Divider(height: 1)`

**Заглушка данных** (hardcoded список из 5–10 рецептов для отображения).

---

✅ **Критерии приёмки:**
- [ ] `SliverAppBar` сворачивается при прокрутке и остаётся как обычный AppBar
- [ ] Горизонтальная лента прокручивается независимо от вертикальной
- [ ] Вертикальный список использует `ListView.builder` или `SliverList` (не `Column` + `map`)
- [ ] Нет вложенных `ListView` без `shrinkWrap` (проверь: нет `shrinkWrap: true` в вертикальном списке)
- [ ] Весь экран — `CustomScrollView` (не `Column + SingleChildScrollView`)
- [ ] Экран отображается без ошибки на эмуляторе/физическом устройстве

---

💡 **Подсказка:** `CustomScrollView` — контейнер для Sliver-виджетов. `SliverToBoxAdapter` позволяет вставить обычный (`BoxWidget`) виджет внутрь `CustomScrollView`. Никогда не используй `shrinkWrap: true` во вложенных `ListView` в `CustomScrollView` — это убивает производительность. Вместо этого используй `SliverList`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/screens/recipes_screen.dart

import 'package:flutter/material.dart';
// import 'package:fitmenu/features/recipes/widgets/recipe_card.dart';
// import 'package:fitmenu/features/recipes/widgets/recipe_tile.dart';

// Заглушка данных
final _mockRecipes = List.generate(
  10,
  (i) => Recipe(
    id: 'r$i',
    title: 'Рецепт ${i + 1}: Куриный суп с овощами',
    imageUrl: 'https://picsum.photos/seed/recipe$i/400/300',
    calories: 300 + i * 30,
    cookingTimeMinutes: 20 + i * 5,
  ),
);

final _recommended = _mockRecipes.take(5).toList();

class RecipesScreen extends StatelessWidget {
  const RecipesScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: CustomScrollView(
        slivers: [
          // Складывающийся AppBar
          SliverAppBar(
            expandedHeight: 200,
            pinned: true,
            flexibleSpace: FlexibleSpaceBar(
              title: const Text('Рецепты'),
              background: Image.network(
                'https://picsum.photos/seed/header/800/400',
                fit: BoxFit.cover,
              ),
            ),
            actions: [
              IconButton(
                icon: const Icon(Icons.search),
                onPressed: () {},
              ),
            ],
          ),

          // Секция "Рекомендованные"
          SliverToBoxAdapter(
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Padding(
                  padding: const EdgeInsets.fromLTRB(16, 16, 16, 8),
                  child: Text(
                    'Рекомендованные',
                    style: Theme.of(context).textTheme.titleMedium?.copyWith(
                          fontWeight: FontWeight.bold,
                        ),
                  ),
                ),
                SizedBox(
                  height: 220,
                  child: ListView.builder(
                    scrollDirection: Axis.horizontal,
                    padding: const EdgeInsets.symmetric(horizontal: 12),
                    itemCount: _recommended.length,
                    itemBuilder: (context, index) => Padding(
                      padding: const EdgeInsets.symmetric(horizontal: 4),
                      child: RecipeCard(
                        recipe: _recommended[index],
                        onTap: () {},
                      ),
                    ),
                  ),
                ),
                Padding(
                  padding: const EdgeInsets.fromLTRB(16, 16, 16, 8),
                  child: Text(
                    'Все рецепты',
                    style: Theme.of(context).textTheme.titleMedium?.copyWith(
                          fontWeight: FontWeight.bold,
                        ),
                  ),
                ),
              ],
            ),
          ),

          // Вертикальный список рецептов (Sliver)
          SliverList.separated(
            itemCount: _mockRecipes.length,
            separatorBuilder: (_, __) => const Divider(height: 1),
            itemBuilder: (context, index) => RecipeTile(
              recipe: _mockRecipes[index],
              onTap: () {},
              onAddToMealPlan: () {},
              tags: const ['#суп', '#курица'],
            ),
          ),
        ],
      ),
    );
  }
}
```

</details>
