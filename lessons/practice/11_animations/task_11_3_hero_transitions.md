# 11.3: Hero Transitions — плавный переход на детали рецепта

> Project: FitMenu | Глава 11 — Animations

### 11.3: Hero анимация карточки рецепта при переходе на детали

🎯 **Цель шага:** Добавить плавный Hero-переход между карточкой рецепта в списке и большой фотографией на экране деталей — изображение "летит" из маленькой карточки в полноэкранный заголовок.

---

📝 **Техническое задание:**

Обнови `RecipeCard` и `RecipeDetailScreen` для поддержки Hero.

**В RecipeCard (источник):**
```dart
Hero(
  tag: 'recipe-image-${recipe.id}', // уникальный тег
  child: Image.network(
    recipe.imageUrl,
    fit: BoxFit.cover,
    width: double.infinity,
    height: double.infinity,
  ),
)
```

**В RecipeDetailScreen (цель):**
```dart
SliverAppBar(
  expandedHeight: 280,
  flexibleSpace: FlexibleSpaceBar(
    background: Hero(
      tag: 'recipe-image-${widget.recipe.id}',
      child: Image.network(recipe.imageUrl, fit: BoxFit.cover),
    ),
  ),
)
```

**Дополнительно — Hero для названия рецепта:**
```dart
// В карточке:
Hero(
  tag: 'recipe-title-${recipe.id}',
  child: Material(
    color: Colors.transparent,
    child: Text(recipe.title, style: ...),
  ),
)

// В деталях:
Hero(
  tag: 'recipe-title-${recipe.id}',
  child: Material(
    color: Colors.transparent,
    child: Text(recipe.title, style: Theme.of(context).textTheme.headlineSmall),
  ),
)
```

**Кастомная анимация перехода с Hero:**
```dart
Navigator.push(context, PageRouteBuilder(
  transitionDuration: const Duration(milliseconds: 500),
  pageBuilder: (_, __, ___) => RecipeDetailScreen(recipe: recipe),
  transitionsBuilder: (_, animation, __, child) =>
      FadeTransition(opacity: animation, child: child),
));
```

---

✅ **Критерии приёмки:**
- [ ] Hero тег уникален для каждого рецепта: `'recipe-image-${recipe.id}'`
- [ ] Изображение плавно "летит" из карточки в детали
- [ ] `Material(color: Colors.transparent)` оборачивает текст в Hero (иначе чёрный фон)
- [ ] Анимация работает и в обратную сторону (pop)
- [ ] Тег одинаковый на обоих экранах (опечатка = нет анимации)
- [ ] `transitionDuration` кастомизирован (не дефолтные 300мс)

---

💡 **Подсказка:** Hero тег должен быть СТРОГО одинаковым на обоих экранах. Если между двумя Hero-виджетами есть несколько маршрутов в стеке — Flutter может выдать ошибку "multiple heroes with same tag". Оборачивай текст в `Material(color: Colors.transparent)` — иначе Hero рисует чёрный прямоугольник поверх.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// Обновлённый RecipeCard с Hero
import 'package:flutter/material.dart';

class RecipeCard extends StatelessWidget {
  const RecipeCard({super.key, required this.recipe, required this.onTap});
  final Recipe recipe;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: onTap,
      child: ClipRRect(
        borderRadius: BorderRadius.circular(12),
        child: SizedBox(
          width: 160,
          height: 200,
          child: Stack(
            fit: StackFit.expand,
            children: [
              // Hero для изображения
              Hero(
                tag: 'recipe-image-${recipe.id}',
                child: Image.network(
                  recipe.imageUrl,
                  fit: BoxFit.cover,
                  // flightShuttleBuilder для кастомного виджета во время полёта:
                  // flightShuttleBuilder: (_, animation, __, ___, ____) => ...
                ),
              ),

              // Нижний оверлей
              Positioned(
                bottom: 0, left: 0, right: 0,
                child: Container(
                  padding: const EdgeInsets.all(8),
                  decoration: const BoxDecoration(
                    gradient: LinearGradient(
                      begin: Alignment.topCenter,
                      end: Alignment.bottomCenter,
                      colors: [Colors.transparent, Colors.black87],
                    ),
                  ),
                  child: Hero(
                    tag: 'recipe-title-${recipe.id}',
                    child: Material(
                      color: Colors.transparent,
                      child: Text(
                        recipe.title,
                        style: const TextStyle(color: Colors.white, fontSize: 13, fontWeight: FontWeight.bold),
                        maxLines: 2,
                        overflow: TextOverflow.ellipsis,
                      ),
                    ),
                  ),
                ),
              ),

              // Бейдж калорий
              Positioned(
                top: 8, right: 8,
                child: Container(
                  padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 3),
                  decoration: BoxDecoration(
                    color: Colors.black.withOpacity(0.6),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Text(
                    '🔥 ${recipe.calories} ккал',
                    style: const TextStyle(color: Colors.white, fontSize: 11),
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

```dart
// Переход с кастомной длительностью
void _openRecipe(BuildContext context, Recipe recipe) {
  Navigator.push(
    context,
    PageRouteBuilder(
      transitionDuration: const Duration(milliseconds: 500),
      reverseTransitionDuration: const Duration(milliseconds: 400),
      pageBuilder: (_, __, ___) => RecipeDetailScreen(recipe: recipe),
      transitionsBuilder: (_, animation, __, child) => FadeTransition(
        opacity: CurvedAnimation(parent: animation, curve: Curves.easeInOut),
        child: child,
      ),
    ),
  );
}
```

</details>
