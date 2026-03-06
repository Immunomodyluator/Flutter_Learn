# 9.1: SharedPreferences — хранение настроек пользователя

> Project: FitMenu | Глава 9 — Storage

### 9.1: UserPreferencesService с SharedPreferences

🎯 **Цель шага:** Реализовать сохранение пользовательских настроек FitMenu (цели КБЖУ, диетические ограничения, единицы измерения) в `SharedPreferences` — быстрое key-value хранилище для небольших данных.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  shared_preferences: ^2.2.0
```

Реализуй `lib/core/storage/user_preferences_service.dart`.

**UserPreferences (модель):**
```dart
class UserPreferences {
  const UserPreferences({
    this.targetCalories = 2000,
    this.targetProtein  = 150,
    this.targetFat      = 65,
    this.targetCarbs    = 250,
    this.isMetricUnits  = true,
    this.dietType       = DietType.balanced,
  });
  final int targetCalories;
  final int targetProtein;
  final int targetFat;
  final int targetCarbs;
  final bool isMetricUnits;
  final DietType dietType;
}

enum DietType { balanced, lowCarb, highProtein, vegetarian, vegan }
```

**UserPreferencesService:**
```dart
class UserPreferencesService {
  static const _keyCalories  = 'target_calories';
  static const _keyProtein   = 'target_protein';
  static const _keyFat       = 'target_fat';
  static const _keyCarbs     = 'target_carbs';
  static const _keyMetric    = 'is_metric';
  static const _keyDietType  = 'diet_type';

  Future<UserPreferences> load() async { ... }
  Future<void> save(UserPreferences prefs) async { ... }
  Future<void> clear() async { ... }
}
```

**Метод `save`:** записывает каждое поле по ключу.
**Метод `load`:** читает все поля, возвращает `UserPreferences.defaults` если данных нет.
**Метод `clear`:** удаляет все ключи.

---

✅ **Критерии приёмки:**
- [ ] Все поля `UserPreferences` сохраняются и восстанавливаются корректно
- [ ] `DietType` сохраняется как строка (`dietType.name`) и восстанавливается через `DietType.values.byName`
- [ ] При отсутствии данных возвращаются дефолтные значения
- [ ] Сервис не хранит состояние — только читает/пишет в `SharedPreferences`
- [ ] `SharedPreferences.getInstance()` вызывается при каждом запросе (или кешируется)

---

💡 **Подсказка:** `prefs.getInt(key) ?? defaultValue` — безопасное чтение с fallback. Для enum используй `.name` при записи и `Enum.values.byName(string)` при чтении. Кеши инстанс `SharedPreferences` через `late final` поле чтобы не вызывать `getInstance()` при каждой операции.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/storage/user_preferences_service.dart

import 'package:shared_preferences/shared_preferences.dart';

enum DietType { balanced, lowCarb, highProtein, vegetarian, vegan }

class UserPreferences {
  const UserPreferences({
    this.targetCalories = 2000,
    this.targetProtein  = 150,
    this.targetFat      = 65,
    this.targetCarbs    = 250,
    this.isMetricUnits  = true,
    this.dietType       = DietType.balanced,
  });

  final int targetCalories;
  final int targetProtein;
  final int targetFat;
  final int targetCarbs;
  final bool isMetricUnits;
  final DietType dietType;

  static const defaults = UserPreferences();

  UserPreferences copyWith({
    int? targetCalories, int? targetProtein, int? targetFat, int? targetCarbs,
    bool? isMetricUnits, DietType? dietType,
  }) =>
      UserPreferences(
        targetCalories: targetCalories ?? this.targetCalories,
        targetProtein:  targetProtein  ?? this.targetProtein,
        targetFat:      targetFat      ?? this.targetFat,
        targetCarbs:    targetCarbs    ?? this.targetCarbs,
        isMetricUnits:  isMetricUnits  ?? this.isMetricUnits,
        dietType:       dietType       ?? this.dietType,
      );
}

class UserPreferencesService {
  static const _keyCalories = 'target_calories';
  static const _keyProtein  = 'target_protein';
  static const _keyFat      = 'target_fat';
  static const _keyCarbs    = 'target_carbs';
  static const _keyMetric   = 'is_metric';
  static const _keyDiet     = 'diet_type';

  SharedPreferences? _prefs;

  Future<SharedPreferences> _getPrefs() async =>
      _prefs ??= await SharedPreferences.getInstance();

  Future<UserPreferences> load() async {
    final prefs = await _getPrefs();
    final dietStr = prefs.getString(_keyDiet);

    return UserPreferences(
      targetCalories: prefs.getInt(_keyCalories) ?? 2000,
      targetProtein:  prefs.getInt(_keyProtein)  ?? 150,
      targetFat:      prefs.getInt(_keyFat)      ?? 65,
      targetCarbs:    prefs.getInt(_keyCarbs)    ?? 250,
      isMetricUnits:  prefs.getBool(_keyMetric)  ?? true,
      dietType: dietStr != null
          ? DietType.values.byName(dietStr)
          : DietType.balanced,
    );
  }

  Future<void> save(UserPreferences p) async {
    final prefs = await _getPrefs();
    await Future.wait([
      prefs.setInt(_keyCalories, p.targetCalories),
      prefs.setInt(_keyProtein,  p.targetProtein),
      prefs.setInt(_keyFat,      p.targetFat),
      prefs.setInt(_keyCarbs,    p.targetCarbs),
      prefs.setBool(_keyMetric,  p.isMetricUnits),
      prefs.setString(_keyDiet,  p.dietType.name),
    ]);
  }

  Future<void> clear() async {
    final prefs = await _getPrefs();
    await Future.wait([
      prefs.remove(_keyCalories), prefs.remove(_keyProtein),
      prefs.remove(_keyFat),      prefs.remove(_keyCarbs),
      prefs.remove(_keyMetric),   prefs.remove(_keyDiet),
    ]);
  }
}
```

</details>
