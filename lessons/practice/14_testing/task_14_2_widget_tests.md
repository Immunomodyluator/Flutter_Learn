# 14.2: Widget-тесты — тестирование UI компонентов

> Project: FitMenu | Глава 14 — Testing

### 14.2: Widget-тесты для RecipeCard и NutritionStatsBar

🎯 **Цель шага:** Написать widget-тесты для компонентов FitMenu — проверить что виджеты корректно отображают данные, реагируют на взаимодействие и не вызывают overflow.

---

📝 **Техническое задание:**

Реализуй тесты в `test/features/recipes/widgets/`.

**Тесты для RecipeCard:**
```dart
group('RecipeCard', () {
  final testRecipe = Recipe(
    id: '1', title: 'Куриный суп',
    imageUrl: 'https://example.com/img.jpg',
    calories: 280, cookingTimeMinutes: 30,
  );

  testWidgets('отображает название рецепта', (tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: RecipeCard(recipe: testRecipe, onTap: () {}),
      ),
    );
    expect(find.text('Куриный суп'), findsOneWidget);
  });

  testWidgets('отображает калории', (tester) async { ... });

  testWidgets('вызывает onTap при нажатии', (tester) async {
    bool tapped = false;
    await tester.pumpWidget(
      MaterialApp(home: RecipeCard(recipe: testRecipe, onTap: () => tapped = true)),
    );
    await tester.tap(find.byType(GestureDetector));
    expect(tapped, isTrue);
  });

  testWidgets('нет overflow на маленьком экране', (tester) async {
    tester.view.physicalSize = const Size(320, 480);
    tester.view.devicePixelRatio = 1.0;
    await tester.pumpWidget(...);
    expect(tester.takeException(), isNull);
    addTearDown(tester.view.resetPhysicalSize);
  });
});
```

**Тесты для NutritionStatsBar:**
```dart
testWidgets('показывает прогресс 62% при 1240/2000 ккал', (tester) async {
  await tester.pumpWidget(MaterialApp(
    home: Scaffold(body: NutritionStatsBar(stats: NutritionStats(
      calories: 1240, targetCalories: 2000,
      protein: 85, fat: 42, carbs: 165,
    ))),
  ));
  final progress = tester.widget<LinearProgressIndicator>(
    find.byType(LinearProgressIndicator));
  expect(progress.value, closeTo(0.62, 0.01));
});
```

---

✅ **Критерии приёмки:**
- [ ] `tester.pumpWidget` оборачивает виджет в `MaterialApp`
- [ ] `find.text()`, `find.byType()`, `find.byKey()` используются корректно
- [ ] `tester.tap()` + `tester.pump()` симулирует нажатие
- [ ] Тест на overflow устанавливает малый размер экрана
- [ ] `closeTo(value, delta)` для проверки double значений
- [ ] `addTearDown` сбрасывает настройки экрана после теста

---

💡 **Подсказка:** `tester.pump()` — один кадр рендеринга. `tester.pumpAndSettle()` — ждёт завершения всех анимаций. `find.descendant(of: find.byType(X), matching: find.text('Y'))` — поиск дочернего виджета. `tester.view.physicalSize` (Flutter 3.10+) устанавливает размер экрана в тесте.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// test/features/recipes/widgets/recipe_card_test.dart

import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  final testRecipe = Recipe(
    id: '1',
    title: 'Куриный суп с овощами',
    imageUrl: 'https://example.com/chicken_soup.jpg',
    calories: 280,
    cookingTimeMinutes: 30,
  );

  group('RecipeCard', () {
    Widget buildWidget({VoidCallback? onTap}) => MaterialApp(
          home: Scaffold(
            body: RecipeCard(
              recipe: testRecipe,
              onTap: onTap ?? () {},
            ),
          ),
        );

    testWidgets('отображает название рецепта', (tester) async {
      await tester.pumpWidget(buildWidget());
      expect(find.text('Куриный суп с овощами'), findsOneWidget);
    });

    testWidgets('отображает калории с эмодзи огня', (tester) async {
      await tester.pumpWidget(buildWidget());
      expect(find.textContaining('280 ккал'), findsOneWidget);
    });

    testWidgets('вызывает onTap при нажатии на карточку', (tester) async {
      var tapped = false;
      await tester.pumpWidget(buildWidget(onTap: () => tapped = true));
      await tester.tap(find.byType(GestureDetector));
      await tester.pump();
      expect(tapped, isTrue);
    });

    testWidgets('нет RenderFlex overflow на экране 320px', (tester) async {
      tester.view.physicalSize = const Size(320, 600);
      tester.view.devicePixelRatio = 1.0;

      await tester.pumpWidget(buildWidget());
      await tester.pump();

      expect(tester.takeException(), isNull);
      addTearDown(tester.view.resetPhysicalSize);
    });

    testWidgets('показывает время приготовления', (tester) async {
      await tester.pumpWidget(buildWidget());
      expect(find.textContaining('30 мин'), findsOneWidget);
    });
  });

  group('NutritionStatsBar', () {
    testWidgets('LinearProgressIndicator показывает 0.62 при 1240/2000', (tester) async {
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: NutritionStatsBar(
              stats: const NutritionStats(
                calories: 1240,
                targetCalories: 2000,
                protein: 85,
                fat: 42,
                carbs: 165,
              ),
            ),
          ),
        ),
      );
      final indicator = tester.widget<LinearProgressIndicator>(
        find.byType(LinearProgressIndicator),
      );
      expect(indicator.value, closeTo(0.62, 0.01));
    });
  });
}
```

</details>
