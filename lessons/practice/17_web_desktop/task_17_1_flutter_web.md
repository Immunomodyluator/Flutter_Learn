# 17.1: Flutter Web — PWA сборка FitMenu

> Project: FitMenu | Глава 17 — Web & Desktop

### 17.1: Веб-версия FitMenu как Progressive Web App

🎯 **Цель шага:** Собрать FitMenu как PWA (Progressive Web App) с адаптивным layout для браузера — добавить web manifest, иконки и обработку URL через GoRouter.

---

📝 **Техническое задание:**

**1. Включить web поддержку:**
```bash
flutter config --enable-web
flutter create . --platforms web
flutter run -d chrome
flutter build web --release --web-renderer canvaskit
```

**2. `web/manifest.json`:**
```json
{
  "name": "FitMenu — Трекер питания",
  "short_name": "FitMenu",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#FFFFFF",
  "theme_color": "#6750A4",
  "orientation": "portrait-primary",
  "icons": [
    { "src": "icons/Icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "icons/Icon-512.png", "sizes": "512x512", "type": "image/png", "purpose": "maskable" }
  ]
}
```

**3. GoRouter для web (поддержка URL в браузере):**
```dart
final router = GoRouter(
  debugLogDiagnostics: kDebugMode,
  routes: [
    GoRoute(path: '/', builder: (_, __) => const HomeScreen()),
    GoRoute(
      path: '/recipes/:id',
      builder: (_, state) => RecipeDetailScreen(
        recipeId: state.pathParameters['id']!,
      ),
    ),
  ],
);
```

**4. Адаптивный layout для web:**
```dart
class AdaptiveLayout extends StatelessWidget {
  const AdaptiveLayout({super.key, required this.body});
  final Widget body;

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;
    if (width >= 1200) return _WideLayout(body: body);
    if (width >= 600)  return _MediumLayout(body: body);
    return _NarrowLayout(body: body);
  }
}
```

**5. `web/index.html` — ServiceWorker для offline:**
```html
<script>
  if ('serviceWorker' in navigator) {
    window.addEventListener('flutter-first-frame', function () {
      navigator.serviceWorker.register('flutter_service_worker.js');
    });
  }
</script>
```

---

✅ **Критерии приёмки:**
- [ ] `flutter build web` компилируется без ошибок
- [ ] Приложение устанавливается как PWA (кнопка "Установить" в Chrome)
- [ ] Прямые URL (`/recipes/123`) работают при перезагрузке страницы
- [ ] Layout адаптируется на ширинах 375, 768, 1280px
- [ ] ServiceWorker кэширует assets для офлайн работы

---

💡 **Подсказка:** `--web-renderer html` — меньше размер, хуже качество; `canvaskit` — полный Skia рендер. `flutter_web_plugins` позволяет регистрировать веб-специфичные плагины. `UrlStrategy` — `PathUrlStrategy()` убирает `#` из URL.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/main.dart

import 'package:flutter_web_plugins/url_strategy.dart';

void main() {
  usePathUrlStrategy();   // убираем # из URL: /recipes вместо /#/recipes
  runApp(const FitMenuApp());
}
```

```dart
// lib/app.dart

class FitMenuApp extends StatelessWidget {
  const FitMenuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title:        'FitMenu',
      debugShowCheckedModeBanner: false,
      theme:        AppTheme.lightTheme,
      darkTheme:    AppTheme.darkTheme,
      themeMode:    ThemeMode.system,
      routerConfig: AppRouter.router,
    );
  }
}
```

```dart
// lib/core/layout/adaptive_layout.dart

import 'package:flutter/material.dart';
import 'package:flutter_adaptive_scaffold/flutter_adaptive_scaffold.dart';

class FitMenuAdaptiveLayout extends StatelessWidget {
  const FitMenuAdaptiveLayout({
    super.key,
    required this.selectedIndex,
    required this.onDestinationSelected,
    required this.body,
  });

  final int selectedIndex;
  final ValueChanged<int> onDestinationSelected;
  final Widget body;

  static const _destinations = [
    NavigationDestination(icon: Icon(Icons.home_outlined),       selectedIcon: Icon(Icons.home),       label: 'Главная'),
    NavigationDestination(icon: Icon(Icons.restaurant_outlined), selectedIcon: Icon(Icons.restaurant), label: 'Рецепты'),
    NavigationDestination(icon: Icon(Icons.calendar_month_outlined), selectedIcon: Icon(Icons.calendar_month), label: 'План'),
    NavigationDestination(icon: Icon(Icons.person_outline),      selectedIcon: Icon(Icons.person),     label: 'Профиль'),
  ];

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;

    if (width >= 1200) {
      // Desktop: NavigationDrawer
      return Scaffold(
        body: Row(
          children: [
            NavigationDrawer(
              selectedIndex:           selectedIndex,
              onDestinationSelected:   onDestinationSelected,
              children: _destinations.map((d) => NavigationDrawerDestination(
                icon:  d.icon,
                label: Text(d.label),
              )).toList(),
            ),
            Expanded(child: body),
          ],
        ),
      );
    }

    if (width >= 600) {
      // Tablet: NavigationRail
      return Scaffold(
        body: Row(
          children: [
            NavigationRail(
              selectedIndex:           selectedIndex,
              onDestinationSelected:   onDestinationSelected,
              extended:                width >= 900,
              destinations: _destinations.map((d) => NavigationRailDestination(
                icon:  d.icon,
                label: Text(d.label),
              )).toList(),
            ),
            Expanded(child: body),
          ],
        ),
      );
    }

    // Mobile: BottomNavigationBar
    return Scaffold(
      body:                  body,
      bottomNavigationBar:   NavigationBar(
        selectedIndex:          selectedIndex,
        onDestinationSelected:  onDestinationSelected,
        destinations:           _destinations,
      ),
    );
  }
}
```

</details>
