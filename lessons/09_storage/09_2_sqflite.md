# 09.2 sqflite — SQLite база данных

## 1. Суть

`sqflite` — Flutter-обёртка над SQLite. Реляционная база данных, встроенная прямо в приложение. Подходит для структурированных данных со связями: заметки, задачи, каталоги, история.

```yaml
# pubspec.yaml
dependencies:
  sqflite: ^2.3.2
  path: ^1.9.0 # для построения пути к файлу БД
```

---

## 2. Как используется во Flutter

Создаётся одна база данных (синглтон), таблицы создаются при первом запуске. Работа через `DatabaseHelper` — один класс, который управляет схемой и предоставляет CRUD-операции.

---

## 3. Основной синтаксис

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

// Открытие / создание БД
final database = await openDatabase(
  join(await getDatabasesPath(), 'app.db'),
  version: 1,
  onCreate: (db, version) async {
    await db.execute('''
      CREATE TABLE tasks (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        is_done INTEGER NOT NULL DEFAULT 0,
        created_at TEXT NOT NULL
      )
    ''');
  },
);

// INSERT
final id = await database.insert(
  'tasks',
  {'title': 'Купить молоко', 'is_done': 0, 'created_at': DateTime.now().toIso8601String()},
  conflictAlgorithm: ConflictAlgorithm.replace,
);

// SELECT все
final List<Map<String, dynamic>> rows = await database.query('tasks');

// SELECT с условием
final rows = await database.query(
  'tasks',
  where: 'is_done = ?',
  whereArgs: [0],
  orderBy: 'created_at DESC',
);

// SELECT через rawQuery
final rows = await database.rawQuery(
  'SELECT * FROM tasks WHERE title LIKE ?',
  ['%молоко%'],
);

// UPDATE
await database.update(
  'tasks',
  {'is_done': 1},
  where: 'id = ?',
  whereArgs: [id],
);

// DELETE
await database.delete(
  'tasks',
  where: 'id = ?',
  whereArgs: [id],
);

// Транзакция
await database.transaction((txn) async {
  await txn.insert('tasks', task1.toMap());
  await txn.insert('tasks', task2.toMap());
});
```

---

## 4. Реальный пример — репозиторий задач

```dart
import 'package:sqflite/sqflite.dart';
import 'package:path/path.dart';

class Task {
  final int? id;
  final String title;
  final bool isDone;
  final DateTime createdAt;

  const Task({
    this.id,
    required this.title,
    this.isDone = false,
    required this.createdAt,
  });

  Map<String, dynamic> toMap() => {
        if (id != null) 'id': id,
        'title': title,
        'is_done': isDone ? 1 : 0,
        'created_at': createdAt.toIso8601String(),
      };

  factory Task.fromMap(Map<String, dynamic> map) => Task(
        id: map['id'] as int?,
        title: map['title'] as String,
        isDone: (map['is_done'] as int) == 1,
        createdAt: DateTime.parse(map['created_at'] as String),
      );

  Task copyWith({bool? isDone}) => Task(
        id: id,
        title: title,
        isDone: isDone ?? this.isDone,
        createdAt: createdAt,
      );
}

// --- Database helper (синглтон) ---
class AppDatabase {
  static Database? _db;

  static Future<Database> get instance async {
    _db ??= await _initDb();
    return _db!;
  }

  static Future<Database> _initDb() => openDatabase(
        join(await getDatabasesPath(), 'tasks.db'),
        version: 1,
        onCreate: _onCreate,
        onUpgrade: _onUpgrade,
      );

  static Future<void> _onCreate(Database db, int version) async {
    await db.execute('''
      CREATE TABLE tasks (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        title TEXT NOT NULL,
        is_done INTEGER NOT NULL DEFAULT 0,
        created_at TEXT NOT NULL
      )
    ''');
  }

  static Future<void> _onUpgrade(Database db, int oldVersion, int newVersion) async {
    if (oldVersion < 2) {
      // Миграция на версию 2: добавляем поле priority
      await db.execute('ALTER TABLE tasks ADD COLUMN priority INTEGER DEFAULT 0');
    }
  }
}

// --- Repository ---
class TaskRepository {
  Future<List<Task>> getAllTasks() async {
    final db = await AppDatabase.instance;
    final rows = await db.query('tasks', orderBy: 'created_at DESC');
    return rows.map(Task.fromMap).toList();
  }

  Future<Task> addTask(String title) async {
    final db = await AppDatabase.instance;
    final task = Task(title: title, createdAt: DateTime.now());
    final id = await db.insert('tasks', task.toMap());
    return task.copyWith(id: id);
  }

  Future<void> toggleTask(int id, bool isDone) async {
    final db = await AppDatabase.instance;
    await db.update(
      'tasks',
      {'is_done': isDone ? 1 : 0},
      where: 'id = ?',
      whereArgs: [id],
    );
  }

  Future<void> deleteTask(int id) async {
    final db = await AppDatabase.instance;
    await db.delete('tasks', where: 'id = ?', whereArgs: [id]);
  }

  Future<List<Task>> searchTasks(String query) async {
    final db = await AppDatabase.instance;
    final rows = await db.rawQuery(
      'SELECT * FROM tasks WHERE title LIKE ? ORDER BY created_at DESC',
      ['%$query%'],
    );
    return rows.map(Task.fromMap).toList();
  }
}
```

---

## 5. Под капотом

- SQLite — встроенный движок, файл БД хранится в `getDatabasesPath()` (песочница приложения).
- `sqflite` выполняет все запросы в отдельном потоке — не блокирует UI.
- `version` в `openDatabase` — для управления миграциями через `onUpgrade`.
- `ConflictAlgorithm.replace` = `INSERT OR REPLACE` в SQLite.

---

## 6. Типичные ошибки

| Ошибка                               | Причина                               | Решение                                 |
| ------------------------------------ | ------------------------------------- | --------------------------------------- |
| `DatabaseException: table not found` | Схема не создана                      | Проверь `onCreate`, увеличь `version`   |
| Данные пропали                       | Пересоздал БД с тем же `version`      | Меняй `version` при изменении схемы     |
| Нет `NULL` ограничения               | SQLite не бросает ошибку по умолчанию | Явно указывай `NOT NULL` в DDL          |
| Медленные запросы                    | Нет индексов                          | `CREATE INDEX` для полей WHERE/ORDER BY |
| Лишние `int` для `bool`              | SQLite нет типа BOOL                  | Используй `1/0` и конвертируй в Dart    |

---

## 7. Рекомендации

1. **Синглтон для базы** — один экземпляр `Database` на всё приложение.
2. **Никогда не хардкодь SQL в виджетах** — только в Repository/DAO.
3. **`version` → `onUpgrade`** — всегда предусматривай миграции.
4. **Параметризованные запросы** (`?`, `whereArgs`) — защита от SQL Injection.
5. **Для сложных проектов** используй `Drift` — типобезопасный SQL с генерацией кода.
6. **Добавляй индексы** на поля, по которым делаешь `WHERE` и `ORDER BY`.
