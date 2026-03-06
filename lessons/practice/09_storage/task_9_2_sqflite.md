# 9.2: sqflite — дневник питания в SQLite

> Project: FitMenu | Глава 9 — Storage

### 9.2: MealLogDatabase с sqflite для журнала приёмов пищи

🎯 **Цель шага:** Создать локальную SQLite БД через `sqflite` для хранения дневника питания — CRUD операции для записей о еде с датой, КБЖУ и типом приёма пищи.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  sqflite: ^2.3.0
  path: ^1.9.0
```

Реализуй `lib/core/database/meal_log_database.dart`.

**Схема таблицы `meal_logs`:**
```sql
CREATE TABLE meal_logs (
  id        TEXT PRIMARY KEY,
  recipe_id TEXT NOT NULL,
  title     TEXT NOT NULL,
  calories  REAL NOT NULL,
  protein   REAL NOT NULL,
  fat       REAL NOT NULL,
  carbs     REAL NOT NULL,
  meal_type TEXT NOT NULL,  -- 'breakfast' | 'lunch' | 'dinner'
  eaten_at  TEXT NOT NULL   -- ISO 8601: '2024-01-15T12:30:00'
)
```

**MealLogDatabase:**
```dart
class MealLogDatabase {
  static const _dbName    = 'fitmenu.db';
  static const _dbVersion = 1;
  static const _table     = 'meal_logs';

  Database? _db;

  Future<Database> get database async { ... } // lazy init + singleton

  Future<void> insert(MealLogEntry entry) async { ... }
  Future<List<MealLogEntry>> getByDate(DateTime date) async { ... }
  Future<void> delete(String id) async { ... }
  Future<double> totalCaloriesForDate(DateTime date) async { ... }
}
```

**getByDate:** запрос с `WHERE eaten_at LIKE '2024-01-15%'` (сравнение по дате без времени).

**MealLogEntry:**
```dart
class MealLogEntry {
  // Все поля + fromMap() + toMap() методы
}
```

---

✅ **Критерии приёмки:**
- [ ] БД инициализируется лениво (первый вызов `database` создаёт файл)
- [ ] `onCreate` создаёт таблицу `meal_logs`
- [ ] `insert` использует `ConflictAlgorithm.replace`
- [ ] `getByDate` фильтрует по дате через `LIKE '${dateStr}%'`
- [ ] `totalCaloriesForDate` использует SQL `SUM(calories)`
- [ ] `MealLogEntry.toMap()` корректно сериализует все поля

---

💡 **Подсказка:** `openDatabase(path, version: 1, onCreate: ...)` — стандартный способ инициализации. `path_provider` + `join(docsPath, 'fitmenu.db')` — правильный путь к БД на устройстве. `rawQuery` для агрегатных функций: `SELECT SUM(calories) FROM meal_logs WHERE ...`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/database/meal_log_database.dart

import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class MealLogEntry {
  const MealLogEntry({
    required this.id,
    required this.recipeId,
    required this.title,
    required this.calories,
    required this.protein,
    required this.fat,
    required this.carbs,
    required this.mealType,
    required this.eatenAt,
  });

  final String id;
  final String recipeId;
  final String title;
  final double calories;
  final double protein;
  final double fat;
  final double carbs;
  final String mealType;
  final DateTime eatenAt;

  Map<String, dynamic> toMap() => {
        'id':        id,
        'recipe_id': recipeId,
        'title':     title,
        'calories':  calories,
        'protein':   protein,
        'fat':       fat,
        'carbs':     carbs,
        'meal_type': mealType,
        'eaten_at':  eatenAt.toIso8601String(),
      };

  factory MealLogEntry.fromMap(Map<String, dynamic> m) => MealLogEntry(
        id:       m['id'] as String,
        recipeId: m['recipe_id'] as String,
        title:    m['title'] as String,
        calories: (m['calories'] as num).toDouble(),
        protein:  (m['protein'] as num).toDouble(),
        fat:      (m['fat'] as num).toDouble(),
        carbs:    (m['carbs'] as num).toDouble(),
        mealType: m['meal_type'] as String,
        eatenAt:  DateTime.parse(m['eaten_at'] as String),
      );
}

class MealLogDatabase {
  static const _dbName    = 'fitmenu.db';
  static const _dbVersion = 1;
  static const _table     = 'meal_logs';

  Database? _db;

  Future<Database> get database async {
    if (_db != null) return _db!;
    final path = join(await getDatabasesPath(), _dbName);
    _db = await openDatabase(
      path,
      version: _dbVersion,
      onCreate: _onCreate,
    );
    return _db!;
  }

  Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE $_table (
        id        TEXT PRIMARY KEY,
        recipe_id TEXT NOT NULL,
        title     TEXT NOT NULL,
        calories  REAL NOT NULL,
        protein   REAL NOT NULL,
        fat       REAL NOT NULL,
        carbs     REAL NOT NULL,
        meal_type TEXT NOT NULL,
        eaten_at  TEXT NOT NULL
      )
    ''');
  }

  Future<void> insert(MealLogEntry entry) async {
    final db = await database;
    await db.insert(_table, entry.toMap(),
        conflictAlgorithm: ConflictAlgorithm.replace);
  }

  Future<List<MealLogEntry>> getByDate(DateTime date) async {
    final db = await database;
    final dateStr = date.toIso8601String().substring(0, 10); // 'YYYY-MM-DD'
    final rows = await db.query(
      _table,
      where: 'eaten_at LIKE ?',
      whereArgs: ['$dateStr%'],
      orderBy: 'eaten_at ASC',
    );
    return rows.map(MealLogEntry.fromMap).toList();
  }

  Future<void> delete(String id) async {
    final db = await database;
    await db.delete(_table, where: 'id = ?', whereArgs: [id]);
  }

  Future<double> totalCaloriesForDate(DateTime date) async {
    final db = await database;
    final dateStr = date.toIso8601String().substring(0, 10);
    final result = await db.rawQuery(
      'SELECT SUM(calories) as total FROM $_table WHERE eaten_at LIKE ?',
      ['$dateStr%'],
    );
    return (result.first['total'] as num?)?.toDouble() ?? 0.0;
  }

  Future<void> close() async => (await database).close();
}
```

</details>
