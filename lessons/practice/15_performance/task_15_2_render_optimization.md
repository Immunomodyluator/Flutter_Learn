# 15.2: Оптимизация рендеринга — const и RepaintBoundary

> Project: FitMenu | Глава 15 — Performance

### 15.2: const-виджеты, RepaintBoundary и ListView.builder

🎯 **Цель шага:** Оптимизировать `RecipesListScreen` через `const` конструкторы, `RepaintBoundary` вокруг тяжёлых виджетов и `ListView.builder` вместо `ListView` для больших списков.

---

📝 **Техническое задание:**

**1. const-конструкторы повсюду:**
```dart
// ПЛОХО: создаёт новый объект при каждом rebuild
child: Padding(
  padding: EdgeInsets.all(16),     // ← нет const
  child: Text('Рецепты'),           // ← нет const
),

// ХОРОШО
child: const Padding(
  padding: EdgeInsets.all(16),
  child: Text('Рецепты'),
),
```

**2. ListView.builder вместо ListView с children:**
```dart
// ПЛОХО: все 200 RecipeCard создаются сразу
ListView(children: recipes.map((r) => RecipeCard(recipe: r)).toList()),

// ХОРОШО: создаются только видимые на экране
ListView.builder(
  itemCount: recipes.length,
  itemExtent: 120.0, // ← фиксированная высота ускоряет layout
  itemBuilder: (context, i) => RecipeCard(key: ValueKey(recipes[i].id), recipe: recipes[i]),
),
```

**3. RepaintBoundary для тяжёлых виджетов:**
```dart
// CalorieRingChart — CustomPainter, который не должен перерисовывать весь экран
RepaintBoundary(
  child: CalorieRingChart(
    consumed: 1240,
    target: 2000,
  ),
),
```

**4. Оптимизация NutritionFact Row:**
```dart
// БЫЛО: каждый Nutrition rebuild пересчитывает layout
Row(
  children: nutrients.map((n) => _NutrientBadge(n)).toList(),
)

// СТАЛО: const + itemExtent в ListView.separated
const _StaticNutritionRow(),
```

**5. Задание:** Сделай замер до/после оптимизации:
- Запусти `flutter run --profile`
- Открой DevTools → Performance
- Прокрути список 5 секунд
- Запиши средний FPS до и после изменений

---

✅ **Критерии приёмки:**
- [ ] Все виджеты без мутабельных параметров используют `const` конструктор
- [ ] `ListView.builder` с `itemExtent` используется для списка рецептов
- [ ] `RepaintBoundary` обёрнут вокруг `CalorieRingChart`
- [ ] `ValueKey(recipe.id)` назначен каждому элементу списка
- [ ] `flutter analyze` не выдаёт prefer_const_constructors warnings

---

💡 **Подсказка:** `flutter analyze --suggestions` покажет места для `const`. `SliverList.builder` (с `sliver: true`) эффективнее в `CustomScrollView`. `itemExtent` в `ListView.builder` — одно из самых мощных улучшений для длинных списков.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/presentation/widgets/recipe_tile.dart

class RecipeTile extends StatelessWidget {
  const RecipeTile({              // ← const конструктор
    super.key,
    required this.recipe,
    this.onTap,
  });

  final Recipe recipe;
  final VoidCallback? onTap;

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(       // ← изолирует перерисовку
      child: Card(
        clipBehavior: Clip.antiAlias,
        child: InkWell(
          onTap: onTap,
          child: SizedBox(
            height: 120,          // ← фиксированная высота для itemExtent
            child: Row(
              children: [
                _RecipeImage(imageUrl: recipe.imageUrl),
                Expanded(
                  child: _RecipeInfo(recipe: recipe),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}

// ВАЖНО: разбиваем на маленькие const-виджеты
class _RecipeImage extends StatelessWidget {
  const _RecipeImage({required this.imageUrl});
  final String imageUrl;
  @override Widget build(BuildContext context) {
    return CachedNetworkImage(
      imageUrl: imageUrl,
      width: 120,
      fit: BoxFit.cover,
    );
  }
}

// lib/features/recipes/presentation/recipes_list_screen.dart

ListView.builder(
  padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
  itemCount: recipes.length,
  itemExtent: 136,   // 120 высота + 16 padding
  itemBuilder: (_, index) => Padding(
    padding: const EdgeInsets.only(bottom: 8),
    child: RecipeTile(
      key: ValueKey(recipes[index].id),
      recipe: recipes[index],
    ),
  ),
),
```

</details>
