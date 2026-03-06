# 9.4: Hive/Isar — кеш рецептов и офлайн-доступ

> Project: FitMenu | Глава 9 — Storage

### 9.4: RecipeCacheBox с Hive для офлайн-кеширования

🎯 **Цель шага:** Реализовать кеш рецептов через `Hive` — сохранять загруженные рецепты локально для офлайн-доступа и быстрого открытия приложения.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  hive_flutter: ^1.1.0

dev_dependencies:
  hive_generator: ^2.0.0
  build_runner: ^2.4.0
```

**RecipeCacheModel (Hive адаптер):**
```dart
@HiveType(typeId: 0)
class RecipeCacheModel extends HiveObject {
  @HiveField(0) late String id;
  @HiveField(1) late String title;
  @HiveField(2) late String imageUrl;
  @HiveField(3) late int calories;
  @HiveField(4) late int cookingTimeMinutes;
  @HiveField(5) late DateTime cachedAt;
}
```

**RecipeCacheService:**
```dart
class RecipeCacheService {
  static const _boxName = 'recipes_cache';
  static const _maxAge  = Duration(hours: 24);

  late Box<RecipeCacheModel> _box;

  Future<void> init() async {
    await Hive.initFlutter();
    Hive.registerAdapter(RecipeCacheModelAdapter());
    _box = await Hive.openBox<RecipeCacheModel>(_boxName);
  }

  Future<void> cacheRecipes(List<Recipe> recipes) async { ... }
  List<Recipe> getCachedRecipes() { ... }
  Future<void> clearExpired() async { ... }
  bool get isCacheValid { ... } // проверяет возраст кеша
}
```

**Логика кеширования:**
- `cacheRecipes`: сохраняет список, ключ = `recipe.id`
- `getCachedRecipes`: возвращает все записи из box как `List<Recipe>`
- `clearExpired`: удаляет записи старше `_maxAge`
- `isCacheValid`: `true` если последняя запись моложе 24 часов

**Паттерн Cache-First в репозитории:**
```dart
Future<List<Recipe>> getPopularRecipes() async {
  if (_cache.isCacheValid) {
    return _cache.getCachedRecipes(); // быстро, из памяти
  }
  final fresh = await _api.getPopularRecipes();
  await _cache.cacheRecipes(fresh);
  return fresh;
}
```

---

✅ **Критерии приёмки:**
- [ ] `@HiveType` и `@HiveField` аннотации на модели
- [ ] `Hive.registerAdapter()` вызывается до открытия box
- [ ] `build_runner` генерирует `RecipeCacheModelAdapter`
- [ ] `clearExpired()` удаляет устаревшие записи
- [ ] Cache-First паттерн реализован в репозитории
- [ ] `Hive.initFlutter()` вызывается в `main()` до `runApp()`

---

💡 **Подсказка:** `HiveObject` позволяет использовать `entry.delete()` напрямую. `box.values` возвращает `Iterable` всех значений. `box.putAll(map)` — пакетная запись. Для сложных запросов/индексов используй `Isar` вместо Hive — он быстрее и поддерживает полноценный query API.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/storage/recipe_cache_model.dart

import 'package:hive_flutter/hive_flutter.dart';

part 'recipe_cache_model.g.dart';

@HiveType(typeId: 0)
class RecipeCacheModel extends HiveObject {
  @HiveField(0) late String id;
  @HiveField(1) late String title;
  @HiveField(2) late String imageUrl;
  @HiveField(3) late int calories;
  @HiveField(4) late int cookingTimeMinutes;
  @HiveField(5) late DateTime cachedAt;
}
```

```dart
// lib/core/storage/recipe_cache_service.dart

import 'package:hive_flutter/hive_flutter.dart';

class RecipeCacheService {
  static const _boxName = 'recipes_cache';
  static const _maxAge  = Duration(hours: 24);

  late Box<RecipeCacheModel> _box;

  Future<void> init() async {
    await Hive.initFlutter();
    if (!Hive.isAdapterRegistered(0)) {
      Hive.registerAdapter(RecipeCacheModelAdapter());
    }
    _box = await Hive.openBox<RecipeCacheModel>(_boxName);
  }

  Future<void> cacheRecipes(List<Recipe> recipes) async {
    final map = {
      for (final r in recipes)
        r.id: RecipeCacheModel()
          ..id                  = r.id
          ..title               = r.title
          ..imageUrl            = r.imageUrl
          ..calories            = r.calories
          ..cookingTimeMinutes  = r.cookingTimeMinutes
          ..cachedAt            = DateTime.now(),
    };
    await _box.putAll(map);
  }

  List<Recipe> getCachedRecipes() {
    return _box.values
        .map((m) => Recipe(
              id:                  m.id,
              title:               m.title,
              imageUrl:            m.imageUrl,
              calories:            m.calories,
              cookingTimeMinutes:  m.cookingTimeMinutes,
            ))
        .toList();
  }

  Future<void> clearExpired() async {
    final cutoff = DateTime.now().subtract(_maxAge);
    final expired = _box.values
        .where((m) => m.cachedAt.isBefore(cutoff))
        .map((m) => m.key)
        .toList();
    await _box.deleteAll(expired);
  }

  bool get isCacheValid {
    if (_box.isEmpty) return false;
    final latest = _box.values
        .map((m) => m.cachedAt)
        .reduce((a, b) => a.isAfter(b) ? a : b);
    return DateTime.now().difference(latest) < _maxAge;
  }

  Future<void> clear() => _box.clear();
}
```

</details>
