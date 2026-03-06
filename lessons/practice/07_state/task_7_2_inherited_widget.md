# 7.2: InheritedWidget — передача темы питания вниз по дереву

> Project: FitMenu | Глава 7 — State Management

### 7.2: InheritedWidget для передачи настроек питания без Provider

🎯 **Цель шага:** Реализовать `NutritionSettings` через `InheritedWidget` — понять базовый механизм передачи данных вниз по дереву виджетов, на котором построены все стейт-менеджеры Flutter.

---

📝 **Техническое задание:**

Реализуй `lib/core/inherited/nutrition_settings_provider.dart`.

**NutritionSettings — модель настроек:**
```dart
class NutritionSettings {
  const NutritionSettings({
    required this.targetCalories,
    required this.targetProtein,
    required this.targetFat,
    required this.targetCarbs,
  });

  final int targetCalories; // ккал/день
  final int targetProtein;  // г/день
  final int targetFat;      // г/день
  final int targetCarbs;    // г/день

  // Стандартные значения для 2000 ккал/день
  static const defaults = NutritionSettings(
    targetCalories: 2000,
    targetProtein: 150,
    targetFat: 65,
    targetCarbs: 250,
  );
}
```

**NutritionSettingsInherited:**
```dart
class NutritionSettingsInherited extends InheritedWidget {
  const NutritionSettingsInherited({
    super.key,
    required this.settings,
    required super.child,
  });

  final NutritionSettings settings;

  // Статический метод доступа
  static NutritionSettings of(BuildContext context) {
    final inherited = context.dependOnInheritedWidgetOfExactType<NutritionSettingsInherited>();
    assert(inherited != null, 'NutritionSettingsInherited не найден в дереве');
    return inherited!.settings;
  }

  @override
  bool updateShouldNotify(NutritionSettingsInherited oldWidget) =>
      settings != oldWidget.settings;
}
```

**Использование в дочернем виджете:**
```dart
// Любой потомок может получить настройки:
final settings = NutritionSettingsInherited.of(context);
Text('Норма: ${settings.targetCalories} ккал');
```

**Обёртка в main:**
```dart
NutritionSettingsInherited(
  settings: NutritionSettings.defaults,
  child: MaterialApp(...),
)
```

---

✅ **Критерии приёмки:**
- [ ] `InheritedWidget` реализован с правильным `updateShouldNotify`
- [ ] Статический метод `of(context)` использует `dependOnInheritedWidgetOfExactType`
- [ ] `assert` срабатывает в debug-режиме при отсутствии провайдера в дереве
- [ ] При изменении `settings` — зависимые виджеты перестраиваются
- [ ] `NutritionSettings` реализован с `const`-конструктором и поддержкой равенства (`==`)
- [ ] Понимаешь разницу между `dependOnInheritedWidgetOfExactType` и `getInheritedWidgetOfExactType`

---

💡 **Подсказка:** `dependOnInheritedWidgetOfExactType` регистрирует зависимость — виджет перестроится при изменении данных. `getInheritedWidgetOfExactType` только читает без регистрации зависимости (используй для `initState`). Именно этот паттерн лежит в основе `Theme.of(context)` и `MediaQuery.of(context)`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/inherited/nutrition_settings_provider.dart

import 'package:flutter/material.dart';

// Иммутабельная модель настроек с поддержкой == через Equatable или вручную
class NutritionSettings {
  const NutritionSettings({
    required this.targetCalories,
    required this.targetProtein,
    required this.targetFat,
    required this.targetCarbs,
  });

  final int targetCalories;
  final int targetProtein;
  final int targetFat;
  final int targetCarbs;

  static const defaults = NutritionSettings(
    targetCalories: 2000,
    targetProtein: 150,
    targetFat: 65,
    targetCarbs: 250,
  );

  NutritionSettings copyWith({
    int? targetCalories,
    int? targetProtein,
    int? targetFat,
    int? targetCarbs,
  }) =>
      NutritionSettings(
        targetCalories: targetCalories ?? this.targetCalories,
        targetProtein:  targetProtein  ?? this.targetProtein,
        targetFat:      targetFat      ?? this.targetFat,
        targetCarbs:    targetCarbs    ?? this.targetCarbs,
      );

  @override
  bool operator ==(Object other) =>
      other is NutritionSettings &&
      other.targetCalories == targetCalories &&
      other.targetProtein  == targetProtein &&
      other.targetFat      == targetFat &&
      other.targetCarbs    == targetCarbs;

  @override
  int get hashCode => Object.hash(
        targetCalories, targetProtein, targetFat, targetCarbs);
}

// InheritedWidget обёртка
class NutritionSettingsInherited extends InheritedWidget {
  const NutritionSettingsInherited({
    super.key,
    required this.settings,
    required super.child,
  });

  final NutritionSettings settings;

  static NutritionSettings of(BuildContext context) {
    final inherited = context
        .dependOnInheritedWidgetOfExactType<NutritionSettingsInherited>();
    assert(
      inherited != null,
      'NutritionSettingsInherited не найден в дереве виджетов. '
      'Убедись, что он находится выше по дереву.',
    );
    return inherited!.settings;
  }

  /// Читает без регистрации зависимости (для initState)
  static NutritionSettings? maybeOf(BuildContext context) {
    return context
        .getInheritedWidgetOfExactType<NutritionSettingsInherited>()
        ?.settings;
  }

  @override
  bool updateShouldNotify(NutritionSettingsInherited oldWidget) =>
      settings != oldWidget.settings;
}
```

```dart
// Пример использования в виджете:
class DailyGoalWidget extends StatelessWidget {
  const DailyGoalWidget({super.key});

  @override
  Widget build(BuildContext context) {
    // Этот виджет перестроится при изменении настроек
    final settings = NutritionSettingsInherited.of(context);

    return Text('Цель: ${settings.targetCalories} ккал/день');
  }
}
```

</details>
