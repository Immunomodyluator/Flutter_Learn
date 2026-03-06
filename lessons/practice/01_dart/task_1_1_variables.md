# 1.1: Переменные и типы данных

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 1 — Введение в Dart

---

### 1.1: Переменные и типы данных

🎯 **Цель шага:** Определить поля модели `NutritionFact` с правильными Dart-типами и продемонстрировать разницу между `var`, `final`, `const` и `late` на реальных данных о питании.

📝 **Техническое задание:**

Создай файл `lib/core/domain/models/nutrition_fact.dart` с классом `NutritionFact`.
Затем в `bin/main.dart` напиши скрипт, который:

1. Объявляет `const NutritionFact breakfast` — фиксированное типовое меню завтрака (не меняется)
2. Объявляет `final double caloriesWithActivity` — расчёт calories × 1.1 (считается один раз)
3. Объявляет `late String mealDescription` — строка, которую заполняешь **после** создания объекта
4. Использует String-интерполяцию для вывода всех значений
5. Показывает `runtimeType` для `int`, `double`, `String`, `bool`

Поля `NutritionFact`: `calories: double`, `protein: double`, `fat: double`, `carbs: double`.

✅ **Критерии приёмки:**
- [ ] `const NutritionFact` компилируется — конструктор помечен `const`, все поля `final`
- [ ] `final` переменная не переприсваивается после инициализации
- [ ] `late` переменная обращается только после того, как присвоена (нет `LateInitializationError`)
- [ ] Нет ни одного `var` или `dynamic` — только явные типы
- [ ] `dart run bin/main.dart` выводит корректные значения без ошибок

💡 **Подсказка:** `const` применимо лишь если все поля класса `final` и конструктор помечен `const`. `late` — это «обещание» компилятору: «я обязательно инициализирую до первого чтения».

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
// lib/core/domain/models/nutrition_fact.dart

/// Макронутриенты блюда или дневного рациона
class NutritionFact {
  final double calories;
  final double protein; // белки, г
  final double fat;     // жиры, г
  final double carbs;   // углеводы, г

  // const конструктор: все экземпляры с known значениями — compile-time константы
  const NutritionFact({
    required this.calories,
    required this.protein,
    required this.fat,
    required this.carbs,
  });

  @override
  String toString() =>
      '${calories.toStringAsFixed(0)} ккал | '
      'Б:${protein}г Ж:${fat}г У:${carbs}г';
}

// bin/main.dart

void main() {
  // const — значение известно на этапе компиляции, неизменяемо
  const NutritionFact breakfast = NutritionFact(
    calories: 320.0,
    protein: 12.0,
    fat: 6.0,
    carbs: 54.0,
  );

  // final — вычисляется один раз в runtime, не переприсваивается
  final double caloriesWithActivity = breakfast.calories * 1.1;

  // late — компилятор доверяет что значение будет до первого обращения
  late String mealDescription;
  mealDescription = 'Завтрак: овсянка с ягодами — '
      '${breakfast.calories.toStringAsFixed(0)} ккал';

  print('🥣 $mealDescription');
  print('🔥 С учётом активности: ${caloriesWithActivity.toStringAsFixed(0)} ккал');
  print('📊 Состав: $breakfast');

  // Явные типы и их runtimeType
  int cookTimeMin = 10;
  double servingWeight = 250.5;
  String unit = 'г';
  bool isVegetarian = true;

  print('\nТипы данных:');
  print('  cookTimeMin ($cookTimeMin): ${cookTimeMin.runtimeType}');
  print('  servingWeight ($servingWeight): ${servingWeight.runtimeType}');
  print('  unit ($unit): ${unit.runtimeType}');
  print('  isVegetarian ($isVegetarian): ${isVegetarian.runtimeType}');
}
```

**Ожидаемый вывод:**
```
🥣 Завтрак: овсянка с ягодами — 320 ккал
🔥 С учётом активности: 352 ккал
📊 Состав: 320 ккал | Б:12.0г Ж:6.0г У:54.0г

Типы данных:
  cookTimeMin (10): int
  servingWeight (250.5): double
  unit (г): String
  isVegetarian (true): bool
```

</details>
