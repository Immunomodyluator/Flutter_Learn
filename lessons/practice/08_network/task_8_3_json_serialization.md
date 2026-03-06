# 8.3: JSON сериализация — модели рецептов с json_serializable

> Project: FitMenu | Глава 8 — Networking

### 8.3: Автогенерация JSON-парсеров через json_serializable

🎯 **Цель шага:** Заменить ручной `fromJson`/`toJson` на автогенерированный код через `json_serializable` — создать типобезопасные DTO-модели для API-ответов.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  json_annotation: ^4.8.0

dev_dependencies:
  json_serializable: ^6.7.0
  build_runner: ^2.4.0
```

**RecipeDto:**
```dart
@JsonSerializable(explicitToJson: true)
class RecipeDto {
  const RecipeDto({required this.id, required this.title, required this.image, required this.readyInMinutes, this.nutrition});

  final int id;
  final String title;
  final String image;

  @JsonKey(name: 'readyInMinutes')
  final int readyInMinutes;

  final NutritionDto? nutrition;

  factory RecipeDto.fromJson(Map<String, dynamic> json) => _$RecipeDtoFromJson(json);
  Map<String, dynamic> toJson() => _$RecipeDtoToJson(this);
}
```

Аналогично: `NutritionDto` (поле `nutrients: List<NutrientDto>`) и `NutrientDto` (поля `name`, `amount`, `unit`).

**Генерация кода:**
```bash
dart run build_runner build --delete-conflicting-outputs
```

**Маппер DTO → Domain:**
```dart
extension RecipeDtoMapper on RecipeDto {
  Recipe toDomain() => Recipe(
    id: id.toString(),
    title: title,
    imageUrl: image,
    calories: nutrition?.caloriesAmount?.toInt() ?? 0,
    cookingTimeMinutes: readyInMinutes,
  );
}
```

---

✅ **Критерии приёмки:**
- [ ] `@JsonSerializable()` на всех DTO-классах
- [ ] `part 'recipe_dto.g.dart';` объявлен
- [ ] `@JsonKey(name: ...)` для snake_case/camelCase маппинга
- [ ] `build_runner build` генерирует файлы без ошибок
- [ ] DTO-классы иммутабельны (`final` + `const` конструктор)
- [ ] `explicitToJson: true` для вложенных объектов

---

💡 **Подсказка:** `@JsonKey(defaultValue: 0)` задаёт дефолт при отсутствии поля в JSON. `@JsonSerializable(explicitToJson: true)` обязателен при вложенных объектах. Файлы `*.g.dart` не добавляй в git вручную.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/data/models/recipe_dto.dart

import 'package:json_annotation/json_annotation.dart';

part 'recipe_dto.g.dart';

@JsonSerializable(explicitToJson: true)
class RecipeDto {
  const RecipeDto({required this.id, required this.title, required this.image, required this.readyInMinutes, this.nutrition});

  final int id;
  final String title;
  final String image;

  @JsonKey(name: 'readyInMinutes')
  final int readyInMinutes;

  final NutritionDto? nutrition;

  factory RecipeDto.fromJson(Map<String, dynamic> json) => _$RecipeDtoFromJson(json);
  Map<String, dynamic> toJson() => _$RecipeDtoToJson(this);
}

@JsonSerializable(explicitToJson: true)
class NutritionDto {
  const NutritionDto({required this.nutrients});

  final List<NutrientDto> nutrients;

  double? get caloriesAmount => _amount('Calories');
  double? get proteinAmount  => _amount('Protein');
  double? get fatAmount      => _amount('Fat');
  double? get carbsAmount    => _amount('Carbohydrates');

  double? _amount(String name) =>
      nutrients.where((n) => n.name == name).firstOrNull?.amount;

  factory NutritionDto.fromJson(Map<String, dynamic> json) => _$NutritionDtoFromJson(json);
  Map<String, dynamic> toJson() => _$NutritionDtoToJson(this);
}

@JsonSerializable()
class NutrientDto {
  const NutrientDto({required this.name, required this.amount, required this.unit});

  final String name;
  final double amount;
  final String unit;

  factory NutrientDto.fromJson(Map<String, dynamic> json) => _$NutrientDtoFromJson(json);
  Map<String, dynamic> toJson() => _$NutrientDtoToJson(this);
}
```

```dart
// lib/features/recipes/data/mappers/recipe_mapper.dart

extension RecipeDtoMapper on RecipeDto {
  Recipe toDomain() => Recipe(
        id: id.toString(),
        title: title,
        imageUrl: image,
        calories: nutrition?.caloriesAmount?.toInt() ?? 0,
        cookingTimeMinutes: readyInMinutes,
      );
}

extension RecipeListMapper on List<RecipeDto> {
  List<Recipe> toDomain() => map((d) => d.toDomain()).toList();
}
```

</details>
