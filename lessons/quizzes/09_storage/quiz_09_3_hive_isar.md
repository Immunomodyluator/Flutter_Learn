# Квиз: Hive и Isar

**Тема:** 09.3 — Hive / Isar  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое Hive и чем он отличается от SQLite?

- A) Hive — реляционная база данных
- B) Hive — легковесная key-value NoSQL база данных написанная на чистом Dart, без нативного кода
- C) Hive — шифрованная версия SharedPreferences
- D) Hive — только для веб-приложений

<details>
<summary>Ответ</summary>

**B) Lightweight key-value NoSQL база, без нативного кода**

Hive преимущества: быстрый (benchmarks ~10x sqflite для простых операций), работает в Web через IndexedDB, простой API. Ограничения: нет joins, нет сложных запросов, нет полнотекстового поиска. Для сложных запросов — Isar или SQLite.

</details>

---

### Вопрос 2 🟢

Как сохранить и получить объект в Hive?

- A) `Hive.save('key', value)` / `Hive.load('key')`
- B) `box.put('key', value)` / `box.get('key')` где `box = await Hive.openBox('name')`
- C) `hive.write(key, value)` / `hive.read(key)`
- D) `Box.instance.set(key, value)` / `Box.instance.get(key)`

<details>
<summary>Ответ</summary>

**B) `box.put()` / `box.get()` через открытый Box**

```dart
// Инициализация:
await Hive.initFlutter();
final box = await Hive.openBox('settings');

// Сохранить:
await box.put('theme', 'dark');
await box.put('userId', 42);

// Получить:
final theme = box.get('theme'); // dynamic
final theme = box.get('theme', defaultValue: 'light'); // с default
```

</details>

---

### Вопрос 3 🟢

Как сохранять кастомные объекты в Hive?

- A) Нельзя — только примитивы
- B) Зарегистрировать `TypeAdapter` аннотацией `@HiveType` + `@HiveField` и сгенерировать через `build_runner`
- C) Вручную сериализовать в `Map` + `hive.put(key, map)`
- D) `box.addObject(user)` — автоматически

<details>
<summary>Ответ</summary>

**B) `@HiveType` + `@HiveField` + `build_runner`**

```dart
@HiveType(typeId: 0)
class User extends HiveObject {
  @HiveField(0) late String name;
  @HiveField(1) late int age;
}
// Регистрация:
Hive.registerAdapter(UserAdapter());
// Использование:
final userBox = await Hive.openBox<User>('users');
await userBox.add(User()..name = 'Alice'..age = 30);
```

</details>

---

### Вопрос 4 🟡

Что такое `HiveObject` и метод `save()`?

- A) Базовый класс всех Hive объектов
- B) Mixin добавляющий `save()`, `delete()`, `key`, `box` к объекту — позволяет сохранять изменения без явного `box.put`
- C) Обёртка для шифрования
- D) Интерфейс для сериализации

<details>
<summary>Ответ</summary>

**B) Mixin с удобными методами `save()`/`delete()`**

```dart
@HiveType(typeId: 0)
class User extends HiveObject {
  @HiveField(0) late String name;
}
// Использование:
final user = userBox.get(0)!;
user.name = 'Bob';
await user.save(); // обновить в box без box.put(user.key, user)
await user.delete(); // удалить из box
```

</details>

---

### Вопрос 5 🟡

Что такое Isar и зачем его использовать вместо Hive?

- A) Isar — зашифрованная версия Hive
- B) Isar — полноценная NoSQL база данных с индексами, полнотекстовым поиском, реактивными запросами и Dart-native синтаксисом
- C) Isar устарел, Hive 3.0 заменяет его
- D) Isar — только для Desktop

<details>
<summary>Ответ</summary>

**B) Мощная NoSQL база с индексами, реактивными запросами, полнотекстовым поиском**

Isar vs Hive:

- **Индексы**: быстрые запросы по полям
- **Фильтрация**: `.where().nameStartsWith('Al')` — type-safe!
- **Полнотекстовый поиск**
- **Реактивность**: `.watchLazy()`, `.watch()`
- **Без `build_runner`** (Isar Inspector)
- Speed: Isar быстрее Hive для сложных запросов

</details>

---

### Вопрос 6 🟡

