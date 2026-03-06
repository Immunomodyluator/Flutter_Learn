# Квиз: SQLite / Drift

**Тема:** 09.2 — SQLite / Drift  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое `sqflite` и когда его использовать?

- A) Пакет для отправки SQL запросов на сервер
- B) Flutter пакет для работы с SQLite — реляционной базой данных на устройстве
- C) Графический инструмент для работы с БД
- D) Пакет для шифрования файлов

<details>
<summary>Ответ</summary>

**B) Flutter пакет для SQLite на устройстве**

SQLite для Flutter: `sqflite` (базовый API) или `drift` (type-safe ORM). Используй когда: нужны сложные запросы, связи между таблицами, транзакции, большие объёмы данных (тысячи записей).

</details>

---

### Вопрос 2 🟢

Как создать базу данных и таблицу с `sqflite`?

- A) `Sqflite.createDatabase(...)`
- B) `openDatabase(path, onCreate: (db, version) => db.execute('CREATE TABLE ...'))`
- C) `Database.create(path, tables: [...])`
- D) `SQLite.init(tables: [UserTable()])`

<details>
<summary>Ответ</summary>

**B) `openDatabase(path, onCreate: ...)`**

```dart
final db = await openDatabase(
  join(await getDatabasesPath(), 'app.db'),
  version: 1,
  onCreate: (db, version) async {
    await db.execute('''
      CREATE TABLE users(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name TEXT NOT NULL,
        email TEXT UNIQUE
      )
    ''');
  },
);
```

</details>

---

### Вопрос 3 🟢

Как вставить запись через `sqflite`?

- A) `db.add(table: 'users', data: user.toMap())`
- B) `db.insert('users', user.toMap())` → возвращает `id`
- C) `db.execute("INSERT INTO users VALUES (?)", [user.name])`
- D) `db.put('users', user)`

<details>
<summary>Ответ</summary>

**B) `db.insert('users', user.toMap())`**

```dart
final id = await db.insert(
  'users',
  {'name': 'Alice', 'email': 'alice@example.com'},
  conflictAlgorithm: ConflictAlgorithm.replace,
);
// ConflictAlgorithm: abort, fail, ignore, replace, rollback
```

</details>

---

### Вопрос 4 🟡

Что такое `Drift` (ранее Moor)?

- A) Форк SQLite для Flutter
- B) Type-safe ORM поверх SQLite с кодогенерацией — таблицы описываются Dart классами, queries — методами
- C) NoSQL база данных для Flutter
- D) Пакет для миграции баз данных

<details>
<summary>Ответ</summary>

**B) Type-safe ORM поверх SQLite с кодогенерацией**

```dart
class Users extends Table {
  IntColumn get id => integer().autoIncrement()();
  TextColumn get name => text().withLength(min: 1, max: 50)();
  TextColumn get email => text().nullable()();
}
```

Генерирует: тип `User`, `UsersCompanion`, методы `into(users).insert(...)`, type-safe where clauses.

</details>

---

### Вопрос 5 🟡

Как выполнить SELECT запрос с фильтрацией в `sqflite`?

- A) `db.select('users', filter: 'id = ?', args: [1])`
- B) `db.query('users', where: 'id = ?', whereArgs: [1])`
- C) `db.find<User>((u) => u.id == 1)`
- D) `db.execute("SELECT * FROM users WHERE id = 1")`

<details>
<summary>Ответ</summary>

**B) `db.query('users', where: '...', whereArgs: [...])`**

```dart
final result = await db.query(
  'users',
  where: 'name LIKE ? AND email IS NOT NULL',
  whereArgs: ['%Alice%'],
  orderBy: 'name ASC',
  limit: 10,
  offset: 0,
);
final users = result.map((m) => User.fromMap(m)).toList();
```

Всегда используй `whereArgs` для параметров — защита от SQL injection.

</details>

---

### Вопрос 6 🟡

Как реализовать миграцию базы данных при обновлении приложения?

- A) Удалить и создать базу заново
- B) `openDatabase(path, version: 2, onUpgrade: (db, oldV, newV) => { if (oldV < 2) db.execute('ALTER TABLE...') })`
- C) Автоматически — Flutter обновляет схему
- D) Через отдельный migration файл

