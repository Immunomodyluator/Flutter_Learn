# 1.7: Streams

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 1 — Введение в Dart

---

### 1.7: Streams

🎯 **Цель шага:** Добавить в FitMenu реактивный «монитор питания» — `MealTracker` со `StreamController`, который транслирует обновления КБЖУ в реальном времени при каждом добавлении блюда в дневник.

📝 **Техническое задание:**

Создай `lib/core/services/meal_tracker.dart`:

1. `StreamController<NutritionSummary>.broadcast()` — broadcast, чтобы несколько подписчиков могли слушать
2. `Stream<NutritionSummary> get nutritionStream` — публичный стрим текущего КБЖУ
3. `void addMeal(Recipe recipe, double servings)` — пересчитывает `_current` и эмитит через `_controller.add`
4. `Stream<bool> get isGoalReached` — трансформация через `.map()`: `true` когда `calories >= dailyGoal`
5. В `main()` подпишись на оба стрима, добавь 5 блюд с задержками, затем вызови `StreamSubscription.cancel()` и `controller.close()`

✅ **Критерии приёмки:**
- [ ] `StreamController` — `broadcast` (несколько `.listen()` без ошибки)
- [ ] После каждого `addMeal` стрим **немедленно** эмитит обновлённое значение
- [ ] `isGoalReached` — отдельный стрим через `.map()`, не дублирует логику
- [ ] `cancel()` вызывается для каждой `StreamSubscription` в конце
- [ ] `controller.close()` вызывается в методе `dispose()`

💡 **Подсказка:** `StreamController.broadcast()` — для нескольких слушателей. Обычный `StreamController()` выбрасывает ошибку при втором `.listen()`. В Flutter `StreamBuilder` делает это автоматически, но понимать механизм важно.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
import 'dart:async';

class Recipe {
  final String title;
  final double calories, protein, fat, carbs;
  const Recipe({required this.title, required this.calories,
      required this.protein, required this.fat, required this.carbs});
}

class NutritionSummary {
  final double calories, protein, fat, carbs;
  final int mealCount;

  const NutritionSummary({
    this.calories = 0, this.protein = 0,
    this.fat = 0, this.carbs = 0, this.mealCount = 0,
  });

  NutritionSummary add(Recipe r, double s) => NutritionSummary(
    calories:  calories  + r.calories  * s,
    protein:   protein   + r.protein   * s,
    fat:       fat       + r.fat       * s,
    carbs:     carbs     + r.carbs     * s,
    mealCount: mealCount + 1,
  );

  @override
  String toString() =>
      '🍽 #$mealCount | ${calories.toStringAsFixed(0)} ккал '
      'Б:${protein.toStringAsFixed(1)} Ж:${fat.toStringAsFixed(1)} '
      'У:${carbs.toStringAsFixed(1)}';
}

class MealTracker {
  final double dailyGoal;
  NutritionSummary _current = const NutritionSummary();

  // broadcast — несколько подписчиков одновременно
  final _controller = StreamController<NutritionSummary>.broadcast();

  MealTracker({this.dailyGoal = 2000}) {
    _controller.add(_current); // начальное значение
  }

  Stream<NutritionSummary> get nutritionStream => _controller.stream;

  // Трансформация: достигнута ли цель
  Stream<bool> get isGoalReached =>
      nutritionStream.map((s) => s.calories >= dailyGoal);

  void addMeal(Recipe recipe, double servings) {
    _current = _current.add(recipe, servings);
    _controller.add(_current);
  }

  void dispose() => _controller.close();
}

Future<void> main() async {
  final tracker = MealTracker(dailyGoal: 1200);

  // Два независимых подписчика (broadcast позволяет это)
  final sub1 = tracker.nutritionStream.listen(
    (s) => print('📊 $s'),
  );
  final sub2 = tracker.isGoalReached.listen(
    (reached) { if (reached) print('🎯 Цель достигнута!'); },
  );

  final meals = [
    (Recipe(title:'Завтрак',  calories:320, protein:12, fat:6, carbs:54), 1.0),
    (Recipe(title:'Перекус',  calories:80,  protein:0,  fat:0, carbs:21), 1.0),
    (Recipe(title:'Обед',     calories:480, protein:52, fat:8, carbs:44), 1.0),
    (Recipe(title:'Ужин',     calories:350, protein:38, fat:18,carbs:2),  1.0),
    (Recipe(title:'Доп.порция',calories:180,protein:22, fat:4, carbs:10), 1.0),
  ];

  for (final (recipe, servings) in meals) {
    await Future.delayed(const Duration(milliseconds: 100));
    tracker.addMeal(recipe, servings);
  }

  // Обязательно отменяем подписки!
  await sub1.cancel();
  await sub2.cancel();
  tracker.dispose();
  print('✅ Трекер закрыт');
}
```

</details>