Как создать коллекцию объектов в Isar?

- A) `@HiveType` аннотация
- B) `@collection` аннотация на класс, `@Id` на первичный ключ → `build_runner`
- C) `IsarCollection<User>.create()`
- D) Автоматически для всех Dart классов

<details>
<summary>Ответ</summary>

**B) `@collection` + `@Id` + `build_runner`**

```dart
import 'package:isar/isar.dart';
part 'user.g.dart';

@collection
class User {
  Id id = Isar.autoIncrement;
  late String name;
  @Index()
  late String email;
  late int age;
}
```

`flutter pub run build_runner build` генерирует `user.g.dart` + type-safe query builder.

</details>

---

### Вопрос 7 🟡

Как выполнить реактивный запрос с фильтрацией в Isar?

- A) `isar.users.where().filter().nameContains('Al').findAll()`
- B) `isar.users.filter().nameContains('Al').watch()`
- C) `isar.users.watchLazy()` + ручной запрос
- D) A — для одноразового запроса, B — для реактивного watch

<details>
<summary>Ответ</summary>

**D) A — разовый, B — реактивный**

```dart
// Разовый:
final users = await isar.users
    .filter()
    .ageGreaterThan(18)
    .nameContains('Alice', caseSensitive: false)
    .sortByNameDesc()
    .findAll();

// Реактивный Stream (обновляется при изменении):
final stream = isar.users
    .filter()
    .isActiveEqualTo(true)
    .watch(fireImmediately: true);
```

</details>

---

### Вопрос 8 🔴

Как использовать транзакции в Isar?

- A) `isar.transaction(() async { ... })`
- B) `isar.writeTxn(() async { await isar.users.put(user); })` для записи; `isar.txn(() async { ... })` для чтения
- C) `isar.beginWrite(); ...; isar.commit()`
- D) Транзакции не нужны — Isar автоматически атомарен

<details>
<summary>Ответ</summary>

**B) `isar.writeTxn()` для записи, `isar.txn()` для чтения**

```dart
// Запись (обязательно в writeTxn):
await isar.writeTxn(() async {
  final id = await isar.users.put(newUser);
  await isar.orders.put(Order(userId: id, ...));
});

// Чтение (опционально, но улучшает производительность для множественных читаний):
await isar.txn(() async {
  final user = await isar.users.get(id);
  final orders = await isar.orders.filter().userIdEqualTo(id).findAll();
});
```

</details>

---

### Вопрос 9 🔴

Как шифровать Hive box?

- A) `HiveBox.encrypted(key: encryptionKey)`
- B) `Hive.openBox('secure', encryptionCipher: HiveAesCipher(key))` — AES-256 шифрование
- C) `EncryptedHive.init(password: 'pass')`
- D) Hive не поддерживает шифрование

<details>
<summary>Ответ</summary>

**B) `Hive.openBox('name', encryptionCipher: HiveAesCipher(key))`**

```dart
// Генерация и безопасное хранение ключа:
var key = Hive.generateSecureKey(); // List<int> - 32 байта
// Сохранить ключ в flutter_secure_storage:
await secureStorage.write(key: 'hive_key', value: base64Url.encode(key));

// Открыть зашифрованный box:
final encryptedBox = await Hive.openBox(
  'secure_data',
  encryptionCipher: HiveAesCipher(key),
);
```

Ключ хранить в `flutter_secure_storage`, не в SharedPreferences.

</details>

---

### Вопрос 10 🔴

Как выполнить полнотекстовый поиск в Isar?

- A) `.filter().textContains('query')`
- B) `@Index(type: IndexType.value)` + `.filter().titleWordsWith('flutter')`
- C) `isar.users.search('query')`
- D) Полнотекстовый поиск отсутствует в Isar

<details>
<summary>Ответ</summary>

**B) `@Index(type: IndexType.value)` / `IndexType.words/hash` + специальные filter методы**

```dart
@collection
class Article {
  Id id = Isar.autoIncrement;

  @Index(type: IndexType.value)
  late String title;

  @Index(type: IndexType.words) // разбивает на слова для FTS
  late String content;
}

// Полнотекстовый поиск:
final results = await isar.articles
    .filter()
    .contentWordsWith('flutter') // поиск по слову
    .findAll();
```

</details>
