# 6.3: GoRouter — декларативная навигация

> Project: FitMenu | Глава 6 — Navigation

### 6.3: Настройка GoRouter с нижней навигацией (BottomNavigationBar)

🎯 **Цель шага:** Заменить именованные маршруты на GoRouter — реализовать shell-маршрут с постоянной `BottomNavigationBar` и 4 вкладками (Главная, Рецепты, План питания, Профиль), где каждая вкладка имеет свой стек навигации.

---

📝 **Техническое задание:**

Реализуй роутер в `lib/core/router/app_router.dart`.

**Зависимость:**
```yaml
dependencies:
  go_router: ^13.0.0
```

**Структура маршрутов:**
```
/home                    → HomeScreen
/recipes                 → RecipesScreen
  /recipes/:id           → RecipeDetailScreen(id)
/meal-plan               → MealPlanScreen
/settings                → SettingsScreen
```

**ShellRoute** — общая оболочка с `BottomNavigationBar`:
```dart
ShellRoute(
  builder: (context, state, child) => AppShell(child: child),
  routes: [
    GoRoute(path: '/home', builder: ...),
    GoRoute(
      path: '/recipes',
      builder: ...,
      routes: [
        GoRoute(
          path: ':id',  // /recipes/:id
          builder: (context, state) {
            final id = state.pathParameters['id']!;
            // получи рецепт по id
            return RecipeDetailScreen(recipeId: id);
          },
        ),
      ],
    ),
    GoRoute(path: '/meal-plan', builder: ...),
    GoRoute(path: '/settings', builder: ...),
  ],
)
```

**AppShell** — Scaffold с BottomNavigationBar:
- 4 таба с иконками и подписями
- Определяет активный таб по `GoRouterState.location`
- При нажатии таба: `context.go('/recipes')`

**Навигация в коде:**
```dart
// Переход на детали:
context.push('/recipes/${recipe.id}');
// или с extra-объектом:
context.push('/recipes/${recipe.id}', extra: recipe);
```

---

✅ **Критерии приёмки:**
- [ ] `GoRouter` настроен с `ShellRoute` и 4 вкладками
- [ ] `BottomNavigationBar` остаётся видимым при переходе между вкладками
- [ ] Переход на `/recipes/:id` работает через `context.push`
- [ ] Активный таб подсвечивается правильно при навигации
- [ ] `initialLocation: '/home'` установлен
- [ ] При нажатии системной кнопки назад из деталей — возврат на список рецептов
- [ ] `errorBuilder` настроен (показывает NotFoundScreen)

---

💡 **Подсказка:** `ShellRoute` сохраняет состояние дочерних маршрутов при переключении вкладок. `state.matchedLocation` (или `state.uri.toString()`) удобнее `state.location` для определения активного таба. `context.go()` заменяет текущий маршрут, `context.push()` добавляет в стек.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/router/app_router.dart

import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

final appRouter = GoRouter(
  initialLocation: '/home',
  errorBuilder: (context, state) => const NotFoundScreen(),
  routes: [
    ShellRoute(
      builder: (context, state, child) => AppShell(child: child),
      routes: [
        GoRoute(
          path: '/home',
          builder: (context, state) => const HomeScreen(),
        ),
        GoRoute(
          path: '/recipes',
          builder: (context, state) => const RecipesScreen(),
          routes: [
            GoRoute(
              path: ':id',
              builder: (context, state) {
                final id = state.pathParameters['id']!;
                final recipe = state.extra as Recipe?;
                return RecipeDetailScreen(recipeId: id, recipe: recipe);
              },
            ),
          ],
        ),
        GoRoute(
          path: '/meal-plan',
          builder: (context, state) => const MealPlanScreen(),
        ),
        GoRoute(
          path: '/settings',
          builder: (context, state) => const SettingsScreen(),
        ),
      ],
    ),
  ],
);
```

```dart
// lib/core/widgets/app_shell.dart

import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class AppShell extends StatelessWidget {
  const AppShell({super.key, required this.child});

  final Widget child;

  static const _tabs = [
    (icon: Icons.home_outlined,      activeIcon: Icons.home,        label: 'Главная',   path: '/home'),
    (icon: Icons.restaurant_outlined, activeIcon: Icons.restaurant,  label: 'Рецепты',   path: '/recipes'),
    (icon: Icons.calendar_today_outlined, activeIcon: Icons.calendar_today, label: 'План',  path: '/meal-plan'),
    (icon: Icons.settings_outlined,  activeIcon: Icons.settings,    label: 'Настройки', path: '/settings'),
  ];

  int _currentIndex(BuildContext context) {
    final location = GoRouterState.of(context).matchedLocation;
    for (var i = 0; i < _tabs.length; i++) {
      if (location.startsWith(_tabs[i].path)) return i;
    }
    return 0;
  }

  @override
  Widget build(BuildContext context) {
    final index = _currentIndex(context);

    return Scaffold(
      body: child,
      bottomNavigationBar: NavigationBar(
        selectedIndex: index,
        onDestinationSelected: (i) => context.go(_tabs[i].path),
        destinations: _tabs
            .map((tab) => NavigationDestination(
                  icon: Icon(tab.icon),
                  selectedIcon: Icon(tab.activeIcon),
                  label: tab.label,
                ))
            .toList(),
      ),
    );
  }
}
```

```dart
// Использование в main.dart:
MaterialApp.router(
  routerConfig: appRouter,
  theme: AppTheme.lightTheme,
  darkTheme: AppTheme.darkTheme,
)
```

</details>
