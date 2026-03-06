# 10.2 Cloud Firestore

## 1. Суть

Firestore — NoSQL документная база данных в реальном времени от Firebase. Структура: **коллекции** → **документы** → **поля** (+ вложенные коллекции). Поддерживает реактивные потоки (`Stream`) — UI обновляется мгновенно при изменении данных.

```yaml
dependencies:
  cloud_firestore: ^4.15.9
```

---

## 2. Структура данных

```
Firestore
├── users/                     ← коллекция
│   ├── uid123/                ← документ (id = uid пользователя)
│   │   ├── name: "Ivan"
│   │   ├── email: "ivan@..."
│   │   └── createdAt: Timestamp
│   └── uid456/
│       └── ...
└── posts/
    ├── post1/
    │   ├── title: "Hello"
    │   ├── authorId: "uid123"
    │   └── comments/          ← вложенная коллекция
    │       └── comment1/
    └── post2/
```

---

## 3. Основной синтаксис

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

final db = FirebaseFirestore.instance;

// --- Запись ---
// Создать с автоID
await db.collection('posts').add({
  'title': 'Новый пост',
  'authorId': 'uid123',
  'createdAt': FieldValue.serverTimestamp(),
});

// Создать с конкретным ID
await db.collection('users').doc('uid123').set({
  'name': 'Ivan',
  'email': 'ivan@example.com',
});

// Обновить отдельные поля (не перезаписывает всё)
await db.collection('users').doc('uid123').update({
  'name': 'Ivan Petrov',
  'score': FieldValue.increment(10), // атомарный инкремент
});

// Добавить элемент в массив
await db.collection('users').doc('uid123').update({
  'tags': FieldValue.arrayUnion(['flutter']),
});

// Удалить
await db.collection('posts').doc('post1').delete();

// --- Чтение (Future) ---
final doc = await db.collection('users').doc('uid123').get();
if (doc.exists) {
  final data = doc.data()!;
  final name = data['name'] as String;
}

// Коллекция с фильтром
final snapshot = await db
    .collection('posts')
    .where('authorId', isEqualTo: 'uid123')
    .orderBy('createdAt', descending: true)
    .limit(20)
    .get();

final posts = snapshot.docs.map((doc) => {
  'id': doc.id,
  ...doc.data(),
}).toList();

// --- Реактивное чтение (Stream) ---
// Один документ
Stream<DocumentSnapshot> userStream =
    db.collection('users').doc('uid123').snapshots();

// Коллекция
Stream<QuerySnapshot> postsStream =
    db.collection('posts').orderBy('createdAt', descending: true).snapshots();
```

---

## 4. Реальный пример — Repository

```dart
import 'package:cloud_firestore/cloud_firestore.dart';

class Post {
  final String id;
  final String title;
  final String content;
  final String authorId;
  final DateTime createdAt;

  const Post({
    required this.id,
    required this.title,
    required this.content,
    required this.authorId,
    required this.createdAt,
  });

  factory Post.fromDoc(DocumentSnapshot doc) {
    final data = doc.data() as Map<String, dynamic>;
    return Post(
      id: doc.id,
      title: data['title'] as String,
      content: data['content'] as String,
      authorId: data['authorId'] as String,
      createdAt: (data['createdAt'] as Timestamp).toDate(),
    );
  }

  Map<String, dynamic> toMap() => {
        'title': title,
        'content': content,
        'authorId': authorId,
        'createdAt': FieldValue.serverTimestamp(),
      };
}

class PostRepository {
  final FirebaseFirestore _db;

  PostRepository([FirebaseFirestore? db])
      : _db = db ?? FirebaseFirestore.instance;

  // Реактивный список постов
  Stream<List<Post>> watchPosts() => _db
      .collection('posts')
      .orderBy('createdAt', descending: true)
      .snapshots()
      .map((snapshot) => snapshot.docs.map(Post.fromDoc).toList());

  // Пагинация
  Future<List<Post>> getPostsPage({
    DocumentSnapshot? lastDoc,
    int limit = 20,
  }) async {
    Query query = _db
        .collection('posts')
        .orderBy('createdAt', descending: true)
        .limit(limit);

    if (lastDoc != null) {
      query = query.startAfterDocument(lastDoc);
    }

    final snapshot = await query.get();
    return snapshot.docs.map(Post.fromDoc).toList();
  }

  Future<String> createPost(String title, String content, String authorId) async {
    final ref = await _db.collection('posts').add(
      Post(
        id: '',
        title: title,
        content: content,
        authorId: authorId,
        createdAt: DateTime.now(),
      ).toMap(),
    );
    return ref.id;
  }

  Future<void> deletePost(String postId) =>
      _db.collection('posts').doc(postId).delete();
}

// --- Транзакция (атомарное обновление) ---
Future<void> likePost(String postId, String userId) async {
  final db = FirebaseFirestore.instance;
  final postRef = db.collection('posts').doc(postId);

  await db.runTransaction((transaction) async {
    final post = await transaction.get(postRef);
    final currentLikes = post.data()?['likes'] as int? ?? 0;
    transaction.update(postRef, {'likes': currentLikes + 1});
  });
}
```

---

## 5. Правила безопасности (Security Rules)

```javascript
// firestore.rules — ВСЕГДА настраивай!
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Посты: читают все, пишет только автор
    match /posts/{postId} {
      allow read: if true;
      allow create: if request.auth != null
        && request.resource.data.authorId == request.auth.uid;
      allow update, delete: if request.auth != null
        && resource.data.authorId == request.auth.uid;
    }

    // Профиль: только сам пользователь
    match /users/{userId} {
      allow read, write: if request.auth != null
        && request.auth.uid == userId;
    }
  }
}
```

---

## 6. Под капотом

- Firestore — документная база с offline-кешем: данные доступны без интернета.
- `snapshots()` возвращает `Stream` через WebSocket (или gRPC на мобильных).
- `FieldValue.serverTimestamp()` — записывает время сервера, не клиента.
- Firestore ограничен: до 1 запись в секунду на документ, 20 000 чтений/записей/удалений в день (бесплатный тариф).

---

## 7. Типичные ошибки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `PERMISSION_DENIED` | Security Rules запрещают доступ | Проверь Rules в Firebase Console |
| `null` данные при `doc.data()` | Документ не существует | Проверяй `doc.exists` |
| Каждый rebuild запрашивает данные | `get()` внутри `build()` | Вынести в `initState` или Provider |
| Медленные запросы | Нет индексов для сложных where+orderBy | Создай композитный индекс в Console |
| Timestamp null при offline | `serverTimestamp()` не заполнен | Используй `hasPendingWrites` для проверки |

---

## 8. Рекомендации

1. **Security Rules обязательны** — по умолчанию всё открыто или закрыто, нужно явно настроить.
2. **`snapshots()` для real-time**, `get()` для однократного чтения.
3. **Денормализуй данные** — Firestore не поддерживает JOIN, храни нужные поля прямо в документе.
4. **Пагинация через `startAfterDocument`** — не через `offset`.
5. **Транзакции** для операций, требующих атомарности (лайки, счётчики).
6. **Индексы** — Firestore сам предложит создать индекс, если запрос не работает (ошибка в логах).
