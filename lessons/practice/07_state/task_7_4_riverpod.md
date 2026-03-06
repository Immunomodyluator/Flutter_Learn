# 7.4: Riverpod — план питания через StateNotifierProvider

> Project: FitMenu | Глава 7 — State Management

### 7.4: MealPlanNotifier с Riverpod и StateNotifierProvider

🎯 **Цель шага:** Реализовать управление планом питания на день через Riverpod — добавление/удаление блюд с автоматическим пересчётом суммарных КБЖУ, используя `StateNotifier` и `StateNotifierProvider`.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  flutter_riverpod: ^2.4.0
```

Реализуй `lib/features/meal_plan/providers/meal_plan_provider.dart`.

**Модель:**
```dart
class MealEntry {
  const MealEntry({
    required this.id,
    required this.recipeId,
    required this.title,
    required this.calories,
    required this.protein,
    required this.fat,
    required this.carbs,
    required this.mealType,
  });
  final String id;
  final String recipeId;
  final String title;
  final double calories;
  final double protein;
  final double fat;
  final double carbs;
  final MealType mealType;
}
```

**MealPlanState:**
```dart
class MealPlanState {
  const MealPlanState({this.entries = const []});
  final List<MealEntry> entries;

  double get totalCalories => entries.fold(0, (s, e) => s + e.calories);
  double get totalProtein  => entries.fold(0, (s, e) => s + e.protein);
  double get totalFat      => entries.fold(0, (s, e) => s + e.fat);
  double get totalCarbs    => entries.fold(0, (s, e) => s + e.carbs);

  List<MealEntry> byType(MealType type) =>
      entries.where((e) => e.mealType == type).toList();

  MealPlanState copyWith({List<MealEntry>? entries}) =>
      MealPlanState(entries: entries ?? this.entries);
}
```

**MealPlanNotifier:**
```dart
class MealPlanNotifier extends StateNotifier<MealPlanState> {
  MealPlanNotifier() : super(const MealPlanState());

  void addEntry(MealEntry entry) { ... }
  void removeEntry(String id) { ... }
  void clearAll() { ... }
}

final mealPlanProvider =
    StateNotifierProvider<MealPlanNotifier, MealPlanState>(
  (ref) => MealPlanNotifier(),
);
```

**Провайдеры-производные (селекторы):**
```dart
// Только калории — перестраивается только при изменении калорий
final totalCaloriesProvider = Provider<double>(
  (ref) => ref.watch(mealPlanProvider).totalCalories,
);
```

**В main.dart:**
```dart
runApp(
  ProviderScope(child: FitMenuApp()),
);
```

**В виджете:**
```dart
final state = ref.watch(mealPlanProvider);
ref.read(mealPlanProvider.notifier).addEntry(entry);
```

---

✅ **Критерии приёмки:**
- [ ] `ProviderScope` оборачивает `FitMenuApp` в `main()`
- [ ] `StateNotifier` — иммутабельные обновления через `copyWith`
- [ ] `ConsumerWidget` или `Consumer` используются в UI (не `StatefulWidget`)
- [ ] Суммарные КБЖУ пересчитываются автоматически при добавлении/удалении
- [ ] `ref.read` в обработчиках, `ref.watch` в `build`
- [ ] Производный провайдер `totalCaloriesProvider` минимизирует перестройку UI

---

💡 **Подсказка:** В Riverpod 2.x предпочитай `@riverpod` аннотации (code generation) для новых проектов, но `StateNotifierProvider` тоже рабочий вариант. `ref.watch(provider.select((s) => s.totalCalories))` — ещё один способ подписаться только на часть состояния.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_plan/providers/meal_plan_provider.dart

import 'package:flutter_riverpod/flutter_riverpod.dart';
// import MealType из task_6_4

class MealEntry {
  const MealEntry({
    required this.id,
    required this.recipeId,
    required this.title,
    required this.calories,
    required this.protein,
    required this.fat,
    required this.carbs,
    required this.mealType,
  });

  final String id;
  final String recipeId;
  final String title;
  final double calories;
  final double protein;
  final double fat;
  final double carbs;
  final MealType mealType;
}

class MealPlanState {
  const MealPlanState({this.entries = const []});
  final List<MealEntry> entries;

  double get totalCalories => entries.fold(0.0, (s, e) => s + e.calories);
  double get totalProtein  => entries.fold(0.0, (s, e) => s + e.protein);
  double get totalFat      => entries.fold(0.0, (s, e) => s + e.fat);
  double get totalCarbs    => entries.fold(0.0, (s, e) => s + e.carbs);

  List<MealEntry> byType(MealType type) =>
      entries.where((e) => e.mealType == type).toList();

  MealPlanState copyWith({List<MealEntry>? entries}) =>
      MealPlanState(entries: entries ?? this.entries);
}

class MealPlanNotifier extends StateNotifier<MealPlanState> {
  MealPlanNotifier() : super(const MealPlanState());

  void addEntry(MealEntry entry) {
    state = state.copyWith(entries: [...state.entries, entry]);
  }

  void removeEntry(String id) {
    state = state.copyWith(
      entries: state.entries.where((e) => e.id != id).toList(),
    );
  }

  void clearAll() {
    state = const MealPlanState();
  }
}

// Основной провайдер
final mealPlanProvider =
    StateNotifierProvider<MealPlanNotifier, MealPlanState>(
  (ref) => MealPlanNotifier(),
);

// Производные провайдеры-селекторы
final totalCaloriesProvider = Provider<double>(
  (ref) => ref.watch(mealPlanProvider).totalCalories,
);

final breakfastEntriesProvider = Provider<List<MealEntry>>(
  (ref) => ref.watch(mealPlanProvider).byType(MealType.breakfast),
);
```

```dart
// Виджет с ConsumerWidget
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

class MealPlanSummary extends ConsumerWidget {
  const MealPlanSummary({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Подписка только на калории — не перестраивается при изменении других полей
    final calories = ref.watch(totalCaloriesProvider);
    final count = ref.watch(
      mealPlanProvider.select((s) => s.entries.length),
    );

    return Card(
      child: Padding(
        padding: const EdgeInsets.all(16),
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceAround,
          children: [
            Text('$count блюд'),
            Text('${calories.toStringAsFixed(0)} ккал'),
            ElevatedButton(
              onPressed: () => ref.read(mealPlanProvider.notifier).clearAll(),
              child: const Text('Очистить'),
            ),
          ],
        ),
      ),
    );
  }
}
```

</details>
