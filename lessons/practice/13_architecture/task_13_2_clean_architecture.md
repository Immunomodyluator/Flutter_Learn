# 13.2: Clean Architecture — слои для фичи рецептов

> Project: FitMenu | Глава 13 — Architecture

### 13.2: Clean Architecture — Domain, Data, Presentation слои

🎯 **Цель шага:** Рефакторинг фичи "Рецепты" по принципам Clean Architecture — выделить Domain (UseCase, Entity, Repository интерфейс), Data (Repository impl, DataSource) и Presentation (ViewModel/BLoC) слои.

---

📝 **Техническое задание:**

Структура папок для фичи `recipes`:
```
lib/features/recipes/
  ├── domain/
  │   ├── entities/recipe.dart           ← чистая Dart-модель без зависимостей
  │   ├── repositories/recipe_repository.dart  ← абстрактный интерфейс
  │   └── usecases/
  │       ├── get_popular_recipes.dart
  │       └── search_recipes.dart
  ├── data/
  │   ├── models/recipe_dto.dart         ← DTO + json_serializable
  │   ├── mappers/recipe_mapper.dart     ← DTO → Domain
  │   ├── datasources/
  │   │   ├── recipe_remote_datasource.dart
  │   │   └── recipe_local_datasource.dart
  │   └── repositories/recipe_repository_impl.dart
  └── presentation/
      ├── screens/recipes_screen.dart
      ├── viewmodels/recipes_view_model.dart
      └── widgets/
```

**Domain — Repository интерфейс:**
```dart
abstract class RecipeRepository {
  Future<Result<List<Recipe>>> getPopular();
  Future<Result<List<Recipe>>> search(String query);
  Future<Result<Recipe>> getById(String id);
}
```

**Domain — UseCase:**
```dart
class SearchRecipesUseCase {
  const SearchRecipesUseCase(this._repository);
  final RecipeRepository _repository;

  Future<Result<List<Recipe>>> call(String query) =>
      _repository.search(query);
}
```

**Data — Repository Implementation:**
```dart
class RecipeRepositoryImpl implements RecipeRepository {
  const RecipeRepositoryImpl(this._remote, this._local);
  final RecipeRemoteDataSource _remote;
  final RecipeLocalDataSource _local;

  @override
  Future<Result<List<Recipe>>> getPopular() async { ... }
}
```

**Правило зависимостей:** Domain не знает о Data и Presentation. Data зависит от Domain (реализует интерфейсы). Presentation зависит от Domain (использует UseCase).

---

✅ **Критерии приёмки:**
- [ ] `domain/entities/recipe.dart` не импортирует ничего кроме dart:core
- [ ] `RecipeRepository` — абстрактный класс в domain, реализован в data
- [ ] `UseCase.call()` — метод, позволяющий вызывать как функцию: `useCase(query)`
- [ ] `Presentation` зависит от `domain`, не от `data`
- [ ] `RecipeRepositoryImpl` зависит от двух DataSource (remote + local)
- [ ] Mapper находится в data слое, не в domain

---

💡 **Подсказка:** Clean Architecture — это про направление зависимостей: всё зависит от Domain, Domain не зависит ни от чего. UseCase с методом `call()` позволяет использовать как `final result = await searchUseCase('chicken')`. Dependency Inversion: Presentation получает `RecipeRepository` (интерфейс), а не `RecipeRepositoryImpl`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/domain/entities/recipe.dart
// Чистая модель — нет зависимостей от фреймворков
class Recipe {
  const Recipe({
    required this.id,
    required this.title,
    required this.imageUrl,
    required this.calories,
    required this.cookingTimeMinutes,
    this.protein = 0,
    this.fat = 0,
    this.carbs = 0,
  });

  final String id;
  final String title;
  final String imageUrl;
  final int calories;
  final int cookingTimeMinutes;
  final double protein;
  final double fat;
  final double carbs;
}
```

```dart
// lib/features/recipes/domain/repositories/recipe_repository.dart
abstract class RecipeRepository {
  Future<Result<List<Recipe>>> getPopular();
  Future<Result<List<Recipe>>> search(String query);
  Future<Result<Recipe>> getById(String id);
}
```

```dart
// lib/features/recipes/domain/usecases/search_recipes_use_case.dart
class SearchRecipesUseCase {
  const SearchRecipesUseCase(this._repository);
  final RecipeRepository _repository;

  Future<Result<List<Recipe>>> call(String query) =>
      query.trim().isEmpty ? _repository.getPopular() : _repository.search(query);
}
```

```dart
// lib/features/recipes/data/repositories/recipe_repository_impl.dart
class RecipeRepositoryImpl implements RecipeRepository {
  const RecipeRepositoryImpl(this._remote, this._local);
  final RecipeRemoteDataSource _remote;
  final RecipeLocalDataSource  _local;

  @override
  Future<Result<List<Recipe>>> getPopular() async {
    try {
      // Cache-first стратегия
      if (_local.isCacheValid) {
        return Success(_local.getCachedRecipes());
      }
      final dtos = await _remote.getPopularRecipes();
      final recipes = dtos.toDomain();
      await _local.cacheRecipes(recipes);
      return Success(recipes);
    } catch (e) {
      return Failure(UnknownError(e.toString()));
    }
  }

  @override
  Future<Result<List<Recipe>>> search(String query) async {
    try {
      final dtos = await _remote.searchRecipes(query);
      return Success(dtos.toDomain());
    } catch (e) {
      return Failure(UnknownError(e.toString()));
    }
  }

  @override
  Future<Result<Recipe>> getById(String id) async {
    try {
      final dto = await _remote.getRecipeById(int.parse(id));
      return Success(dto.toDomain());
    } catch (e) {
      return Failure(UnknownError(e.toString()));
    }
  }
}
```

</details>
