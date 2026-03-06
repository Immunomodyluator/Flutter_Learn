# 2.1: Структура Flutter-проекта

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 2 — Настройка окружения Flutter

---

### 2.1: Структура Flutter-проекта

🎯 **Цель шага:** Создать Flutter-проект FitMenu с правильной feature-first структурой папок и настроенным `pubspec.yaml` — фундамент, на котором будет расти всё приложение.

📝 **Техническое задание:**

1. Создай проект: `flutter create --org com.fitmenufood --platforms=android,ios fitfood`
2. Построй в `lib/` следующую структуру (создай все директории, в пустых оставь `.gitkeep`):
   ```
   lib/
     core/
       domain/models/    ← Recipe, Ingredient, NutritionFact из главы 1
       utils/            ← nutrition_utils.dart
       theme/            ← app_theme.dart
     features/
       recipes/
         screens/        ← recipe_list_screen.dart
         widgets/        ← recipe_card.dart
       meal_log/
         screens/        ← meal_log_screen.dart
         widgets/        ← meal_entry_tile.dart
     main.dart
   ```
3. Настрой `pubspec.yaml`: добавь `assets: [assets/images/, assets/icons/]` и `fonts: [{family: Nunito, ...}]`
4. Минимальный `main.dart`: `MaterialApp` с `HomeScreen`, без лишнего кода из шаблона

✅ **Критерии приёмки:**
- [ ] Проект создан с правильным `--org` (не `com.example`)
- [ ] `lib/` содержит только `main.dart` на верхнем уровне
- [ ] `flutter pub get` завершается без ошибок
- [ ] `flutter analyze` выдаёт 0 issues на стартовом коде
- [ ] В `pubspec.yaml` секции `assets` и `fonts` оформлены синтаксически корректно

💡 **Подсказка:** Файл `pubspec.lock` **обязательно коммитить** в git — он обеспечивает воспроизводимые сборки в команде и CI. `.dart_tool/` и `build/` — в `.gitignore`.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```yaml
# pubspec.yaml
name: fitfood
description: FitMenu — трекер питания и рецептов (Project Orchid)
publish_to: 'none'
version: 1.0.0+1

environment:
  sdk: '>=3.3.0 <4.0.0'

dependencies:
  flutter:
    sdk: flutter
  google_fonts: ^6.2.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^4.0.0

flutter:
  uses-material-design: true
  assets:
    - assets/images/
    - assets/icons/
  fonts:
    - family: Nunito
      fonts:
        - asset: assets/fonts/Nunito-Regular.ttf
        - asset: assets/fonts/Nunito-Bold.ttf
          weight: 700
        - asset: assets/fonts/Nunito-SemiBold.ttf
          weight: 600
```

```dart
// lib/main.dart
import 'package:flutter/material.dart';
import 'core/theme/app_theme.dart';
import 'features/recipes/screens/recipe_list_screen.dart';

void main() => runApp(const FitMenuApp());

class FitMenuApp extends StatelessWidget {
  const FitMenuApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'FitMenu',
      debugShowCheckedModeBanner: false,
      theme: AppTheme.light,
      home: const RecipeListScreen(),
    );
  }
}

// lib/core/theme/app_theme.dart
class AppTheme {
  static const Color primaryGreen  = Color(0xFF4CAF50);
  static const Color accentOrange  = Color(0xFFFF7043);

  static ThemeData get light => ThemeData(
    colorScheme: ColorScheme.fromSeed(seedColor: primaryGreen),
    useMaterial3: true,
    fontFamily: 'Nunito',
  );

  static ThemeData get dark => ThemeData(
    colorScheme: ColorScheme.fromSeed(
      seedColor: primaryGreen, brightness: Brightness.dark),
    useMaterial3: true,
    fontFamily: 'Nunito',
  );
}
```

```
# .gitignore (добавить)
.dart_tool/
build/
*.g.dart          # кодогенерация — не коммитим
*.freezed.dart    # кодогенерация
pubspec.lock      # НЕ добавлять сюда! lock ДОЛЖЕН быть в git
```

</details>
