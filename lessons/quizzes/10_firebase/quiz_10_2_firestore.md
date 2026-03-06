# Квиз: Cloud Firestore

**Тема:** 10.2 — Cloud Firestore  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какова структура данных в Cloud Firestore?

- A) Таблицы и строки (реляционная)
- B) Коллекции → Документы → Поля; документы могут содержать подколлекции
- C) Ключ-значение (только строки)
- D) XML документы

<details>
<summary>Ответ</summary>

**B) Коллекции → Документы → Поля (+ вложенные подколлекции)**

```
📁 users (collection)
  📄 uid123 (document)
    name: "Alice"
    age: 30
    📁 orders (subcollection)
      📄 order1
        total: 150
```

Максимум: 1 MB на документ, 20 000 полей. Firestore — не для blob данных.

</details>

---

### Вопрос 2 🟢

Как добавить документ в коллекцию с автогенерированным ID?

- A) `db.users.insert(data)`
- B) `await FirebaseFirestore.instance.collection('users').add(data)` — возвращает `DocumentReference`
- C) `db.collection('users').create(data)`
- D) `Firestore.add('users', data)`

<details>
<summary>Ответ</summary>

**B) `collection.add(data)` = авто-ID; `doc(id).set(data)` = свой ID**

```dart
final db = FirebaseFirestore.instance;

// Авто-ID:
final ref = await db.collection('users').add({
  'name': 'Alice',
  'createdAt': FieldValue.serverTimestamp(),
});
print('ID: ${ref.id}');

// Свой ID:
await db.collection('users').doc('custom-id').set({
  'name': 'Bob',
});
```

</details>

---

### Вопрос 3 🟢

Как получить документ по ID?

- A) `db.find('users', id)`
- B) `await db.collection('users').doc(id).get()` → `DocumentSnapshot`
- C) `db.users.where('id', isEqualTo: id).first()`
- D) `Firestore.getDoc(collection, id)`

<details>
<summary>Ответ</summary>

**B) `doc(id).get()` → `DocumentSnapshot`**

```dart
final snap = await db.collection('users').doc(userId).get();

if (snap.exists) {
  final data = snap.data()!; // Map<String, dynamic>
  final name = data['name'] as String;

  // Или с fromJson:
  final user = User.fromJson({...data, 'id': snap.id});
}
```

</details>

---

### Вопрос 4 🟡

Как подписаться на real-time обновления документа?

- A) `db.doc(path).listenForUpdates(callback)`
- B) `db.collection('users').doc(id).snapshots()` — возвращает `Stream<DocumentSnapshot>`
- C) `FirebaseFirestore.stream(path)`
- D) `db.doc(path).watch(onChange)`

<details>
<summary>Ответ</summary>

**B) `.snapshots()` → `Stream<DocumentSnapshot>`**

```dart
// Слушать один документ:
db.collection('users').doc(uid).snapshots().listen((snap) {
  if (snap.exists) {
    final user = User.fromJson(snap.data()!);
    setState(() => _user = user);
  }
});

// Слушать коллекцию с фильтрацией:
db.collection('messages')
    .where('roomId', isEqualTo: currentRoomId)
    .orderBy('createdAt', descending: true)
    .limit(50)
    .snapshots()
    .listen((querySnapshot) {
      final messages = querySnapshot.docs
          .map((d) => Message.fromJson(d.data()))
          .toList();
    });
```

</details>

---

### Вопрос 5 🟡

Чем отличается `set()` от `update()`?

- A) `set` быстрее; `update` — с транзакцией
- B) `set()` заменяет документ целиком (или создаёт новый); `update()` обновляет конкретные поля (бросает ошибку если документ не существует)
- C) `update()` поддерживает вложенные поля; `set()` — только корень
- D) Нет разницы

<details>
<summary>Ответ</summary>

**B) `set` = replace/create; `update` = partial update (ошибка если нет документа)**

```dart
// set() — полная замена:
await doc.set({'name': 'Alice', 'age': 30}); // остальные поля удалятся

// set() с merge — как update, но создаёт если нет:
await doc.set({'age': 31}, SetOptions(merge: true));

// update() — только указанные поля:
await doc.update({
  'age': 31,
  'address.city': 'Moscow', // nested field через точку
  'tags': FieldValue.arrayUnion(['flutter']),
  'score': FieldValue.increment(1),
});
```

</details>

---

### Вопрос 6 🟡

Что такое транзакция в Firestore и когда её использовать?

- A) Пакетная запись нескольких документов
- B) Атомарная операция чтение→запись: гарантирует что запись основана на актуальных данных (избегает race conditions)
- C) Автоматическое разрешение конфликтов
- D) Шифрование данных при передаче

<details>
<summary>Ответ</summary>

**B) Атомарная read→write операция против race conditions**

