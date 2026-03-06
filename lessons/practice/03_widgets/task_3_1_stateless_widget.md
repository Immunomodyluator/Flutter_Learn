# 3.1: StatelessWidget

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 3 — Основы виджетов

---

### 3.1: StatelessWidget

🎯 **Цель шага:** Создать набор переиспользуемых `StatelessWidget`-компонентов FitMenu: `NutritionBadge`, `CategoryChip` и `RecipeCard` — неизменяемые, `const`-совместимые «кирпичики» UI.

📝 **Техническое задание:**

Создай три виджета в `lib/features/recipes/widgets/`:

1. **`NutritionBadge`** — строчный виджет: иконка + значение + единица. Параметры: `double value`, `String unit`, `IconData icon`, `Color color`
2. **`CategoryChip`** — цветная плашка категории: разные цвета и эмодзи для `'завтрак'`/`'обед'`/`'ужин'`/`'перекус'`
3. **`RecipeCard`** — карточка рецепта, использует `NutritionBadge` и `CategoryChip` внутри. Параметры: `Recipe recipe`, `VoidCallback? onTap`, `VoidCallback? onFavoriteTap`

Все конструкторы — `const`.

✅ **Критерии приёмки:**
- [ ] Все три виджета — `StatelessWidget` (нет `setState`, нет изменяемых полей)
- [ ] Конструкторы помечены `const` (компилятор не выдаёт предупреждение)
- [ ] `RecipeCard` ничего не знает о бизнес-логике — только отображает переданные данные
- [ ] `NutritionBadge` корректно форматирует `double` через `toStringAsFixed(0)`
- [ ] `flutter analyze` — 0 issues

💡 **Подсказка:** `const` конструктор требует чтобы все поля были `final`. Если `CategoryChip` использует `Map` — объяви его как `static const` внутри класса, иначе `const` конструктор невозможен.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/widgets/nutrition_badge.dart
import 'package:flutter/material.dart';

class NutritionBadge extends StatelessWidget {
  final double value;
  final String unit;
  final IconData icon;
  final Color color;

  const NutritionBadge({
    super.key,
    required this.value,
    required this.unit,
    required this.icon,
    required this.color,
  });

  @override
  Widget build(BuildContext context) {
    return Row(
      mainAxisSize: MainAxisSize.min,
      children: [
        Icon(icon, size: 14, color: color),
        const SizedBox(width: 3),
        Text(
          '${value.toStringAsFixed(0)} $unit',
          style: TextStyle(fontSize: 12, color: color,
              fontWeight: FontWeight.w600),
        ),
      ],
    );
  }
}

// lib/features/recipes/widgets/category_chip.dart
import 'package:flutter/material.dart';

class CategoryChip extends StatelessWidget {
  final String category;

  const CategoryChip({super.key, required this.category});

  // static const — доступно в const конструкторе
  static const _data = {
    'завтрак': (emoji: '🥞', color: Color(0xFFFFF3E0)),
    'обед':    (emoji: '🥗', color: Color(0xFFE8F5E9)),
    'ужин':    (emoji: '🌙', color: Color(0xFFE3F2FD)),
    'перекус': (emoji: '🍎', color: Color(0xFFFCE4EC)),
  };

  @override
  Widget build(BuildContext context) {
    final info = _data[category] ??
        (emoji: '🍽', color: const Color(0xFFF5F5F5));
    return Container(
      padding: const EdgeInsets.symmetric(horizontal: 8, vertical: 3),
      decoration: BoxDecoration(
        color: info.color,
        borderRadius: BorderRadius.circular(12),
      ),
      child: Text('${info.emoji} $category',
          style: const TextStyle(fontSize: 12, fontWeight: FontWeight.w500)),
    );
  }
}

// lib/features/recipes/widgets/recipe_card.dart
import 'package:flutter/material.dart';
import '../../../core/domain/models/recipe.dart';
import 'category_chip.dart';
import 'nutrition_badge.dart';

class RecipeCard extends StatelessWidget {
  final Recipe recipe;
  final VoidCallback? onTap;
  final VoidCallback? onFavoriteTap;

  const RecipeCard({
    super.key,
    required this.recipe,
    this.onTap,
    this.onFavoriteTap,
  });

  @override
  Widget build(BuildContext context) {
    return Card(
      clipBehavior: Clip.antiAlias,
      child: InkWell(
        onTap: onTap,
        child: Padding(
          padding: const EdgeInsets.all(12),
          child: Row(
            children: [
              CircleAvatar(
                backgroundColor: Colors.green.shade100,
                radius: 28,
                child: const Icon(Icons.restaurant, color: Colors.green),
              ),
              const SizedBox(width: 12),
              Expanded(
                child: Column(
                  crossAxisAlignment: CrossAxisAlignment.start,
                  children: [
                    Text(recipe.title,
                        style: const TextStyle(
                            fontWeight: FontWeight.bold, fontSize: 15)),
                    const SizedBox(height: 4),
                    CategoryChip(category: recipe.category),
                    const SizedBox(height: 6),
                    NutritionBadge(
                      value: recipe.calories,
                      unit: 'ккал',
                      icon: Icons.local_fire_department,
                      color: Colors.deepOrange,
                    ),
                  ],
                ),
              ),
              if (onFavoriteTap != null)
                IconButton(
                  icon: Icon(
                    recipe.isFavorite ? Icons.favorite : Icons.favorite_border,
                    color: recipe.isFavorite ? Colors.red : Colors.grey,
                  ),
                  onPressed: onFavoriteTap,
                ),
            ],
          ),
        ),
      ),
    );
  }
}
```

</details>
