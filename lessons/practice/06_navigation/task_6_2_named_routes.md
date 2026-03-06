# 6.2: Named Routes — именованная навигация

> Project: FitMenu | Глава 6 — Navigation

### 6.2: Настройка named routes и передача аргументов

🎯 **Цель шага:** Настроить именованные маршруты (`onGenerateRoute`) в FitMenu и передавать аргументы между экранами через `RouteSettings.arguments` — подготовка к переходу на GoRouter.

---

📝 **Техническое задание:**

Реализуй маршрутизацию в `lib/core/router/app_routes.dart` и `lib/core/router/route_generator.dart`.

**Константы маршрутов:**
```dart
class AppRoutes {
  static const home         = '/';
  static const recipes      = '/recipes';
  static const recipeDetail = '/recipes/detail';
  static const mealPlan     = '/meal-plan';
  static const settings     = '/settings';
}
```

**RouteGenerator (`onGenerateRoute`):**
```dart
class RouteGenerator {
  static Route<dynamic> generate(RouteSettings settings) {
    switch (settings.name) {
      case AppRoutes.home:
        return MaterialPageRoute(builder: (_) => const HomeScreen());
      case AppRoutes.recipeDetail:
        final recipe = settings.arguments as Recipe;
        return MaterialPageRoute(
          builder: (_) => RecipeDetailScreen(recipe: recipe),
        );
      // ... остальные маршруты
      default:
        return MaterialPageRoute(builder: (_) => const NotFoundScreen());
    }
  }
}
```

**NotFoundScreen:** простой экран с текстом "404 — Страница не найдена" и кнопкой "На главную".

**Навигация по имени:**
```dart
// Переход с аргументом:
Navigator.pushNamed(context, AppRoutes.recipeDetail, arguments: recipe);

// Возврат на главную, очистив стек:
Navigator.pushNamedAndRemoveUntil(context, AppRoutes.home, (_) => false);
```

**В MaterialApp:**
```dart
MaterialApp(
  onGenerateRoute: RouteGenerator.generate,
  initialRoute: AppRoutes.home,
)
```

---

✅ **Критерии приёмки:**
- [ ] Все маршруты определены как константы в `AppRoutes`
- [ ] `onGenerateRoute` обрабатывает все маршруты приложения
- [ ] Аргументы передаются через `settings.arguments` с приведением типа (`as Recipe`)
- [ ] Несуществующий маршрут показывает `NotFoundScreen` (не крашится)
- [ ] `initialRoute: AppRoutes.home` установлен в `MaterialApp`
- [ ] Нет хардкода строк маршрутов в виджетах (только константы из `AppRoutes`)

---

💡 **Подсказка:** `settings.arguments` имеет тип `Object?` — всегда делай безопасное приведение типа. Для передачи нескольких аргументов используй `Map<String, dynamic>` или отдельный класс аргументов. `Navigator.pushNamedAndRemoveUntil` полезен при logout — очищает весь стек навигации.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/router/app_routes.dart

class AppRoutes {
  AppRoutes._();

  static const home         = '/';
  static const recipes      = '/recipes';
  static const recipeDetail = '/recipes/detail';
  static const mealPlan     = '/meal-plan';
  static const settings     = '/settings';
}
```

```dart
// lib/core/router/route_generator.dart

import 'package:flutter/material.dart';
// import соответствующие экраны

class RouteGenerator {
  RouteGenerator._();

  static Route<dynamic> generate(RouteSettings settings) {
    switch (settings.name) {
      case AppRoutes.home:
        return _fade(const HomeScreen());

      case AppRoutes.recipes:
        return MaterialPageRoute(builder: (_) => const RecipesScreen());

      case AppRoutes.recipeDetail:
        final recipe = settings.arguments as Recipe?;
        if (recipe == null) return _error('Рецепт не передан');
        return MaterialPageRoute(
          builder: (_) => RecipeDetailScreen(recipe: recipe),
        );

      case AppRoutes.mealPlan:
        return MaterialPageRoute(builder: (_) => const MealPlanScreen());

      case AppRoutes.settings:
        return MaterialPageRoute(builder: (_) => const SettingsScreen());

      default:
        return MaterialPageRoute(builder: (_) => const NotFoundScreen());
    }
  }

  // Кастомный переход: fade
  static PageRoute _fade(Widget page) => PageRouteBuilder(
        pageBuilder: (_, __, ___) => page,
        transitionsBuilder: (_, animation, __, child) =>
            FadeTransition(opacity: animation, child: child),
      );

  // Страница с ошибкой аргументов
  static PageRoute _error(String message) => MaterialPageRoute(
        builder: (_) => Scaffold(
          body: Center(child: Text('Ошибка навигации: $message')),
        ),
      );
}
```

```dart
// lib/features/common/screens/not_found_screen.dart

import 'package:flutter/material.dart';

class NotFoundScreen extends StatelessWidget {
  const NotFoundScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            const Text('404', style: TextStyle(fontSize: 64, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            const Text('Страница не найдена'),
            const SizedBox(height: 24),
            ElevatedButton(
              onPressed: () => Navigator.pushNamedAndRemoveUntil(
                context,
                AppRoutes.home,
                (_) => false,
              ),
              child: const Text('На главную'),
            ),
          ],
        ),
      ),
    );
  }
}
```

</details>
