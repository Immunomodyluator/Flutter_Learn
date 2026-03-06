# 13.1: MVVM — ViewModel для экрана рецептов

> Project: FitMenu | Глава 13 — Architecture

### 13.1: RecipesViewModel с MVVM паттерном

🎯 **Цель шага:** Применить паттерн MVVM к экрану рецептов — выделить `RecipesViewModel` с бизнес-логикой и состоянием, оставив View только ответственность за отображение.

---

📝 **Техническое задание:**

Реализуй `lib/features/recipes/viewmodels/recipes_view_model.dart`.

**RecipesState:**
```dart
class RecipesState {
  const RecipesState({this.recipes = const [], this.isLoading = false, this.error});
  final List<Recipe> recipes;
  final bool isLoading;
  final String? error;

  RecipesState copyWith({List<Recipe>? recipes, bool? isLoading, String? error}) => ...
}
```

**RecipesViewModel (ChangeNotifier):**
```dart
class RecipesViewModel extends ChangeNotifier {
  RecipesViewModel(this._repository);
  final RecipeRepository _repository;

  RecipesState _state = const RecipesState();
  RecipesState get state => _state;

  Future<void> loadRecipes() async { ... }
  Future<void> search(String query) async { ... }
  void clearError() { ... }
}
```

**View (StatelessWidget + Consumer):**
```dart
class RecipesScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Consumer<RecipesViewModel>(
      builder: (context, vm, _) {
        if (vm.state.isLoading) return const LoadingWidget();
        if (vm.state.error != null) return ErrorWidget(vm.state.error!);
        return RecipesList(recipes: vm.state.recipes);
      },
    );
  }
}
```

---

✅ **Критерии приёмки:**
- [ ] `RecipesViewModel` не импортирует Flutter виджеты (`dart:ui` — ок, `material` — нет)
- [ ] View не содержит бизнес-логики — только `vm.loadRecipes()` вызовы
- [ ] `RecipesState` иммутабельный с `copyWith`
- [ ] `notifyListeners()` вызывается после каждого обновления `_state`
- [ ] `ChangeNotifierProvider<RecipesViewModel>` создаётся вне экрана

---

💡 **Подсказка:** ViewModel — посредник между View и Model. Он знает о состоянии UI, но не о виджетах. `ChangeNotifier` — минимальная реализация MVVM в Flutter без лишних зависимостей. `context.select<RecipesViewModel, bool>((vm) => vm.state.isLoading)` — подписка только на часть состояния.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/viewmodels/recipes_view_model.dart

import 'package:flutter/foundation.dart'; // только ChangeNotifier, не material

class RecipesState {
  const RecipesState({
    this.recipes  = const [],
    this.isLoading = false,
    this.error,
  });

  final List<Recipe> recipes;
  final bool isLoading;
  final String? error;

  bool get hasError => error != null;
  bool get isEmpty  => recipes.isEmpty && !isLoading;

  RecipesState copyWith({
    List<Recipe>? recipes,
    bool? isLoading,
    String? error,
    bool clearError = false,
  }) =>
      RecipesState(
        recipes:   recipes   ?? this.recipes,
        isLoading: isLoading ?? this.isLoading,
        error: clearError ? null : (error ?? this.error),
      );
}

class RecipesViewModel extends ChangeNotifier {
  RecipesViewModel(this._repository);

  final RecipeRepository _repository;
  RecipesState _state = const RecipesState();
  RecipesState get state => _state;

  void _setState(RecipesState Function(RecipesState) updater) {
    _state = updater(_state);
    notifyListeners();
  }

  Future<void> loadRecipes() async {
    _setState((s) => s.copyWith(isLoading: true, clearError: true));
    final result = await _repository.getPopularRecipes();
    result.when(
      success: (recipes) => _setState(
        (s) => s.copyWith(recipes: recipes, isLoading: false),
      ),
      failure: (error) => _setState(
        (s) => s.copyWith(error: error.message, isLoading: false),
      ),
    );
  }

  Future<void> search(String query) async {
    if (query.isEmpty) { await loadRecipes(); return; }
    _setState((s) => s.copyWith(isLoading: true));
    final result = await _repository.searchRecipes(query);
    result.when(
      success: (recipes) => _setState((s) => s.copyWith(recipes: recipes, isLoading: false)),
      failure: (error)   => _setState((s) => s.copyWith(error: error.message, isLoading: false)),
    );
  }

  void clearError() => _setState((s) => s.copyWith(clearError: true));
}
```

</details>
