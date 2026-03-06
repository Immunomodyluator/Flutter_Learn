# 09.3 Drift — типобезопасный SQLite ORM

## 1. Суть

`Drift` (ранее `moor`) — ORM поверх SQLite для Flutter/Dart. Генерирует типобезопасный Dart-код из описаний таблиц. Поддерживает реактивные запросы через `Stream`, миграции, транзакции, JOIN.

```yaml
# pubspec.yaml
dependencies:
  drift: ^2.14.1
  sqlite3_flutter_libs: ^0.5.0 # нативный SQLite
  path_provider: ^2.1.2
  path: ^1.9.0

dev_dependencies:
  drift_dev: ^2.14.1
  build_runner: ^2.4.8
```

---

## 2. Как используется во Flutter

Описываешь таблицы как Dart-классы → `build_runner` генерирует весь шаблонный код. Запросы пишешь на Dart — компилятор проверяет их корректность. Watch-запросы — реактивны (Stream), UI обновляется автоматически при изменении данных.

---

## 3. Описание таблиц и базы данных

```dart
import 'package:drift/drift.dart';
import 'package:drift/native.dart';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;
import 'dart:io';

part 'app_database.g.dart'; // сгенерированный файл

// --- Описание таблицы ---
class Tasks extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get title => text().withLength(min: 1, max: 200)();
  BoolColumn get isDone => boolean().withDefault(const Constant(false))();
  DateTimeColumn get createdAt => dateTime().withDefault(currentDateAndTime)();
  IntColumn get priority => integer().withDefault(const Constant(0))();
}

class Categories extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text()();
  TextColumn get color => text().withDefault(const Constant('#607D8B'))();
}

// --- Описание базы данных ---
@DriftDatabase(tables: [Tasks, Categories])
class AppDatabase extends _$AppDatabase {
  AppDatabase() : super(_openConnection());

  @override
  int get schemaVersion => 1;

  @override
  MigrationStrategy get migration => MigrationStrategy(
    onCreate: (Migrator m) async {
      await m.createAll();
    },
    onUpgrade: (Migrator m, int from, int to) async {
      if (from < 2) {
        await m.addColumn(tasks, tasks.priority);
      }
    },
  );
}

// Соединение с файлом БД
LazyDatabase _openConnection() => LazyDatabase(() async {
      final dbFolder = await getApplicationDocumentsDirectory();
      final file = File(p.join(dbFolder.path, 'app.db'));
      return NativeDatabase.createInBackground(file);
    });
```

```bash
# Генерация кода
dart run build_runner build --delete-conflicting-outputs
```

---

## 4. CRUD операции

```dart
// Доступ к DAO через базу данных
final db = AppDatabase();

// SELECT все (Future)
final allTasks = await db.select(db.tasks).get();

// SELECT с условием
final pending = await (db.select(db.tasks)
      ..where((t) => t.isDone.equals(false))
      ..orderBy([(t) => OrderingTerm.desc(t.createdAt)]))
    .get();

// Реактивный запрос (Stream — обновляется при изменении данных)
Stream<List<Task>> watchPendingTasks() =>
    (db.select(db.tasks)
          ..where((t) => t.isDone.equals(false))
          ..orderBy([(t) => OrderingTerm.desc(t.createdAt)]))
        .watch();

// INSERT
final id = await db.into(db.tasks).insert(
  TasksCompanion.insert(
    title: 'Новая задача',
    isDone: const Value(false),
  ),
);

// UPDATE по объекту
await db.update(db.tasks).replace(
  task.copyWith(isDone: true),
);

// UPDATE с условием
await (db.update(db.tasks)..where((t) => t.id.equals(id)))
    .write(const TasksCompanion(isDone: Value(true)));

// DELETE
await (db.delete(db.tasks)..where((t) => t.id.equals(id))).go();

// Транзакция
await db.transaction(() async {
  final categoryId = await db.into(db.categories).insert(
    CategoriesCompanion.insert(name: 'Работа'),
  );
  await db.into(db.tasks).insert(
    TasksCompanion.insert(title: 'Важная задача'),
  );
});
```

---

## 5. Реальный пример — DAO с watch-потоком

```dart
// Выносим запросы в DAO (Data Access Object)
part of 'app_database.dart';

extension TaskDao on AppDatabase {
  // Реактивный поток всех задач
  Stream<List<Task>> watchAllTasks() =>
      (select(tasks)..orderBy([(t) => OrderingTerm.desc(t.createdAt)])).watch();

  // Поиск
  Future<List<Task>> searchTasks(String query) =>
      (select(tasks)..where((t) => t.title.contains(query))).get();

  // CRUD
  Future<int> createTask(String title) => into(tasks).insert(
    TasksCompanion.insert(title: title),
  );

  Future<void> toggleTask(Task task) => update(tasks).replace(
    task.copyWith(isDone: !task.isDone),
  );

  Future<int> deleteTask(int id) =>
      (delete(tasks)..where((t) => t.id.equals(id))).go();
}

// --- Verwendung im Widget ---
class TaskListScreen extends StatelessWidget {
  final AppDatabase db;

  const TaskListScreen({super.key, required this.db});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<List<Task>>(
      stream: db.watchAllTasks(),
      builder: (context, snapshot) {
        if (!snapshot.hasData) return const CircularProgressIndicator();
        final tasks = snapshot.data!;
        return ListView.builder(
          itemCount: tasks.length,
          itemBuilder: (context, index) {
            final task = tasks[index];
            return CheckboxListTile(
              title: Text(task.title),
              value: task.isDone,
              onChanged: (_) => db.toggleTask(task),
            );
          },
        );
      },
    );
  }
}
```

---

## 6. Под капотом

- `@DriftDatabase` → `build_runner` генерирует `_$AppDatabase` в `.g.dart`.
- `Companion` классы — типобезопасная вставка/обновление без `Map<String, dynamic>`.
- `.watch()` → `Stream` через Dart `StreamController`, пересчитывается при любом `INSERT/UPDATE/DELETE` в таблице.
- Запросы выполняются в фоновом `Isolate` через `createInBackground`.

---

## 7. Типичные ошибки

| Ошибка                      | Причина                               | Решение                                  |
| --------------------------- | ------------------------------------- | ---------------------------------------- |
| `part ... not found`        | Не запущен `build_runner`             | `dart run build_runner build`            |
| `schemaVersion` не обновлён | Изменил таблицу без версии            | Увеличь `schemaVersion`, добавь миграцию |
| Stream не обновляется       | Используется `get()` вместо `watch()` | Замени на `.watch()`                     |
| `Value` vs `Value.absent()` | Companion требует явного указания     | `Value.absent()` = не изменять поле      |
| Медленный DEBUG сборка      | Много кодогенерации                   | Нормально, RELEASE значительно быстрее   |

---

## 8. Рекомендации

1. **DAO через `extension`** — выноси все запросы из виджетов в расширения базы данных.
2. **`watch()` вместо `get()`** — реактивные списки при работе с UI.
3. **`schemaVersion` + `onUpgrade`** — плановые миграции с первого дня.
4. **`createInBackground`** — не блокирует main isolate при тяжёлых запросах.
5. **Drift вместо sqflite** если нужны: Stream-запросы, типобезопасность, сложные JOIN.
