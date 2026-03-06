# 4.2: Stack и Positioned — карточка рецепта с оверлеем

> Project: FitMenu | Глава 4 — Layouts

### 4.2: RecipeCard с калорийным бейджем через Stack и Positioned

🎯 **Цель шага:** Создать виджет `RecipeCard`, в котором фотография рецепта занимает всю карточку, а поверх неё расположены бейдж с калориями (Positioned вверху справа) и градиентный оверлей с названием блюда (Positioned внизу) — используя `Stack` и `Positioned`.

---

📝 **Техническое задание:**

Реализуй виджет `RecipeCard` в файле `lib/features/recipes/widgets/recipe_card.dart`.

**Требования к UI:**
- Карточка размером 160×200 px (используй `SizedBox` или `AspectRatio`)
- Фоновое изображение: `Image.network` с `fit: BoxFit.cover`
- **Нижний оверлей** (через `Positioned(bottom: 0, left: 0, right: 0)`):
  - Градиент от прозрачного к чёрному (используй `BoxDecoration` с `LinearGradient`)
  - Название рецепта (белый жирный текст, максимум 2 строки)
  - Время приготовления (иконка + текст)
- **Бейдж калорий** (`Positioned(top: 8, right: 8)`):
  - Скруглённый контейнер с полупрозрачным чёрным фоном
  - Текст: "🔥 350 ккал"
- При нажатии карточка вызывает `onTap` callback
- Скруглённые углы у всей карточки (`ClipRRect` с `borderRadius: 12`)

**Модель данных:**
```dart
class Recipe {
  final String id;
  final String title;
  final String imageUrl;
  final int calories;
  final int cookingTimeMinutes;
}
```

---

✅ **Критерии приёмки:**
- [ ] Изображение заполняет всю карточку без белых полей
- [ ] Бейдж калорий не выходит за пределы карточки
- [ ] Название не перекрывает бейдж (расположены в разных углах)
- [ ] `ClipRRect` обрезает углы изображения
- [ ] Пока изображение загружается — виден `CircularProgressIndicator` (используй `loadingBuilder`)
- [ ] Виджет StatelessWidget с параметрами через конструктор

---

💡 **Подсказка:** `Stack` по умолчанию подстраивается под размер наибольшего дочернего элемента — задай размер через `SizedBox` или оберни в `AspectRatio`. `Positioned.fill` удобен для фонового изображения. Для градиента используй `LinearGradient(begin: Alignment.topCenter, end: Alignment.bottomCenter, colors: [Colors.transparent, Colors.black87])`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/widgets/recipe_card.dart

import 'package:flutter/material.dart';

class Recipe {
  const Recipe({
    required this.id,
    required this.title,
    required this.imageUrl,
    required this.calories,
    required this.cookingTimeMinutes,
  });

  final String id;
  final String title;
  final String imageUrl;
  final int calories;
  final int cookingTimeMinutes;
}

class RecipeCard extends StatelessWidget {
  const RecipeCard({
    super.key,
    required this.recipe,
    required this.onTap,
  });

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
              // Фоновое изображение
              Image.network(
                recipe.imageUrl,
                fit: BoxFit.cover,
                loadingBuilder: (context, child, loadingProgress) {
                  if (loadingProgress == null) return child;
                  return Container(
                    color: Colors.grey[200],
                    child: const Center(child: CircularProgressIndicator()),
                  );
                },
                errorBuilder: (_, __, ___) => Container(
                  color: Colors.grey[300],
                  child: const Icon(Icons.broken_image, size: 48),
                ),
              ),

              // Нижний градиентный оверлей с названием
              Positioned(
                bottom: 0,
                left: 0,
                right: 0,
                child: Container(
                  padding: const EdgeInsets.all(8),
                  decoration: const BoxDecoration(
                    gradient: LinearGradient(
                      begin: Alignment.topCenter,
                      end: Alignment.bottomCenter,
                      colors: [Colors.transparent, Colors.black87],
                    ),
                  ),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      Text(
                        recipe.title,
                        style: const TextStyle(
                          color: Colors.white,
                          fontWeight: FontWeight.bold,
                          fontSize: 13,
                        ),
                        maxLines: 2,
                        overflow: TextOverflow.ellipsis,
                      ),
                      const SizedBox(height: 4),
                      Row(
                        children: [
                          const Icon(Icons.timer, color: Colors.white70, size: 12),
                          const SizedBox(width: 2),
                          Text(
                            '${recipe.cookingTimeMinutes} мин',
                            style: const TextStyle(
                              color: Colors.white70,
                              fontSize: 11,
                            ),
                          ),
                        ],
                      ),
                    ],
                  ),
                ),
              ),

              // Бейдж калорий
              Positioned(
                top: 8,
                right: 8,
                child: Container(
                  padding: const EdgeInsets.symmetric(horizontal: 6, vertical: 3),
                  decoration: BoxDecoration(
                    color: Colors.black.withOpacity(0.6),
                    borderRadius: BorderRadius.circular(8),
                  ),
                  child: Text(
                    '🔥 ${recipe.calories} ккал',
                    style: const TextStyle(
                      color: Colors.white,
                      fontSize: 11,
                      fontWeight: FontWeight.w600,
                    ),
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

</details>
