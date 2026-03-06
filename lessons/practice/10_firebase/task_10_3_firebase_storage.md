# 10.3: Firebase Storage — загрузка фото блюд

> Project: FitMenu | Глава 10 — Firebase

### 10.3: FoodPhotoService с Firebase Storage

🎯 **Цель шага:** Реализовать загрузку пользовательских фото блюд в Firebase Storage — пользователь выбирает фото из галереи, оно загружается в облако и URL сохраняется в Firestore.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  firebase_storage: ^11.6.0
  image_picker: ^1.0.0
```

Реализуй `lib/features/meal_log/services/food_photo_service.dart`.

**FoodPhotoService:**
```dart
class FoodPhotoService {
  final _storage = FirebaseStorage.instance;
  final _picker  = ImagePicker();

  // Выбор фото из галереи
  Future<File?> pickPhoto() async { ... }

  // Загрузка в Storage + возврат URL
  Future<String> uploadPhoto(File photo, String userId) async { ... }

  // Удаление фото по URL
  Future<void> deletePhoto(String photoUrl) async { ... }

  // Виджет прогресса загрузки через Stream
  Stream<double> uploadWithProgress(File photo, String userId) { ... }
}
```

**Путь хранения:** `users/{userId}/food_photos/{timestamp}_{uuid}.jpg`

**uploadPhoto:**
1. Сжать изображение перед загрузкой (`image` пакет или `ResizeImage`)
2. `storage.ref(path).putFile(file)`
3. `await task.getDownloadURL()`
4. Вернуть URL

**uploadWithProgress** — через `UploadTask.snapshotEvents`:
```dart
uploadTask.snapshotEvents.map(
  (s) => s.bytesTransferred / s.totalBytes
)
```

**UI — кнопка с прогрессом:**
```dart
StreamBuilder<double>(
  stream: _uploadStream,
  builder: (context, snapshot) {
    if (snapshot.hasData) {
      return LinearProgressIndicator(value: snapshot.data);
    }
    return ElevatedButton(onPressed: _pickAndUpload, child: Text('Добавить фото'));
  },
)
```

---

✅ **Критерии приёмки:**
- [ ] Путь к файлу включает `userId` (изоляция данных)
- [ ] Прогресс загрузки отображается через `LinearProgressIndicator`
- [ ] `UploadTask.snapshotEvents` используется для прогресса
- [ ] `getDownloadURL()` вызывается после успешной загрузки
- [ ] Ошибки загрузки обрабатываются (нет сети, размер файла)
- [ ] `deletePhoto` использует `ref.refFromURL(url).delete()`

---

💡 **Подсказка:** `storage.ref().child(path)` создаёт ссылку. `putFile()` возвращает `UploadTask` — не `Future`. Для показа прогресса используй `.snapshotEvents` стрим. `FirebaseStorage.instance.refFromURL(downloadUrl)` позволяет получить ссылку по URL для удаления.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_log/services/food_photo_service.dart

import 'dart:io';
import 'package:firebase_storage/firebase_storage.dart';
import 'package:image_picker/image_picker.dart';

class FoodPhotoService {
  final _storage = FirebaseStorage.instance;
  final _picker  = ImagePicker();

  Future<File?> pickPhoto({ImageSource source = ImageSource.gallery}) async {
    final picked = await _picker.pickImage(
      source: source,
      maxWidth: 1024,
      maxHeight: 1024,
      imageQuality: 85,
    );
    return picked != null ? File(picked.path) : null;
  }

  Future<String> uploadPhoto(File photo, String userId) async {
    final fileName = '${DateTime.now().millisecondsSinceEpoch}.jpg';
    final ref = _storage.ref('users/$userId/food_photos/$fileName');

    final task = await ref.putFile(
      photo,
      SettableMetadata(contentType: 'image/jpeg'),
    );

    return task.ref.getDownloadURL();
  }

  Stream<double> uploadWithProgress(File photo, String userId) {
    final fileName = '${DateTime.now().millisecondsSinceEpoch}.jpg';
    final ref = _storage.ref('users/$userId/food_photos/$fileName');
    final task = ref.putFile(photo);

    return task.snapshotEvents.map(
      (snapshot) => snapshot.bytesTransferred / snapshot.totalBytes,
    );
  }

  Future<void> deletePhoto(String photoUrl) async {
    try {
      await _storage.refFromURL(photoUrl).delete();
    } on FirebaseException catch (e) {
      if (e.code != 'object-not-found') rethrow;
      // Файл уже удалён — игнорируем
    }
  }
}
```

</details>
