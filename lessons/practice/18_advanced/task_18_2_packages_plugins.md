# 18.2: Создание Flutter пакета

> Project: FitMenu | Глава 18 — Advanced

### 18.2: Публикация reusable пакета nutrition_widgets

🎯 **Цель шага:** Вынести виджеты `CalorieRingChart` и `NutritionStatsBar` из FitMenu в отдельный Flutter-пакет `nutrition_widgets`, опубликовать его на pub.dev (или использовать локально через `path:`).

---

📝 **Техническое задание:**

**1. Создать пакет:**
```bash
flutter create --template=package nutrition_widgets
cd nutrition_widgets
```

**Структура пакета:**
```
nutrition_widgets/
  lib/
    nutrition_widgets.dart        # barrel export
    src/
      calorie_ring_chart.dart
      nutrition_stats_bar.dart
      nutrition_ring_data.dart
  example/
    lib/main.dart                  # демо-приложение
  test/
    nutrition_widgets_test.dart
  pubspec.yaml
  README.md
  CHANGELOG.md
  LICENSE
```

**2. `pubspec.yaml` пакета:**
```yaml
name: nutrition_widgets
description: Reusable Flutter widgets for nutrition tracking apps
version: 0.1.0
homepage: https://github.com/yourusername/nutrition_widgets

environment:
  sdk: '>=3.0.0 <4.0.0'
  flutter: '>=3.16.0'

dependencies:
  flutter:
    sdk: flutter

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
```

**3. Barrel export `lib/nutrition_widgets.dart`:**
```dart
library nutrition_widgets;

export 'src/models/nutrition_ring_data.dart';
export 'src/widgets/calorie_ring_chart.dart';
export 'src/widgets/nutrition_stats_bar.dart';
```

**4. Использование из FitMenu:**
```yaml
# fitmenu/pubspec.yaml
dependencies:
  nutrition_widgets:
    path: ../nutrition_widgets   # локально
    # или:
    # git:
    #   url: https://github.com/yourusername/nutrition_widgets
    # или после публикации:
    # nutrition_widgets: ^0.1.0
```

**5. Публикация на pub.dev:**
```bash
flutter pub publish --dry-run   # проверить без публикации
flutter pub publish             # публикация
```

---

✅ **Критерии приёмки:**
- [ ] `flutter pub publish --dry-run` выходит с кодом 0
- [ ] Пакет имеет `README.md` с примером использования
- [ ] `CHANGELOG.md` заполнен
- [ ] `LICENSE` файл добавлен (MIT или Apache 2.0)
- [ ] `example/` содержит рабочее демо-приложение
- [ ] `flutter test` в директории пакета проходит

---

💡 **Подсказка:** `dart pub score` в терминале покажет оценку по критериям pub.dev (документация, тесты, поддержка платформ). `/// ` (три слэша) — dartdoc комментарии, они появляются на pub.dev. Минимальный score для "Verified Publisher" — 100/130.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// nutrition_widgets/lib/src/models/nutrition_ring_data.dart

/// Данные питания для отображения в [CalorieRingChart].
class NutritionRingData {
  /// Создаёт данные для кольцевой диаграммы питания.
  const NutritionRingData({
    required this.calories,
    required this.targetCalories,
    required this.protein,
    required this.targetProtein,
    required this.fat,
    required this.targetFat,
    required this.carbs,
    required this.targetCarbs,
  });

  /// Потреблённые калории (ккал).
  final double calories;

  /// Целевые калории (ккал).
  final double targetCalories;

  /// Потреблённые белки (г).
  final double protein;

  /// Целевые белки (г).
  final double targetProtein;

  /// Потреблённые жиры (г).
  final double fat;

  /// Целевые жиры (г).
  final double targetFat;

  /// Потреблённые углеводы (г).
  final double carbs;

  /// Целевые углеводы (г).
  final double targetCarbs;

  @override
  bool operator ==(Object other) =>
      identical(this, other) ||
      other is NutritionRingData &&
          calories == other.calories &&
          targetCalories == other.targetCalories &&
          protein == other.protein;

  @override
  int get hashCode => Object.hash(calories, targetCalories, protein, fat, carbs);
}
```

```dart
// nutrition_widgets/lib/nutrition_widgets.dart

/// Flutter widgets для приложений по трекингу питания.
///
/// Основные виджеты:
/// - [CalorieRingChart] — кольцевая диаграмма КБЖУ
/// - [NutritionStatsBar] — горизонтальная полоска прогресса
library nutrition_widgets;

export 'src/models/nutrition_ring_data.dart';
export 'src/widgets/calorie_ring_chart.dart';
export 'src/widgets/nutrition_stats_bar.dart';
```

```markdown
<!-- nutrition_widgets/README.md -->

# nutrition_widgets

Flutter widgets для приложений трекинга питания. Включает кольцевую диаграмму КБЖУ и полоску прогресса.

## Установка

```yaml
dependencies:
  nutrition_widgets: ^0.1.0
```

## Использование

```dart
import 'package:nutrition_widgets/nutrition_widgets.dart';

CalorieRingChart(
  data: NutritionRingData(
    calories: 1240, targetCalories: 2000,
    protein: 85,    targetProtein: 120,
    fat: 42,        targetFat: 65,
    carbs: 165,     targetCarbs: 250,
  ),
  size: 200,
)
```
```

</details>
