# 4.5: Responsive Layout — адаптивная сетка рецептов

> Project: FitMenu | Глава 4 — Layouts

### 4.5: Адаптивная RecipesGrid с MediaQuery и LayoutBuilder

🎯 **Цель шага:** Сделать сетку рецептов FitMenu адаптивной: на телефоне — 2 колонки, на планшете — 3, на десктопе — 4. Использовать `MediaQuery`, `LayoutBuilder` и `GridView.builder` с динамическим `crossAxisCount`.

---

📝 **Техническое задание:**

Реализуй виджет `RecipesGrid` в файле `lib/features/recipes/widgets/recipes_grid.dart`.

**Логика адаптивности:**
```
ширина < 600px  → 2 колонки  (телефон)
600px – 900px  → 3 колонки  (планшет / большой телефон горизонтально)
> 900px         → 4 колонки  (планшет широкий / десктоп)
```

**Требования:**
- Используй `LayoutBuilder` для получения ширины контейнера (`constraints.maxWidth`)
- **Не** используй `MediaQuery` для ширины экрана — `LayoutBuilder` предпочтительнее (ширина контейнера, а не экрана)
- Используй `GridView.builder` с `SliverGridDelegateWithFixedCrossAxisCount`
- `childAspectRatio: 0.75` (карточки выше, чем широкие)
- `crossAxisSpacing: 8`, `mainAxisSpacing: 8`
- Отступы по краям: `EdgeInsets.all(8)`
- Каждый элемент — `RecipeCard` (из задания 4.2)
- При пустом списке — центрированный текст "Рецепты не найдены 🍽️"

**Дополнительно:**
- Создай вспомогательный метод `int _crossAxisCount(double width)` внутри State/widget
- Добавь `MediaQuery.removePadding` вокруг `GridView`, если он вложен в Scaffold

---

✅ **Критерии приёмки:**
- [ ] На экране шириной 360px — ровно 2 колонки
- [ ] При повороте устройства (horizontal) колонки пересчитываются автоматически
- [ ] Используется `LayoutBuilder`, а не `MediaQuery.of(context).size.width` напрямую
- [ ] `GridView.builder` (не `GridView.count` со статическим числом)
- [ ] Пустой список показывает заглушку (не белый экран)
- [ ] `childAspectRatio` подобран так, что заголовок рецепта не обрезается

---

💡 **Подсказка:** `LayoutBuilder` даёт `BoxConstraints` — размеры родительского контейнера, не экрана. Это важно при вложении в боковую панель. `SliverGridDelegateWithMaxCrossAxisExtent` — альтернатива, которая сама вычисляет число колонок по максимальной ширине ячейки (например, `maxCrossAxisExtent: 200`).

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/widgets/recipes_grid.dart

import 'package:flutter/material.dart';
// import 'package:fitmenu/features/recipes/widgets/recipe_card.dart';

class RecipesGrid extends StatelessWidget {
  const RecipesGrid({
    super.key,
    required this.recipes,
    required this.onRecipeTap,
  });

  final List<Recipe> recipes;
  final void Function(Recipe recipe) onRecipeTap;

  int _crossAxisCount(double width) {
    if (width >= 900) return 4;
    if (width >= 600) return 3;
    return 2;
  }

  @override
  Widget build(BuildContext context) {
    if (recipes.isEmpty) {
      return const Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text('🍽️', style: TextStyle(fontSize: 48)),
            SizedBox(height: 8),
            Text('Рецепты не найдены'),
          ],
        ),
      );
    }

    return LayoutBuilder(
      builder: (context, constraints) {
        final columns = _crossAxisCount(constraints.maxWidth);

        return GridView.builder(
          padding: const EdgeInsets.all(8),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount: columns,
            childAspectRatio: 0.75,
            crossAxisSpacing: 8,
            mainAxisSpacing: 8,
          ),
          itemCount: recipes.length,
          itemBuilder: (context, index) => RecipeCard(
            recipe: recipes[index],
            onTap: () => onRecipeTap(recipes[index]),
          ),
        );
      },
    );
  }
}

// Альтернатива через SliverGridDelegateWithMaxCrossAxisExtent:
// gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
//   maxCrossAxisExtent: 200,
//   childAspectRatio: 0.75,
//   crossAxisSpacing: 8,
//   mainAxisSpacing: 8,
// ),
```

</details>
