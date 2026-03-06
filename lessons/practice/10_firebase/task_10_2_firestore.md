# 10.2: Firestore — облачная синхронизация плана питания

> Project: FitMenu | Глава 10 — Firebase

### 10.2: MealPlanRepository с Cloud Firestore

🎯 **Цель шага:** Сохранять план питания пользователя в Cloud Firestore — данные синхронизируются между устройствами в реальном времени, доступны офлайн через встроенное кеширование.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  cloud_firestore: ^4.14.0
```

**Структура коллекции Firestore:**
```
users/{userId}/meal_plans/{date}/entries/{entryId}
  - recipeId: string
  - title: string
  - calories: number
  - protein: number
  - fat: number
  - carbs: number
  - mealType: string ('breakfast'|'lunch'|'dinner')
  - addedAt: timestamp
```

Реализуй `lib/features/meal_plan/data/firestore_meal_plan_repository.dart`.

**FirestoreMealPlanRepository:**
```dart
class FirestoreMealPlanRepository {
  final _db   = FirebaseFirestore.instance;
  final _auth = FirebaseAuth.instance;

  String get _userId => _auth.currentUser!.uid;

  CollectionReference<Map<String,dynamic>> _entriesRef(DateTime date) =>
      _db.collection('users').doc(_userId)
         .collection('meal_plans').doc(_dateKey(date))
         .collection('entries');

  Stream<List<MealEntry>> watchEntriesForDate(DateTime date) { ... }
  Future<void> addEntry(MealEntry entry, DateTime date) async { ... }
  Future<void> removeEntry(String entryId, DateTime date) async { ... }
}
```

**watchEntriesForDate:** возвращает `Stream`, обновляющийся в реальном времени.

**Офлайн поддержка:**
```dart
// В main.dart при инициализации:
FirebaseFirestore.instance.settings = const Settings(
  persistenceEnabled: true,
  cacheSizeBytes: Settings.CACHE_SIZE_UNLIMITED,
);
```

---

✅ **Критерии приёмки:**
- [ ] Данные сохраняются в структуру `users/{uid}/meal_plans/{date}/entries`
- [ ] `watchEntriesForDate` возвращает `Stream` с реальным временем
- [ ] `persistenceEnabled: true` включён для офлайн-поддержки
- [ ] `_userId` берётся из `FirebaseAuth.instance.currentUser!.uid`
- [ ] Timestamp конвертируется через `FieldValue.serverTimestamp()`
- [ ] При удалении пользователя — данные изолированы по `uid`

---

💡 **Подсказка:** `snapshots()` на CollectionReference даёт `Stream<QuerySnapshot>`. Маппинг: `snapshot.docs.map((d) => MealEntry.fromMap(d.data()))`. `FieldValue.serverTimestamp()` надёжнее чем `DateTime.now()` — сервер ставит время независимо от часового пояса клиента.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_plan/data/firestore_meal_plan_repository.dart

import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_auth/firebase_auth.dart';

class FirestoreMealPlanRepository {
  final _db   = FirebaseFirestore.instance;
  final _auth = FirebaseAuth.instance;

  String get _userId {
    final uid = _auth.currentUser?.uid;
    assert(uid != null, 'Пользователь не авторизован');
    return uid!;
  }

  String _dateKey(DateTime date) =>
      '${date.year}-'
      '${date.month.toString().padLeft(2, '0')}-'
      '${date.day.toString().padLeft(2, '0')}';

  CollectionReference<Map<String, dynamic>> _entriesRef(DateTime date) => _db
      .collection('users')
      .doc(_userId)
      .collection('meal_plans')
      .doc(_dateKey(date))
      .collection('entries');

  Stream<List<MealEntry>> watchEntriesForDate(DateTime date) {
    return _entriesRef(date)
        .orderBy('addedAt')
        .snapshots()
        .map((snap) => snap.docs
            .map((d) => MealEntry.fromMap({...d.data(), 'id': d.id}))
            .toList());
  }

  Future<void> addEntry(MealEntry entry, DateTime date) async {
    await _entriesRef(date).doc(entry.id).set({
      'recipeId': entry.recipeId,
      'title':    entry.title,
      'calories': entry.calories,
      'protein':  entry.protein,
      'fat':      entry.fat,
      'carbs':    entry.carbs,
      'mealType': entry.mealType,
      'addedAt':  FieldValue.serverTimestamp(),
    });
  }

  Future<void> removeEntry(String entryId, DateTime date) async {
    await _entriesRef(date).doc(entryId).delete();
  }

  Future<double> totalCaloriesForDate(DateTime date) async {
    final snap = await _entriesRef(date).get();
    return snap.docs.fold(0.0, (sum, d) {
      return sum + ((d.data()['calories'] as num?)?.toDouble() ?? 0.0);
    });
  }
}
```

</details>
