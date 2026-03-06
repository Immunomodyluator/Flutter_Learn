# 10.3 Firebase Storage

## 1. Суть

Firebase Storage — облачное хранилище файлов (фото, видео, документы). Интегрируется с Firebase Auth, поддерживает прогресс загрузки и паузу/возобновление.

```yaml
dependencies:
  firebase_storage: ^11.6.9
  image_picker: ^1.0.7     # выбор фото из галереи/камеры
```

---

## 2. Основной синтаксис

```dart
import 'package:firebase_storage/firebase_storage.dart';
import 'dart:io';

final storage = FirebaseStorage.instance;

// Ссылка на файл
final ref = storage.ref('users/uid123/avatar.jpg');

// Загрузка файла (upload)
final uploadTask = ref.putFile(File('/path/to/image.jpg'));

// Наблюдение за прогрессом
uploadTask.snapshotEvents.listen((TaskSnapshot snapshot) {
  final progress = snapshot.bytesTransferred / snapshot.totalBytes;
  print('Прогресс: ${(progress * 100).toStringAsFixed(1)}%');
});

// Ожидание завершения
await uploadTask;

// Получить URL для скачивания
final downloadUrl = await ref.getDownloadURL();

// Загрузка из байт
await ref.putData(
  imageBytes,
  SettableMetadata(contentType: 'image/jpeg'),
);

// Скачать байты
final bytes = await ref.getData(maxSize: 10 * 1024 * 1024); // 10 MB max

// Удалить файл
await ref.delete();

// Список файлов в директории
final result = await storage.ref('users/uid123/photos').listAll();
for (final item in result.items) {
  print(item.name);
}
```

---

## 3. Реальный пример — загрузка аватара

```dart
import 'package:firebase_storage/firebase_storage.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:image_picker/image_picker.dart';
import 'dart:io';

class StorageRepository {
  final FirebaseStorage _storage;
  final FirebaseAuth _auth;

  StorageRepository()
      : _storage = FirebaseStorage.instance,
        _auth = FirebaseAuth.instance;

  Future<String> uploadAvatar(File imageFile) async {
    final uid = _auth.currentUser?.uid;
    if (uid == null) throw Exception('Пользователь не авторизован');

    final ext = imageFile.path.split('.').last;
    final ref = _storage.ref('avatars/$uid/avatar.$ext');

    final uploadTask = ref.putFile(
      imageFile,
      SettableMetadata(contentType: 'image/$ext'),
    );

    await uploadTask;
    return ref.getDownloadURL();
  }

  // Загрузка с прогрессом
  Stream<double> uploadAvatarWithProgress(File imageFile) async* {
    final uid = _auth.currentUser!.uid;
    final ref = _storage.ref('avatars/$uid/avatar.jpg');

    final uploadTask = ref.putFile(imageFile);

    await for (final snapshot in uploadTask.snapshotEvents) {
      if (snapshot.totalBytes > 0) {
        yield snapshot.bytesTransferred / snapshot.totalBytes;
      }
    }
  }

  Future<void> deleteAvatar() async {
    final uid = _auth.currentUser!.uid;
    final ref = _storage.ref('avatars/$uid');
    final list = await ref.listAll();
    for (final item in list.items) {
      await item.delete();
    }
  }
}

// --- Виджет загрузки аватара ---
class AvatarUploadWidget extends StatefulWidget {
  const AvatarUploadWidget({super.key});

  @override
  State<AvatarUploadWidget> createState() => _AvatarUploadWidgetState();
}

class _AvatarUploadWidgetState extends State<AvatarUploadWidget> {
  final _storage = StorageRepository();
  final _picker = ImagePicker();
  double? _progress;
  String? _avatarUrl;

  Future<void> _pickAndUpload() async {
    final picked = await _picker.pickImage(
      source: ImageSource.gallery,
      maxWidth: 512,
      maxHeight: 512,
      imageQuality: 85,
    );
    if (picked == null) return;

    setState(() => _progress = 0);

    _storage.uploadAvatarWithProgress(File(picked.path)).listen(
      (progress) => setState(() => _progress = progress),
      onDone: () async {
        final url = await _storage.uploadAvatar(File(picked.path));
        setState(() {
          _avatarUrl = url;
          _progress = null;
        });
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        CircleAvatar(
          radius: 60,
          backgroundImage: _avatarUrl != null ? NetworkImage(_avatarUrl!) : null,
          child: _avatarUrl == null ? const Icon(Icons.person, size: 60) : null,
        ),
        if (_progress != null) ...[
          const SizedBox(height: 8),
          LinearProgressIndicator(value: _progress),
          Text('${(_progress! * 100).toInt()}%'),
        ],
        TextButton(
          onPressed: _pickAndUpload,
          child: const Text('Загрузить фото'),
        ),
      ],
    );
  }
}
```

---

## 4. Security Rules для Storage

```javascript
// storage.rules
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    // Аватары: только сам пользователь
    match /avatars/{userId}/{fileName} {
      allow read: if true; // публичные аватары
      allow write: if request.auth != null
        && request.auth.uid == userId
        && request.resource.size < 5 * 1024 * 1024  // max 5MB
        && request.resource.contentType.matches('image/.*');
    }
  }
}
```

---

## 5. Под капотом

- Firebase Storage использует Google Cloud Storage под капотом.
- Загрузка разбита на чанки (resumable upload) для надёжности.
- `putFile` автоматически возобновляет загрузку при обрыве соединения.
- Download URL — публичная ссылка с токеном аутентификации (можно отозвать).

---

## 6. Типичные ошибки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `UNAUTHORIZED` | Security Rules не разрешают запись | Проверь rules, убедись что пользователь залогинен |
| `QUOTA_EXCEEDED` | Превышен лимит бесплатного плана | Используй `maxWidth`/`maxHeight` при пике, сжимай изображения |
| Загружаешь оригинал 20MB | Нет сжатия | `imageQuality: 85` и `maxWidth: 1024` в `ImagePicker` |
| URL не работает через день | Токен в URL отозван | Не кешируй URL вечно, получай заново при необходимости |
| Загрузка падает в фоне | App killed OS | Используй `WorkManager`-подобный подход для критичных загрузок |

---

## 7. Рекомендации

1. **Сжимай изображения** перед загрузкой — `imageQuality: 85`, `maxWidth: 1024`.
2. **Security Rules** для Storage — ограничивай тип файлов (`contentType.matches('image/.*')`) и размер.
3. **Показывай прогресс** при загрузке видео и больших файлов.
4. **Структура пути**: `{тип}/{uid}/{filename}` — удобно для Security Rules.
5. **Не храни Download URL** вечно — если безопасность важна, запрашивай заново.
