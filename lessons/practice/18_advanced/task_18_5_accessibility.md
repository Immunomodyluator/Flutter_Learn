# 18.5: Доступность — Semantics и a11y аудит

> Project: FitMenu | Глава 18 — Advanced

### 18.5: Accessibility — Semantics, TalkBack/VoiceOver аудит FitMenu

🎯 **Цель шага:** Сделать FitMenu доступным для пользователей с ограниченными возможностями: добавить `Semantics` к кастомным виджетам, обеспечить правильный порядок фокуса и пройти аудит через `SemanticsService`.

---

📝 **Техническое задание:**

**1. Семантика для CalorieRingChart:**
```dart
// CustomPainter не читается скрин-ридером — оборачиваем Semantics
Semantics(
  label: 'Диаграмма калорий',
  value: '${data.calories.toInt()} из ${data.targetCalories.toInt()} килокалорий потреблено, '
         '${((data.calories / data.targetCalories) * 100).toInt()} процентов',
  child: CalorieRingChart(data: data),
)
```

**2. Семантика для RecipeCard:**
```dart
Semantics(
  label: 'Рецепт: ${recipe.title}',
  hint:  'Нажмите, чтобы открыть',
  value: '${recipe.calories} килокалорий, время приготовления ${recipe.cookingTimeMinutes} минут',
  button: true,
  onTap: onTap,
  child: _RecipeCardContent(recipe: recipe),
)
```

**3. ExcludeSemantics для декоративных элементов:**
```dart
// Декоративные иконки не должны читаться скрин-ридером
ExcludeSemantics(
  child: Icon(Icons.star, color: Colors.amber),
)
```

**4. MergeSemantics — объединяем несколько виджетов:**
```dart
MergeSemantics(
  child: Row(
    children: [
      const Icon(Icons.local_fire_department),
      const SizedBox(width: 4),
      Text('${recipe.calories} ккал'),
    ],
  ),
)
```

**5. Фокус-порядок с FocusTraversalGroup:**
```dart
FocusTraversalGroup(
  policy: OrderedTraversalPolicy(),
  child: Column(children: [
    FocusTraversalOrder(order: const NumericFocusOrder(1), child: _TitleField()),
    FocusTraversalOrder(order: const NumericFocusOrder(2), child: _CaloriesField()),
    FocusTraversalOrder(order: const NumericFocusOrder(3), child: _SaveButton()),
  ]),
)
```

**6. Аудит в тестах:**
```dart
testWidgets('a11y аудит RecipeCard', (tester) async {
  await tester.pumpWidget(MaterialApp(
    home: Scaffold(body: RecipeCard(recipe: testRecipe, onTap: () {})),
  ));

  final checker = AccessibilityGuideline.androidTapTargetGuideline;
  expect(tester, meetsGuideline(checker));
  expect(tester, meetsGuideline(labeledTapTargetGuideline));
  expect(tester, meetsGuideline(textContrastGuideline));
});
```

---

✅ **Критерии приёмки:**
- [ ] `CalorieRingChart` читается TalkBack/VoiceOver с числовым значением
- [ ] Все кнопки и кликабельные виджеты имеют `Semantics(button: true)`
- [ ] Декоративные изображения обёрнуты в `ExcludeSemantics`
- [ ] `meetsGuideline(androidTapTargetGuideline)` — минимальный tap target 48×48
- [ ] `meetsGuideline(textContrastGuideline)` — контраст текста ≥ 4.5:1
- [ ] Порядок фокуса при навигации клавиатурой логичен (сверху вниз)

---

💡 **Подсказка:** Включи TalkBack (Android): Настройки → Спец. возможности → TalkBack. Включи VoiceOver (iOS): Настройки → Универсальный доступ → VoiceOver. `SemanticsDebugger` виджет показывает семантическое дерево прямо в приложении.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/presentation/widgets/accessible_recipe_card.dart

class AccessibleRecipeCard extends StatelessWidget {
  const AccessibleRecipeCard({
    super.key,
    required this.recipe,
    required this.onTap,
  });

  final Recipe  recipe;
  final VoidCallback onTap;

  @override
  Widget build(BuildContext context) {
    final caloriesPercent = ((recipe.calories / 2000) * 100).toInt();

    return Semantics(
      label:  'Рецепт ${recipe.title}',
      value:  '${recipe.calories} килокалорий, '
              '$caloriesPercent процентов от дневной нормы, '
              'время приготовления ${recipe.cookingTimeMinutes} минут',
      hint:   'Дважды нажмите для открытия рецепта',
      button: true,
      onTap:  onTap,
      child: Card(
        child: InkWell(
          onTap: onTap,
          child: Row(
            children: [
              // Фото — декоративное
              ExcludeSemantics(
                child: RecipeImage(imageUrl: recipe.imageUrl, width: 100, height: 100),
              ),
              Expanded(
                child: Padding(
                  padding: const EdgeInsets.all(12),
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text(recipe.title, style: Theme.of(context).textTheme.titleMedium),
                      const SizedBox(height: 4),
                      // Объединяем иконку + текст в один элемент
                      MergeSemantics(
                        child: Row(
                          children: [
                            ExcludeSemantics(child: const Icon(Icons.local_fire_department, size: 16)),
                            Text(' ${recipe.calories} ккал'),
                          ],
                        ),
                      ),
                    ],
                  ),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

```dart
// test/features/recipes/widgets/accessibility_test.dart

import 'package:flutter_test/flutter_test.dart';

void main() {
  final recipe = Recipe(
    id: '1', title: 'Куриный суп',
    imageUrl: '', calories: 280, cookingTimeMinutes: 30,
  );

  group('Accessibility', () {
    testWidgets('RecipeCard: tap target ≥ 48dp', (tester) async {
      await tester.pumpWidget(MaterialApp(
        home: Scaffold(body: AccessibleRecipeCard(recipe: recipe, onTap: () {})),
      ));
      await expectLater(
        tester,
        meetsGuideline(androidTapTargetGuideline),
      );
    });

    testWidgets('RecipeCard: текстовый контраст ≥ 4.5:1', (tester) async {
      await tester.pumpWidget(MaterialApp(
        theme: AppTheme.lightTheme,
        home: Scaffold(body: AccessibleRecipeCard(recipe: recipe, onTap: () {})),
      ));
      await expectLater(tester, meetsGuideline(textContrastGuideline));
    });
  });
}
```

---

## 🎉 Поздравляем! Проект FitMenu завершён

К этому моменту приложение включает:
- **Модели данных** Dart с null safety и sealed classes
- **Декларативный UI** с Material 3 и адаптивным layout
- **Навигацию** через GoRouter с deep links
- **State Management** Riverpod + Cubit
- **Сетевой слой** Dio + json_serializable
- **Локальное хранилище** Drift + Hive
- **Firebase** Auth + Firestore + FCM
- **Анимации** — Hero, Lottie, CustomPainter
- **Clean Architecture** — Domain/Data/Presentation
- **Тестирование** — Unit, Widget, Integration, BDD
- **CI/CD** — GitHub Actions + Fastlane
- **Кроссплатформенность** — Mobile + Web + Desktop
- **Производительность** — const, RepaintBoundary, кэширование
- **Безопасность** — Keychain, certificate pinning
- **Доступность** — Semantics, a11y аудит

</details>
