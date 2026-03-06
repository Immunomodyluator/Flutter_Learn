# 1.6: Асинхронность: Future и async/await

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 1 — Введение в Dart

---

### 1.6: Асинхронность: Future и async/await

🎯 **Цель шага:** Реализовать асинхронный `RecipeRepository` с обработкой ошибок, таймаутом, retry-логикой и параллельной загрузкой через `Future.wait`.

📝 **Техническое задание:**

Создай `lib/data/recipe_repository.dart`:

1. `Future<List<Recipe>> getRecipes()` — имитирует сетевой запрос через `Future.delayed`, применяет `.timeout(Duration(seconds: 5))`
2. `Future<Recipe> getRecipeById(String id)` — выбрасывает `NotFoundException` если рецепт не найден
3. `Future<T> withRetry<T>(Future<T> Function() fn, {int maxAttempts = 3})` — retry с экспоненциальной задержкой (1с, 2с, 4с)
4. В `main()` загрузи 3 рецепта **параллельно** через `Future.wait` и замерь время через `Stopwatch`
5. Поймай `TimeoutException` и `NotFoundException` в разных `catch`-блоках

✅ **Критерии приёмки:**
- [ ] `withRetry` делает максимум `maxAttempts` попыток и `rethrow` после последней
- [ ] `TimeoutException` перехватывается и возвращается fallback-кеш (не крэш)
- [ ] `NotFoundException` — типизированное исключение, не просто `Exception('...')`
- [ ] `Future.wait([f1, f2, f3])` запускает запросы **параллельно** (не последовательно)
- [ ] `Stopwatch` показывает что параллельная загрузка быстрее последовательной

💡 **Подсказка:** `Future.wait` принимает `List<Future<T>>` и возвращает `Future<List<T>>`. Сравни: 3 запроса по 200мс последовательно = ~600мс, параллельно = ~200мс.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
import 'dart:async';
import 'dart:math';

class NotFoundException implements Exception {
  final String message;
  const NotFoundException(this.message);
  @override
  String toString() => 'NotFoundException: $message';
}

class NetworkException implements Exception {
  final String message;
  const NetworkException(this.message);
  @override
  String toString() => 'NetworkException: $message';
}

class Recipe {
  final String id;
  final String title;
  final double calories;
  const Recipe({required this.id, required this.title, required this.calories});
}

class RecipeRepository {
  int _failCount = 0; // для симуляции двух сбоев

  static const List<Recipe> _db = [
    Recipe(id: '1', title: 'Овсянка',    calories: 320),
    Recipe(id: '2', title: 'Курица',     calories: 480),
    Recipe(id: '3', title: 'Греч. салат',calories: 200),
  ];

  Future<List<Recipe>> getRecipes() async {
    try {
      return await _simulateNetwork().timeout(const Duration(seconds: 5));
    } on TimeoutException {
      print('⚠️  Timeout! Возвращаем кеш');
      return _db;
    }
  }

  Future<Recipe> getRecipeById(String id) async {
    final list = await getRecipes();
    return list.firstWhere(
      (r) => r.id == id,
      orElse: () => throw NotFoundException('id=$id не найден'),
    );
  }

  Future<T> withRetry<T>(Future<T> Function() fn,
      {int maxAttempts = 3}) async {
    for (int attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        return await fn();
      } catch (e) {
        if (attempt == maxAttempts) rethrow;
        final delay = Duration(seconds: pow(2, attempt - 1).toInt());
        print('🔄 Попытка $attempt/$maxAttempts — повтор через ${delay.inSeconds}с');
        await Future.delayed(delay);
      }
    }
    throw NetworkException('Все попытки исчерпаны');
  }

  Future<List<Recipe>> _simulateNetwork() async {
    await Future.delayed(const Duration(milliseconds: 200));
    if (_failCount < 2) {
      _failCount++;
      throw NetworkException('Сервер недоступен (попытка $_failCount)');
    }
    return _db;
  }
}

Future<void> main() async {
  final repo = RecipeRepository();

  // 1. Retry
  print('=== Retry ===');
  final recipes = await repo.withRetry(repo.getRecipes);
  print('Загружено: ${recipes.map((r) => r.title).join(', ')}');

  // 2. NotFoundException
  print('\n=== NotFoundException ===');
  try {
    await repo.getRecipeById('999');
  } on NotFoundException catch (e) {
    print('❌ $e');
  }

  // 3. Future.wait — параллельно
  print('\n=== Parallel Future.wait ===');
  final sw = Stopwatch()..start();
  final results = await Future.wait([
    repo.getRecipeById('1'),
    repo.getRecipeById('2'),
    repo.getRecipeById('3'),
  ]);
  sw.stop();
  print('✅ ${results.length} рецептов за ${sw.elapsedMilliseconds}мс');
}
```

</details>
