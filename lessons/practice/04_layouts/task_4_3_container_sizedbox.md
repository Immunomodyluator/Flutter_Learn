# 4.3: Container и SizedBox — стилизованный тайл рецепта

> Project: FitMenu | Глава 4 — Layouts

### 4.3: RecipeTile с BoxDecoration, отступами и SizedBox-спейсерами

🎯 **Цель шага:** Создать горизонтальный виджет `RecipeTile` для списка рецептов — с миниатюрой, заголовком, тегами и кнопкой добавления в план питания. Отработать разницу между `Container`, `SizedBox`, `Padding`, `BoxDecoration`.

---

📝 **Техническое задание:**

Реализуй виджет `RecipeTile` в файле `lib/features/recipes/widgets/recipe_tile.dart`.

**Макет (горизонтальный):**
```
[ Фото 80×80 ]  [ Название (жирный)         ] [ + ]
                [ Теги: #суп #курица         ]
                [ ⏱ 30 мин  🔥 450 ккал     ]
```

**Требования:**
- Высота тайла фиксирована: `80 px` минимум (используй `ConstrainedBox` или `IntrinsicHeight`)
- Фото — скруглённый квадрат 80×80 через `Container` с `BoxDecoration(borderRadius, image: DecorationImage)`
- Между фото и текстом — `SizedBox(width: 12)`
- Название — максимум 2 строки, `overflow: TextOverflow.ellipsis`
- Теги — горизонтальный `Wrap` с чипами `Chip(label: Text(tag))`, высота `labelPadding` минимальная
- Кнопка "+" — `IconButton(icon: Icon(Icons.add_circle_outline))`
- Весь тайл — `InkWell` с `onTap`
- Разделитель снизу — `Divider`

**Данные:**
```dart
class RecipeTile extends StatelessWidget {
  // recipe: Recipe (из задания 4.2)
  // onTap: VoidCallback
  // onAddToMealPlan: VoidCallback
}
```

---

✅ **Критерии приёмки:**
- [ ] Фото отображается с `BoxFit.cover` и `borderRadius: 8`
- [ ] Теги не переполняют строку — используется `Wrap`, а не `Row`
- [ ] `SizedBox` используется как спейсер (не `Padding` для горизонтальных отступов между компонентами)
- [ ] `Container` с `BoxDecoration` для стилизации фото (не `ClipRRect + Image`)
- [ ] Кнопка добавления вызывает `onAddToMealPlan`, а не `onTap`
- [ ] На экране списка 10 тайлов нет лишнего рендеринга (виджет `const`-совместим)

---

💡 **Подсказка:** Используй `BoxDecoration` с `image: DecorationImage(image: NetworkImage(url), fit: BoxFit.cover)` прямо внутри `Container` — это эффективнее, чем `ClipRRect + Image.network` для небольших превью. `SizedBox` без дочернего виджета — это "пустой спейсер". `Wrap` автоматически переносит чипы на новую строку, если не помещаются.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/widgets/recipe_tile.dart

import 'package:flutter/material.dart';
// Используем Recipe из task_4_2
// import 'package:fitmenu/features/recipes/widgets/recipe_card.dart';

class RecipeTile extends StatelessWidget {
  const RecipeTile({
    super.key,
    required this.recipe,
    required this.onTap,
    required this.onAddToMealPlan,
    this.tags = const [],
  });

  final Recipe recipe;
  final VoidCallback onTap;
  final VoidCallback onAddToMealPlan;
  final List<String> tags;

  @override
  Widget build(BuildContext context) {
    final textTheme = Theme.of(context).textTheme;

    return InkWell(
      onTap: onTap,
      child: Padding(
        padding: const EdgeInsets.symmetric(horizontal: 16, vertical: 8),
        child: Row(
          crossAxisAlignment: CrossAxisAlignment.center,
          children: [
            // Миниатюра через Container + BoxDecoration
            Container(
              width: 80,
              height: 80,
              decoration: BoxDecoration(
                borderRadius: BorderRadius.circular(8),
                image: DecorationImage(
                  image: NetworkImage(recipe.imageUrl),
                  fit: BoxFit.cover,
                ),
                color: Colors.grey[200], // заглушка до загрузки
              ),
            ),

            const SizedBox(width: 12),

            // Текстовый блок
            Expanded(
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                mainAxisSize: MainAxisSize.min,
                children: [
                  // Название
                  Text(
                    recipe.title,
                    style: textTheme.titleSmall
                        ?.copyWith(fontWeight: FontWeight.bold),
                    maxLines: 2,
                    overflow: TextOverflow.ellipsis,
                  ),

                  const SizedBox(height: 4),

                  // Теги
                  if (tags.isNotEmpty)
                    Wrap(
                      spacing: 4,
                      runSpacing: 0,
                      children: tags
                          .map(
                            (tag) => Chip(
                              label: Text(tag),
                              labelStyle: textTheme.labelSmall,
                              padding: EdgeInsets.zero,
                              labelPadding:
                                  const EdgeInsets.symmetric(horizontal: 4),
                              visualDensity: VisualDensity.compact,
                            ),
                          )
                          .toList(),
                    ),

                  const SizedBox(height: 4),

                  // Время и калории
                  Row(
                    children: [
                      const Icon(Icons.timer, size: 14, color: Colors.grey),
                      const SizedBox(width: 2),
                      Text(
                        '${recipe.cookingTimeMinutes} мин',
                        style: textTheme.labelSmall,
                      ),
                      const SizedBox(width: 8),
                      const Icon(Icons.local_fire_department,
                          size: 14, color: Colors.orange),
                      const SizedBox(width: 2),
                      Text(
                        '${recipe.calories} ккал',
                        style: textTheme.labelSmall,
                      ),
                    ],
                  ),
                ],
              ),
            ),

            // Кнопка добавления в план
            IconButton(
              icon: const Icon(Icons.add_circle_outline),
              onPressed: onAddToMealPlan,
              tooltip: 'Добавить в план питания',
            ),
          ],
        ),
      ),
    );
  }
}
```

</details>
