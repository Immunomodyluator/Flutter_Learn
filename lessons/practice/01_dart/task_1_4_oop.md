# 1.4: ООП в Dart

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 1 — Введение в Dart

---

### 1.4: ООП в Dart

🎯 **Цель шага:** Спроектировать иерархию классов FitMenu: абстрактный `BaseFood`, класс `Recipe` с именованным и фабричным конструктором, класс `MealEntry`, миксины `Printable` и `Calculable`.

📝 **Техническое задание:**

Создай `lib/core/domain/models/` со следующими классами:

1. **`abstract class BaseFood`** — абстрактный геттер `double calories`, метод `String describe()`
2. **`mixin Printable`** — метод `void printInfo()` выводит поля в JSON-подобном формате
3. **`mixin Calculable`** — метод `double scaledCalories(double servings)` и `double dailyValuePercent()`
4. **`class Recipe extends BaseFood with Printable, Calculable`**:
   - Обычный конструктор
   - `Recipe.fromJson(Map<String, dynamic> json)` — именованный
   - `factory Recipe.placeholder()` — фабричный, возвращает заглушку
5. **`class MealEntry extends BaseFood with Printable, Calculable`** — рецепт + порция + время

✅ **Критерии приёмки:**
- [ ] `BaseFood()` вызывает ошибку компиляции — нельзя создать напрямую (абстрактный)
- [ ] `Recipe.fromJson` парсит все поля без ошибок при корректном Map
- [ ] `Recipe.placeholder()` не бросает исключений — возвращает безопасную заглушку
- [ ] `MealEntry.calories` возвращает `recipe.calories * servings`
- [ ] `@override` стоит на всех переопределённых методах и геттерах

💡 **Подсказка:** Фабричный конструктор (`factory`) может возвращать другой конструктор: `factory Recipe.placeholder() => Recipe(title: '...', ...)`. Миксины не могут иметь конструктора с параметрами.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
// lib/core/domain/models/base_food.dart

abstract class BaseFood {
  String get title;
  double get calories; // абстрактный геттер

  /// Базовое описание — подклассы могут переопределить
  String describe() => '$title — ${calories.toStringAsFixed(0)} ккал';
}

mixin Printable {
  String get title;
  double get calories;

  void printInfo() {
    print('{');
    print('  "title": "$title",');
    print('  "calories": $calories');
    print('}');
  }
}

mixin Calculable {
  double get calories;

  double scaledCalories(double servings) => calories * servings;
  double dailyValuePercent([double goal = 2000]) => calories / goal * 100;
}

// lib/core/domain/models/recipe.dart

class Recipe extends BaseFood with Printable, Calculable {
  @override
  final String title;
  @override
  final double calories;
  final double protein;
  final double fat;
  final double carbs;
  final String category;

  Recipe({
    required this.title,
    required this.calories,
    required this.protein,
    required this.fat,
    required this.carbs,
    required this.category,
  });

  // Именованный конструктор — разбор ответа API
  Recipe.fromJson(Map<String, dynamic> json)
      : title    = json['title'] as String,
        calories = (json['calories'] as num).toDouble(),
        protein  = (json['protein']  as num).toDouble(),
        fat      = (json['fat']      as num).toDouble(),
        carbs    = (json['carbs']    as num).toDouble(),
        category = json['category'] as String;

  // Фабричный конструктор — безопасная заглушка
  factory Recipe.placeholder() => Recipe(
        title: 'Загружается...',
        calories: 0, protein: 0, fat: 0, carbs: 0,
        category: 'неизвестно',
      );

  @override
  String describe() =>
      '${super.describe()} | Б:${protein}г Ж:${fat}г У:${carbs}г';
}

// lib/core/domain/models/meal_entry.dart

class MealEntry extends BaseFood with Printable, Calculable {
  final Recipe recipe;
  final double servings;
  final DateTime addedAt;

  MealEntry({
    required this.recipe,
    required this.servings,
    DateTime? addedAt,
  }) : addedAt = addedAt ?? DateTime.now();

  @override
  String get title => recipe.title;

  @override
  double get calories => recipe.calories * servings; // с учётом порции

  @override
  String describe() =>
      '${recipe.title} ×$servings = ${calories.toStringAsFixed(0)} ккал';
}

// bin/oop_demo.dart
void main() {
  // fromJson
  final chicken = Recipe.fromJson({
    'title': 'Куриная грудка', 'calories': 480,
    'protein': 52.0, 'fat': 8.0, 'carbs': 44.0, 'category': 'обед',
  });
  print(chicken.describe());
  chicken.printInfo();

  // placeholder
  final stub = Recipe.placeholder();
  print('\n${stub.title}'); // Загружается...

  // MealEntry — 1.5 порции
  final entry = MealEntry(recipe: chicken, servings: 1.5);
  print('\n${entry.describe()}');
  print('${entry.dailyValuePercent().toStringAsFixed(1)}% суточной нормы');

  // Полиморфизм через BaseFood
  final List<BaseFood> foods = [chicken, entry];
  for (final food in foods) {
    print('— ${food.describe()}');
  }
}
```

</details>
