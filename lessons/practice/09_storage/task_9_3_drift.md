# 9.3: Drift — типобезопасная ORM для дневника питания

> Project: FitMenu | Глава 9 — Storage

### 9.3: MealLogDao с Drift (type-safe SQLite ORM)

🎯 **Цель шага:** Переработать хранилище дневника питания на `Drift` — получить типобезопасные запросы, автогенерированные DAO, реактивные стримы и автоматические миграции схемы.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  drift: ^2.14.0
  sqlite3_flutter_libs: ^0.5.0
  path_provider: ^2.1.0
  path: ^1.9.0

dev_dependencies:
  drift_dev: ^2.14.0
  build_runner: ^2.4.0
```

**Определение таблицы:**
```dart
// lib/core/database/tables/meal_logs_table.dart

class MealLogs extends Table {
  TextColumn get id        => text()();
  TextColumn get recipeId  => text().named('recipe_id')();
  TextColumn get title     => text()();
  RealColumn get calories  => real()();
  RealColumn get protein   => real()();
  RealColumn get fat       => real()();
  RealColumn get carbs     => real()();
  TextColumn get mealType  => text().named('meal_type')();
  DateTimeColumn get eatenAt => dateTime().named('eaten_at')();

  @override
  Set<Column> get primaryKey => {id};
}
```

**AppDatabase:**
```dart
@DriftDatabase(tables: [MealLogs])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  // DAO методы
  Future<List<MealLog>> getLogsForDate(DateTime date) => ...
  Stream<List<MealLog>> watchLogsForDate(DateTime date) => ...
  Future<void> insertLog(MealLogsCompanion entry) => ...
  Future<void> deleteLog(String id) => ...
}
```

**Реактивный запрос (Stream):**
```dart
Stream<List<MealLog>> watchLogsForDate(DateTime date) {
  return (select(mealLogs)
    ..where((t) => t.eatenAt.year.equals(date.year) &
                   t.eatenAt.month.equals(date.month) &
                   t.eatenAt.day.equals(date.day)))
    .watch();
}
```

**Генерация кода:**
```bash
dart run build_runner build --delete-conflicting-outputs
```

---

✅ **Критерии приёмки:**
- [ ] Таблица `MealLogs` определена через Drift `Table`
- [ ] `@DriftDatabase(tables: [...])` аннотация на `AppDatabase`
- [ ] `build_runner` генерирует `app_database.g.dart` без ошибок
- [ ] `watchLogsForDate` возвращает `Stream<List<MealLog>>` (реактивный)
- [ ] В UI использован `StreamBuilder` для отображения дневника
- [ ] `MealLogsCompanion` используется для вставки (не `MealLog` напрямую)

---

💡 **Подсказка:** Drift генерирует `DataClass` (например, `MealLog`) и `Companion` (для вставки/обновления). `StreamBuilder<List<MealLog>>` + `watchLogsForDate()` даёт реактивное обновление UI при изменении БД. Миграции описываются в `MigrationStrategy` внутри `AppDatabase`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/database/app_database.dart

import 'dart:io';
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

part 'app_database.g.dart';

// Определение таблицы
class MealLogs extends Table {
  TextColumn   get id       => text()();
  TextColumn   get recipeId => text().named('recipe_id')();
  TextColumn   get title    => text()();
  RealColumn   get calories => real()();
  RealColumn   get protein  => real()();
  RealColumn   get fat      => real()();
  RealColumn   get carbs    => real()();
  TextColumn   get mealType => text().named('meal_type')();
  DateTimeColumn get eatenAt => dateTime().named('eaten_at')();

  @override
  Set<Column> get primaryKey => {id};
}

@DriftDatabase(tables: [MealLogs])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());
  AppDatabase.forTesting(super.e); // для тестов

  @override
  int get schemaVersion => 1;

  // Запрос списка за дату (Future)
  Future<List<MealLog>> getLogsForDate(DateTime date) {
    return (select(mealLogs)
          ..where(
            (t) =>
                t.eatenAt.year.equals(date.year) &
                t.eatenAt.month.equals(date.month) &
                t.eatenAt.day.equals(date.day),
          )
          ..orderBy([(t) => OrderingTerm.asc(t.eatenAt)]))
        .get();
  }

  // Реактивный стрим
  Stream<List<MealLog>> watchLogsForDate(DateTime date) {
    return (select(mealLogs)
          ..where(
            (t) =>
                t.eatenAt.year.equals(date.year) &
                t.eatenAt.month.equals(date.month) &
                t.eatenAt.day.equals(date.day),
          ))
        .watch();
  }

  Future<void> insertLog(MealLogsCompanion entry) =>
      into(mealLogs).insertOnConflictUpdate(entry);

  Future<void> deleteLog(String id) =>
      (delete(mealLogs)..where((t) => t.id.equals(id))).go();

  Future<double> totalCaloriesForDate(DateTime date) async {
    final logs = await getLogsForDate(date);
    return logs.fold(0.0, (sum, l) => sum + l.calories);
  }
}

LazyDatabase _openConnection() {
  return LazyDatabase(() async {
    final dir  = await getApplicationDocumentsDirectory();
    final file = File(p.join(dir.path, 'fitmenu.db'));
    return NativeDatabase.createInBackground(file);
  });
}
```

```dart
// Использование с StreamBuilder в виджете:
StreamBuilder<List<MealLog>>(
  stream: database.watchLogsForDate(DateTime.now()),
  builder: (context, snapshot) {
    if (!snapshot.hasData) return const CircularProgressIndicator();
    final logs = snapshot.data!;
    return ListView.builder(
      itemCount: logs.length,
      itemBuilder: (_, i) => ListTile(
        title: Text(logs[i].title),
        subtitle: Text('${logs[i].calories} ккал'),
      ),
    );
  },
)
```

</details>
