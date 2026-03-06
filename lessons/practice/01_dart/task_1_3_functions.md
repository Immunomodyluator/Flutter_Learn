# 1.3: Функции и стрелочные функции

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 1 — Введение в Dart

---

### 1.3: Функции и стрелочные функции

🎯 **Цель шага:** Написать утилитарную библиотеку `nutrition_utils.dart` для FitMenu — функции расчёта BMR, фильтрации и сортировки рецептов с правильным использованием именованных, позиционных и опциональных параметров.

📝 **Техническое задание:**

Создай `lib/core/utils/nutrition_utils.dart` с функциями:

1. `double calculateBMR({required double weight, required double height, required int age, required String gender})` — формула Миффлина-Сан Жеора. Только именованные `required` параметры
2. `List<Recipe> filterRecipes(List<Recipe> recipes, {int? maxCalories, String? category})` — опциональные именованные; без фильтров — возвращает всё
3. `List<Recipe> sortByCalories(List<Recipe> recipes, [bool ascending = true])` — позиционный опциональный параметр (со значением по умолчанию)
4. Стрелочная функция `bool isLowCalorie(Recipe r)` — `< 300` ккал, одна строка
5. Обобщённая функция высшего порядка `List<T> filterList<T>(List<T> list, bool Function(T) predicate)` — работает с любым типом

✅ **Критерии приёмки:**
- [ ] `calculateBMR` вызывается только с именованными параметрами, без позиционных
- [ ] `filterRecipes` возвращает весь список если `maxCalories == null && category == null`
- [ ] `sortByCalories([false])` возвращает убывающий порядок, `sortByCalories()` — возрастающий
- [ ] `isLowCalorie` записана через `=>` в одну строку без `return`
- [ ] `filterList<String>` и `filterList<Recipe>` работают корректно

💡 **Подсказка:** `list.sort(...)` мутирует оригинал — сначала создай копию: `final sorted = [...list]`. Для generic-функции тип `T` выводится автоматически при передаче конкретного списка.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
// lib/core/utils/nutrition_utils.dart

/// Базовый обмен веществ по формуле Миффлина-Сан Жеора
double calculateBMR({
  required double weight, // кг
  required double height, // см
  required int age,
  required String gender, // 'male' | 'female'
}) {
  final base = 10 * weight + 6.25 * height - 5 * age;
  return gender == 'male' ? base + 5 : base - 161;
}

/// Фильтр по максимальным калориям и/или категории
List<Recipe> filterRecipes(
  List<Recipe> recipes, {
  int? maxCalories,
  String? category,
}) =>
    recipes.where((r) {
      final calOk = maxCalories == null || r.calories <= maxCalories;
      final catOk = category == null || r.category == category;
      return calOk && catOk;
    }).toList();

/// Сортировка; ascending — позиционный опциональный параметр
List<Recipe> sortByCalories(List<Recipe> recipes, [bool ascending = true]) {
  final sorted = [...recipes]; // не мутируем оригинал
  sorted.sort((a, b) => ascending
      ? a.calories.compareTo(b.calories)
      : b.calories.compareTo(a.calories));
  return sorted;
}

/// Стрелочная функция — низкокалорийный рецепт (< 300 ккал)
bool isLowCalorie(Recipe r) => r.calories < 300;

/// Обобщённая функция высшего порядка
List<T> filterList<T>(List<T> list, bool Function(T) predicate) =>
    list.where(predicate).toList();

// === Использование ===
void main() {
  final recipes = [
    Recipe(id: '1', title: 'Овсянка',    category: 'завтрак', calories: 320),
    Recipe(id: '2', title: 'Салат',      category: 'обед',    calories: 180),
    Recipe(id: '3', title: 'Курица',     category: 'обед',    calories: 480),
    Recipe(id: '4', title: 'Творог',     category: 'ужин',    calories: 140),
    Recipe(id: '5', title: 'Рис с рыбой',category: 'ужин',    calories: 390),
  ];

  // 1. BMR
  final bmr = calculateBMR(weight: 70, height: 175, age: 28, gender: 'male');
  print('BMR: ${bmr.toStringAsFixed(0)} ккал/сутки');

  // 2. Фильтрация
  final lightObeds = filterRecipes(recipes, maxCalories: 300, category: 'обед');
  print('Лёгкие обеды: ${lightObeds.map((r) => r.title).join(', ')}');

  // 3. Сортировка по убыванию (передаём false как позиционный аргумент)
  final sorted = sortByCalories(recipes, false);
  print('По убыванию: ${sorted.map((r) => '${r.title}(${r.calories})').join(', ')}');

  // 4. Стрелочная функция
  final lowCal = filterList<Recipe>(recipes, isLowCalorie);
  print('Низкокалорийные: ${lowCal.map((r) => r.title).join(', ')}');

  // 5. filterList<String>
  final shortNames = filterList<String>(
    recipes.map((r) => r.title).toList(),
    (name) => name.length <= 6,
  );
  print('Короткие названия: $shortNames');
}

class Recipe {
  final String id;
  final String title;
  final String category;
  final double calories;
  const Recipe({required this.id, required this.title,
      required this.category, required this.calories});
}
```

</details>
