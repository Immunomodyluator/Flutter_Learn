# 13.4: Feature-first структура проекта

> Project: FitMenu | Глава 13 — Architecture

### 13.4: Рефакторинг FitMenu на feature-first структуру

🎯 **Цель шага:** Реорганизовать всю структуру проекта FitMenu с layer-first (lib/data, lib/domain, lib/presentation) на feature-first (lib/features/recipes, lib/features/meal_plan) — для масштабируемости и изоляции фич.

---

📝 **Техническое задание:**

**Целевая структура `lib/`:**
```
lib/
├── core/                          ← общий код всего приложения
│   ├── auth/
│   ├── database/
│   ├── di/
│   ├── network/
│   ├── notifications/
│   ├── router/
│   ├── storage/
│   ├── theme/
│   └── widgets/                   ← общие виджеты (OfflineBanner, AppShell...)
├── features/
│   ├── home/
│   │   ├── screens/home_screen.dart
│   │   └── widgets/
│   ├── recipes/
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   ├── repositories/
│   │   │   └── usecases/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   ├── models/
│   │   │   ├── mappers/
│   │   │   └── repositories/
│   │   └── presentation/
│   │       ├── screens/
│   │       ├── viewmodels/
│   │       └── widgets/
│   ├── meal_plan/
│   ├── favorites/
│   ├── settings/
│   └── shopping/
└── main.dart
```

**Задание:**
1. Создай файл `lib/core/router/app_router.dart` с GoRouter (если не создан)
2. Создай `lib/features/home/screens/home_screen.dart` — главный экран с `NutritionStatsBar` + `AnimatedCalorieBar`
3. Опиши `barrel exports` для каждой фичи: `lib/features/recipes/recipes.dart` который экспортирует все публичные компоненты фичи
4. Создай `lib/features/README.md` с описанием правил добавления новых фич

**Barrel export пример:**
```dart
// lib/features/recipes/recipes.dart
export 'domain/entities/recipe.dart';
export 'domain/usecases/search_recipes_use_case.dart';
export 'presentation/screens/recipes_screen.dart';
export 'presentation/widgets/recipe_card.dart';
// НЕ экспортируем: data/models, data/repositories — это детали реализации
```

---

✅ **Критерии приёмки:**
- [ ] Структура `lib/features/{feature}/domain/data/presentation` для каждой фичи
- [ ] `lib/core/` содержит только cross-feature код
- [ ] Barrel export файл для каждой фичи
- [ ] `core/widgets/` содержит `AppShell`, `OfflineBanner`, `LoadingWidget`
- [ ] `features/README.md` описывает архитектурные правила
- [ ] Нет прямых импортов из `data/` в другие фичи (только через `domain/`)

---

💡 **Подсказка:** Feature-first даёт команде независимость — каждая фича как мини-приложение. Barrel export (`feature.dart`) скрывает внутреннюю структуру. Правило: другая фича может импортировать только `domain/entities` из чужой фичи (не data и не presentation).

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/recipes.dart — barrel export
library recipes;

// Domain (публичный API фичи)
export 'domain/entities/recipe.dart';
export 'domain/repositories/recipe_repository.dart';
export 'domain/usecases/search_recipes_use_case.dart';
export 'domain/usecases/get_popular_recipes_use_case.dart';

// Presentation (публичные виджеты)
export 'presentation/screens/recipes_screen.dart';
export 'presentation/screens/recipe_detail_screen.dart';
export 'presentation/widgets/recipe_card.dart';
export 'presentation/widgets/recipe_tile.dart';
export 'presentation/widgets/recipes_grid.dart';

// НЕ экспортируем data-слой — это детали реализации:
// export 'data/models/recipe_dto.dart';         // ❌
// export 'data/repositories/recipe_repo_impl.dart'; // ❌
```

```markdown
<!-- lib/features/README.md -->
# Архитектура фич FitMenu

## Структура фичи
Каждая фича следует Clean Architecture:
```
feature/
  domain/    ← бизнес-логика, не зависит ни от чего
  data/      ← реализация репозиториев и datasource
  presentation/ ← виджеты, экраны, viewmodels
```

## Правила
1. Фича A может импортировать из фичи B только `domain/entities`
2. `data/` — приватный, не импортировать из других фич
3. Каждая фича имеет `feature_name.dart` barrel export
4. Новый экран → новый UseCase → новый тест
```

```dart
// lib/features/home/screens/home_screen.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('FitMenu')),
      body: ListView(
        padding: const EdgeInsets.all(16),
        children: [
          // КБЖУ за сегодня
          const NutritionStatsBar(
            stats: NutritionStats(
              calories: 1240,
              targetCalories: 2000,
              protein: 85,
              fat: 42,
              carbs: 165,
            ),
          ),
          const SizedBox(height: 16),
          // Анимированная полоса калорий
          const AnimatedCalorieBar(current: 1240, target: 2000),
        ],
      ),
    );
  }
}
```

</details>
