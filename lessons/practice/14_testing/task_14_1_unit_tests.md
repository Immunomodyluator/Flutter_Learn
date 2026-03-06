# 14.1: Unit-тесты — тестирование UseCase и ViewModel

> Project: FitMenu | Глава 14 — Testing

### 14.1: Unit-тесты для SearchRecipesUseCase и RecipesViewModel

🎯 **Цель шага:** Написать unit-тесты для `SearchRecipesUseCase` и `RecipesViewModel` с мок-репозиторием через `mockito` — проверить логику поиска, состояния загрузки и обработку ошибок.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.0
  build_runner: ^2.4.0
```

Реализуй тесты в `test/features/recipes/`.

**Мок-репозиторий:**
```dart
@GenerateMocks([RecipeRepository])
void main() { ... }
```

**Тесты для SearchRecipesUseCase:**
```dart
group('SearchRecipesUseCase', () {
  late MockRecipeRepository mockRepo;
  late SearchRecipesUseCase useCase;

  setUp(() {
    mockRepo = MockRecipeRepository();
    useCase = SearchRecipesUseCase(mockRepo);
  });

  test('возвращает результаты при непустом запросе', () async {
    final recipes = [Recipe(id: '1', title: 'Test', ...)];
    when(mockRepo.search('chicken')).thenAnswer((_) async => Success(recipes));

    final result = await useCase('chicken');

    expect(result, isA<Success<List<Recipe>>>());
    expect((result as Success).value, recipes);
    verify(mockRepo.search('chicken')).called(1);
  });

  test('вызывает getPopular при пустом запросе', () async { ... });
  test('возвращает Failure при ошибке сети', () async { ... });
});
```

**Тесты для RecipesViewModel:**
```dart
group('RecipesViewModel', () {
  test('начальное состояние — isLoading: false, recipes: []', () { ... });
  test('loadRecipes устанавливает isLoading в true во время загрузки', () async { ... });
  test('loadRecipes сохраняет рецепты в state при успехе', () async { ... });
  test('loadRecipes сохраняет ошибку при Failure', () async { ... });
});
```

---

✅ **Критерии приёмки:**
- [ ] `@GenerateMocks([RecipeRepository])` + `build_runner` генерирует моки
- [ ] `when(...).thenAnswer(...)` настраивает поведение мока
- [ ] `verify(...)` проверяет вызовы методов мока
- [ ] Тесты изолированы через `setUp()` — новый мок для каждого теста
- [ ] `expect(result, isA<Success<List<Recipe>>>())` — проверка типа Result
- [ ] Все крайние случаи покрыты: пустой запрос, ошибка сети, пустой результат

---

💡 **Подсказка:** `when(mock.method()).thenAnswer((_) async => value)` — для async методов. `thenReturn` — для sync. `verifyNever(mock.method())` — метод НЕ был вызван. `any` из mockito — любой аргумент. `argThat(predicate)` — аргумент удовлетворяющий предикату.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// test/features/recipes/search_recipes_use_case_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

@GenerateMocks([RecipeRepository])
void main() {}
```

```dart
// test/features/recipes/recipes_view_model_test.dart

import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

void main() {
  late MockRecipeRepository mockRepo;
  late RecipesViewModel viewModel;

  final testRecipes = [
    Recipe(id: '1', title: 'Куриный суп', imageUrl: '', calories: 280, cookingTimeMinutes: 30),
    Recipe(id: '2', title: 'Овсянка',    imageUrl: '', calories: 350, cookingTimeMinutes: 10),
  ];

  setUp(() {
    mockRepo  = MockRecipeRepository();
    viewModel = RecipesViewModel(mockRepo);
  });

  tearDown(() => viewModel.dispose());

  group('начальное состояние', () {
    test('isLoading = false', () => expect(viewModel.state.isLoading, isFalse));
    test('recipes = []',      () => expect(viewModel.state.recipes, isEmpty));
    test('error = null',      () => expect(viewModel.state.error, isNull));
  });

  group('loadRecipes', () {
    test('устанавливает isLoading во время загрузки', () async {
      when(mockRepo.getPopular()).thenAnswer(
        (_) async {
          await Future.delayed(const Duration(milliseconds: 10));
          return Success(testRecipes);
        },
      );

      final future = viewModel.loadRecipes();
      expect(viewModel.state.isLoading, isTrue); // во время загрузки
      await future;
      expect(viewModel.state.isLoading, isFalse); // после завершения
    });

    test('сохраняет рецепты при Success', () async {
      when(mockRepo.getPopular()).thenAnswer((_) async => Success(testRecipes));
      await viewModel.loadRecipes();
      expect(viewModel.state.recipes, equals(testRecipes));
      expect(viewModel.state.error, isNull);
    });

    test('сохраняет ошибку при Failure', () async {
      when(mockRepo.getPopular()).thenAnswer(
        (_) async => const Failure(NetworkError('Нет сети', statusCode: 0)),
      );
      await viewModel.loadRecipes();
      expect(viewModel.state.error, isNotNull);
      expect(viewModel.state.recipes, isEmpty);
    });
  });
}
```

</details>