<details>
<summary>Ответ</summary>

**B) `onUpgrade` callback с версионированием**

```dart
openDatabase(
  path,
  version: 3,
  onUpgrade: (db, oldVersion, newVersion) async {
    if (oldVersion < 2) {
      await db.execute('ALTER TABLE users ADD COLUMN phone TEXT');
    }
    if (oldVersion < 3) {
      await db.execute('CREATE TABLE tags(...)');
    }
  },
);
```

`Drift` имеет встроенную систему миграций: `MigrationStrategy(onUpgrade: ...)`.

</details>

---

### Вопрос 7 🟡

Как выполнить транзакцию в `sqflite`?

- A) `db.runTransaction(() async { ... })`
- B) `db.transaction((txn) async { await txn.insert(...); await txn.update(...); })` — автоматический rollback при исключении
- C) `db.beginTransaction(); ... db.commit()`
- D) Транзакции не поддерживаются в sqflite

<details>
<summary>Ответ</summary>

**B) `db.transaction((txn) async { ... })`**

```dart
await db.transaction((txn) async {
  final senderId = await txn.query('accounts', where: 'id = ?', whereArgs: [from]);
  await txn.update('accounts', {'balance': newBalance1}, where: 'id = ?', whereArgs: [from]);
  await txn.update('accounts', {'balance': newBalance2}, where: 'id = ?', whereArgs: [to]);
  // Если бросит исключение → автоматический ROLLBACK
});
```

</details>

---

### Вопрос 8 🔴

Как в Drift получить реактивный Stream из query?

- A) `.watch()` вместо `.get()` возвращает `Stream<List<T>>`
- B) `Drift.stream(query)`
- C) Вручную через polling
- D) `db.watchQuery(UsersQuery())`

<details>
<summary>Ответ</summary>

**A) `.watch()` возвращает reactive `Stream`**

```dart
// Разовый запрос:
final users = await (select(usersTable)).get();

// Реактивный stream (обновляется при изменении БД):
final usersStream = (select(usersTable)
    ..where((u) => u.isActive.equals(true))
    ..orderBy([(u) => OrderingTerm.asc(u.name)]))
  .watch(); // Stream<List<User>>

// В виджете:
StreamBuilder<List<User>>(
  stream: db.watchActiveUsers(),
  builder: (context, snapshot) { ... },
)
```

</details>

---

### Вопрос 9 🔴

Как организовать Database Singleton и избежать повторного открытия?

- A) Открывать database при каждом методе
- B) Lazy singleton: статическое поле + `static Database? _instance; static Future<Database> get instance async => _instance ??= await _open();`
- C) `DatabaseFactory.getInstance()`
- D) Через `Provider` или другой DI

<details>
<summary>Ответ</summary>

**D) Через Provider/DI — предпочтительный способ**

```dart
// Drift database как singleton через Riverpod:
@Riverpod(keepAlive: true)
AppDatabase appDatabase(Ref ref) {
  final db = AppDatabase();
  ref.onDispose(db.close);
  return db;
}

// Использование:
final users = await ref.read(appDatabaseProvider).watchAllUsers();
```

`keepAlive: true` сохраняет базу на протяжении всей жизни приложения.

</details>

---

### Вопрос 10 🔴

Как защититься от SQL Injection в sqflite?

- A) Экранировать строки вручную
- B) Всегда использовать параметризованные запросы: `whereArgs`, `db.rawQuery(sql, args)` — никогда не интерполировать пользовательский ввод в SQL строку
- C) Запросы изнутри приложения всегда безопасны
- D) `SqlSanitizer.sanitize(userInput)`

<details>
<summary>Ответ</summary>

**B) Параметризованные запросы — `whereArgs` и `db.rawQuery(sql, args)`**

```dart
// НЕБЕЗОПАСНО ❌:
db.rawQuery("SELECT * FROM users WHERE name = '${userInput}'");

// БЕЗОПАСНО ✅:
db.rawQuery("SELECT * FROM users WHERE name = ?", [userInput]);
db.query('users', where: 'name = ?', whereArgs: [userInput]);
```

sqflite параметризует через `?` placeholder. Drift автоматически параметризует (type-safe queries не имеют SQL injection по природе).

</details>