```dart
// Транзакция: прочитать баланс и списать (атомарно):
await db.runTransaction((transaction) async {
  final snap = await transaction.get(userDoc);
  final balance = snap.get('balance') as int;
  if (balance < amount) throw Exception('Insufficient funds');
  transaction.update(userDoc, {'balance': balance - amount});
  transaction.update(recipientDoc, {'balance': FieldValue.increment(amount)});
});

// WriteBatch — без чтения, просто батч запись (быстрее):
final batch = db.batch();
batch.set(doc1, data1);
batch.update(doc2, {'field': 'value'});
batch.delete(doc3);
await batch.commit();
```

</details>

---

### Вопрос 7 🟡

Как работают составные индексы (composite indexes) в Firestore?

- A) Автоматически создаются для всех `where` запросов
- B) Нужны при комбинировании `where()` по разным полям или `where()` + `orderBy()` — создаются в Firebase Console или через `firestore.indexes.json`
- C) Только для `orderBy` запросов
- D) Реализованы через Google Cloud Spanner

<details>
<summary>Ответ</summary>

**B) Нужны при комбинировании фильтров по разным полям / orderBy**

```dart
// Этот запрос требует составного индекса (status + createdAt):
db.collection('orders')
    .where('status', isEqualTo: 'pending')
    .orderBy('createdAt', descending: true)
    .limit(20);
```

Firestore сам предложит ссылку в лог-консоли при ошибке:
`"The query requires an index. You can create it here: https://..."`

`firestore.indexes.json`:

```json
{
  "indexes": [
    {
      "collectionGroup": "orders",
      "fields": [
        { "fieldPath": "status", "order": "ASCENDING" },
        { "fieldPath": "createdAt", "order": "DESCENDING" }
      ]
    }
  ]
}
```

</details>

---

### Вопрос 8 🔴

Как написать Security Rules для разрешения пользователю читать/писать только свои данные?

- A) Security Rules не поддерживают auth проверки
- B) `allow read, write: if request.auth != null && request.auth.uid == userId`
- C) Настраивается только в коде Flutter
- D) `allow read: if true`

<details>
<summary>Ответ</summary>

**B) `request.auth.uid == resource.data.userId` или путевый wildcard**

```javascript
// Firestore Security Rules:
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Пользователь может читать/писать только свой документ:
    match /users/{userId} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }

    // Пользователь может читать чужие посты, но писать только свои:
    match /posts/{postId} {
      allow read: if request.auth != null;
      allow write: if request.auth.uid == resource.data.authorId;
      allow create: if request.auth.uid == request.resource.data.authorId;
    }
  }
}
```

</details>

---

### Вопрос 9 🔴

Как оптимизировать стоимость Firestore (чтения тарифицируются)?

- A) Хранить всё в одном огромном документе
- B) Использовать `select()` для частичного чтения, кэшировать данные локально, избегать неограниченных подписок, использовать `limit()`
- C) Firestore бесплатен без ограничений
- D) Только пакетная запись снижает стоимость

<details>
<summary>Ответ</summary>

**B) Selective reads + кэширование + limit + правильная денормализация**

Стратегии:

1. **`select()`**: `doc.get().then(s => s.get('field'))` — читает весь документ, но можно денормализовать
2. **Offline кэш**: Firestore SDK кэширует локально — `GetOptions(source: Source.cache)` для force cache
3. **Pagination**: `startAfter(lastDoc)` вместо `limit(1000)`
4. **Денормализация**: дублировать часто читаемые данные (имя автора в посте) → меньше joins/reads
5. **Aggregation counts**: `FieldValue.increment()` в отдельном counter doc вместо `count()` запроса

</details>

---

### Вопрос 10 🔴

Как обрабатывать конфликты при offline-first работе с Firestore?

- A) Firestore автоматически разрешает все конфликты
- B) Firestore использует Last-Write-Wins для обычных полей; `FieldValue.increment/arrayUnion` атомарны даже offline — использовать их для счётчиков и массивов
- C) Транзакции работают offline
- D) Offline запись запрещена по умолчанию

<details>
<summary>Ответ</summary>

**B) LWW для полей; атомарные операции для предотвращения конфликтов**

```dart
// ПЛОХО: конфликт при offline (race condition):
final snap = await doc.get(const GetOptions(source: Source.cache));
final count = snap.get('likes') as int;
await doc.update({'likes': count + 1}); // может перезаписать чужое изменение

// ХОРОШО: атомарный increment (безопасно offline):
await doc.update({'likes': FieldValue.increment(1)});

// Массивы:
await doc.update({
  'tags': FieldValue.arrayUnion(['flutter']), // нет дублей
  'removed': FieldValue.arrayRemove(['old']),
});

// Проверить pending writes:
doc.snapshots().listen((snap) {
  print(snap.metadata.hasPendingWrites); // true если есть несохранённые
});
```

</details>
