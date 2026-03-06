# 17.3: Adaptive UI — AdaptiveScaffold и NavigationRail

> Project: FitMenu | Глава 17 — Web & Desktop

### 17.3: Адаптивный интерфейс с NavigationRail и AdaptiveScaffold

🎯 **Цель шага:** Сделать FitMenu по-настоящему адаптивным: на мобильном — `NavigationBar`, на планшете — `NavigationRail`, на десктопе — `NavigationDrawer` с расширенными подписями.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  flutter_adaptive_scaffold: ^0.1.7
```

**Брейкпоинты Material 3:**
- `xs` < 600px  → BottomNavBar
- `sm` 600–840px → NavigationRail (collapsed)
- `md` 840–1200px → NavigationRail (expanded)
- `lg` ≥ 1200px → NavigationDrawer

**Реализация с `flutter_adaptive_scaffold`:**
```dart
class RootScreen extends StatefulWidget {
  const RootScreen({super.key, required this.child, required this.location});
  final Widget child;
  final String location;
  @override State<RootScreen> createState() => _RootScreenState();
}

class _RootScreenState extends State<RootScreen> {
  int get _selectedIndex {
    const tabs = ['/', '/recipes', '/plan', '/profile'];
    final index = tabs.indexWhere((t) => widget.location.startsWith(t));
    return index < 0 ? 0 : index;
  }

  @override
  Widget build(BuildContext context) {
    return AdaptiveScaffold(
      destinations: const [
        NavigationDestination(icon: Icon(Icons.home_outlined),       selectedIcon: Icon(Icons.home),       label: 'Главная'),
        NavigationDestination(icon: Icon(Icons.restaurant_outlined), selectedIcon: Icon(Icons.restaurant), label: 'Рецепты'),
        NavigationDestination(icon: Icon(Icons.calendar_month_outlined), selectedIcon: Icon(Icons.calendar_month), label: 'План'),
        NavigationDestination(icon: Icon(Icons.person_outline),      selectedIcon: Icon(Icons.person),     label: 'Профиль'),
      ],
      selectedIndex:          _selectedIndex,
      onSelectedIndexChange:  (i) => _navigate(context, i),
      body:                   (_) => widget.child,
    );
  }

  void _navigate(BuildContext context, int index) {
    const routes = ['/', '/recipes', '/plan', '/profile'];
    context.go(routes[index]);
  }
}
```

**Адаптивная сетка рецептов:**
```dart
LayoutBuilder(
  builder: (context, constraints) {
    final columns = switch (constraints.maxWidth) {
      < 600    => 1,
      < 900    => 2,
      < 1200   => 3,
      _        => 4,
    };
    return GridView.builder(
      gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: columns,
        childAspectRatio: 4 / 3,
        crossAxisSpacing: 16,
        mainAxisSpacing: 16,
      ),
      ...
    );
  },
),
```

---

✅ **Критерии приёмки:**
- [ ] На ширине < 600px — `NavigationBar` (снизу)
- [ ] На ширине 600–840px — `NavigationRail` (слева, иконки)
- [ ] На ширине ≥ 840px — `NavigationRail` с подписями
- [ ] `GridView.builder` меняет количество колонн с 1 до 4
- [ ] Горизонтальная ориентация планшета обрабатывается корректно

---

💡 **Подсказка:** `Breakpoints.mediumAndUp` из `flutter_adaptive_scaffold` — удобные брейкпоинты Material 3. `MediaQuery.sizeOf(context)` вместо `MediaQuery.of(context).size` — оптимизированная версия (перестраивает только при изменении size). `OrientationBuilder` для реакции на поворот.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/shell/presentation/adaptive_shell_screen.dart

import 'package:flutter/material.dart';
import 'package:go_router/go_router.dart';

class AdaptiveShellScreen extends StatelessWidget {
  const AdaptiveShellScreen({
    super.key,
    required this.navigationShell,
  });

  final StatefulNavigationShell navigationShell;

  static const _destinations = [
    NavigationDestination(
      icon:         Icon(Icons.home_outlined),
      selectedIcon: Icon(Icons.home),
      label:        'Главная',
    ),
    NavigationDestination(
      icon:         Icon(Icons.restaurant_menu_outlined),
      selectedIcon: Icon(Icons.restaurant_menu),
      label:        'Рецепты',
    ),
    NavigationDestination(
      icon:         Icon(Icons.event_note_outlined),
      selectedIcon: Icon(Icons.event_note),
      label:        'План',
    ),
    NavigationDestination(
      icon:         Icon(Icons.person_outline),
      selectedIcon: Icon(Icons.person),
      label:        'Профиль',
    ),
  ];

  void _onDestinationSelected(int index) {
    navigationShell.goBranch(
      index,
      initialLocation: index == navigationShell.currentIndex,
    );
  }

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;

    if (width >= 1200) return _DrawerLayout(
      destinations:      _destinations,
      selectedIndex:     navigationShell.currentIndex,
      onDestinationSelected: _onDestinationSelected,
      body:              navigationShell,
    );

    if (width >= 600) return _RailLayout(
      destinations:      _destinations,
      selectedIndex:     navigationShell.currentIndex,
      onDestinationSelected: _onDestinationSelected,
      extended:          width >= 840,
      body:              navigationShell,
    );

    return Scaffold(
      body: navigationShell,
      bottomNavigationBar: NavigationBar(
        selectedIndex:         navigationShell.currentIndex,
        onDestinationSelected: _onDestinationSelected,
        destinations:          _destinations,
        labelBehavior:         NavigationDestinationLabelBehavior.onlyShowSelected,
      ),
    );
  }
}

// lib/features/recipes/presentation/responsive_recipes_grid.dart

class ResponsiveRecipesGrid extends StatelessWidget {
  const ResponsiveRecipesGrid({super.key, required this.recipes});
  final List<Recipe> recipes;

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        final crossAxisCount = switch (constraints.maxWidth) {
          < 600  => 1,
          < 900  => 2,
          < 1200 => 3,
          _      => 4,
        };
        return GridView.builder(
          padding:  const EdgeInsets.all(16),
          gridDelegate: SliverGridDelegateWithFixedCrossAxisCount(
            crossAxisCount:  crossAxisCount,
            childAspectRatio: 3 / 2,
            crossAxisSpacing: 16,
            mainAxisSpacing:  16,
          ),
          itemCount: recipes.length,
          itemBuilder: (_, i) => RecipeCard(
            key:    ValueKey(recipes[i].id),
            recipe: recipes[i],
          ),
        );
      },
    );
  }
}
```

</details>
