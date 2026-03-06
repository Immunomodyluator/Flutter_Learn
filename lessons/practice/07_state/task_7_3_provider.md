# 7.3: Provider — список избранных рецептов

> Project: FitMenu | Глава 7 — State Management

### 7.3: FavoritesProvider с ChangeNotifier для избранных рецептов

🎯 **Цель шага:** Реализовать управление избранными рецептами через `ChangeNotifier` + `Provider` — добавление/удаление/проверка избранного с доступом из любого места приложения.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  provider: ^6.1.0
```

Реализуй `lib/features/recipes/providers/favorites_provider.dart`.

**FavoritesNotifier:**
```dart
class FavoritesNotifier extends ChangeNotifier {
  final Set<String> _favoriteIds = {};

  List<String> get favoriteIds => List.unmodifiable(_favoriteIds);
  int get count => _favoriteIds.length;

  bool isFavorite(String recipeId) => _favoriteIds.contains(recipeId);

  void toggle(String recipeId) {
    if (_favoriteIds.contains(recipeId)) {
      _favoriteIds.remove(recipeId);
    } else {
      _favoriteIds.add(recipeId);
    }
    notifyListeners();
  }

  void addAll(List<String> ids) { ... }
  void clear() { ... }
}
```

**Кнопка избранного в RecipeCard:**
- Иконка `Icons.favorite` / `Icons.favorite_border`
- Использует `Consumer<FavoritesNotifier>` или `context.watch`
- При нажатии: `context.read<FavoritesNotifier>().toggle(recipe.id)`

**Экран "Избранное":**
- `lib/features/favorites/screens/favorites_screen.dart`
- Список избранных рецептов через `Consumer<FavoritesNotifier>`
- Кнопка очистки всех избранных

**Регистрация провайдера в main:**
```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => FavoritesNotifier()),
    ChangeNotifierProvider(create: (_) => ThemeNotifier()),
  ],
  child: const FitMenuApp(),
)
```

---

✅ **Критерии приёмки:**
- [ ] `ChangeNotifierProvider` зарегистрирован выше `MaterialApp`
- [ ] `context.read<T>()` используется в обработчиках событий (не в `build`)
- [ ] `context.watch<T>()` или `Consumer<T>` используется для реактивного UI
- [ ] Иконка сердечка мгновенно меняется при нажатии
- [ ] `Set<String>` используется для хранения (не `List`) — O(1) поиск
- [ ] `notifyListeners()` вызывается после каждого изменения

---

💡 **Подсказка:** Главное правило: `context.read` в обработчиках (onTap, onPressed), `context.watch` в `build`. Никогда не используй `context.watch` внутри `initState` или колбэков — только в методе `build`. `Consumer<T>` позволяет сузить rebuild до минимального поддерева.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/providers/favorites_notifier.dart

import 'package:flutter/material.dart';

class FavoritesNotifier extends ChangeNotifier {
  final Set<String> _favoriteIds = {};

  List<String> get favoriteIds => List.unmodifiable(_favoriteIds.toList());
  int get count => _favoriteIds.length;

  bool isFavorite(String recipeId) => _favoriteIds.contains(recipeId);

  void toggle(String recipeId) {
    if (_favoriteIds.contains(recipeId)) {
      _favoriteIds.remove(recipeId);
    } else {
      _favoriteIds.add(recipeId);
    }
    notifyListeners();
  }

  void addAll(Iterable<String> ids) {
    _favoriteIds.addAll(ids);
    notifyListeners();
  }

  void clear() {
    _favoriteIds.clear();
    notifyListeners();
  }
}
```

```dart
// Кнопка избранного (виджет)
import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class FavoriteButton extends StatelessWidget {
  const FavoriteButton({super.key, required this.recipeId});

  final String recipeId;

  @override
  Widget build(BuildContext context) {
    // watch — подписка на изменения
    final isFav = context.watch<FavoritesNotifier>().isFavorite(recipeId);

    return IconButton(
      icon: Icon(
        isFav ? Icons.favorite : Icons.favorite_border,
        color: isFav ? Colors.red : null,
      ),
      onPressed: () =>
          // read — только чтение без подписки
          context.read<FavoritesNotifier>().toggle(recipeId),
    );
  }
}
```

```dart
// lib/features/favorites/screens/favorites_screen.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';

class FavoritesScreen extends StatelessWidget {
  const FavoritesScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Избранное'),
        actions: [
          IconButton(
            icon: const Icon(Icons.delete_sweep),
            onPressed: () => context.read<FavoritesNotifier>().clear(),
            tooltip: 'Очистить всё',
          ),
        ],
      ),
      // Consumer сужает rebuild до ListView
      body: Consumer<FavoritesNotifier>(
        builder: (context, favorites, _) {
          if (favorites.favoriteIds.isEmpty) {
            return const Center(child: Text('Нет избранных рецептов ❤️'));
          }
          return ListView.builder(
            itemCount: favorites.favoriteIds.length,
            itemBuilder: (context, index) {
              final id = favorites.favoriteIds[index];
              return ListTile(
                leading: const Icon(Icons.favorite, color: Colors.red),
                title: Text('Рецепт $id'),
                trailing: IconButton(
                  icon: const Icon(Icons.close),
                  onPressed: () => favorites.toggle(id),
                ),
              );
            },
          );
        },
      ),
    );
  }
}
```

</details>
