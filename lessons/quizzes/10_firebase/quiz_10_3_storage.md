# Квиз: Firebase Storage

**Тема:** 10.3 — Firebase Storage  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как загрузить файл в Firebase Storage?

- A) `FirebaseStorage.upload(file)`
- B) `await FirebaseStorage.instance.ref('path/file.jpg').putFile(file)`
- C) `storage.bucket.upload(file, 'path')`
- D) `FirebaseStorage.instance.upload(file, path)`

<details>
<summary>Ответ</summary>

**B) `ref('path').putFile(file)` — загрузить `dart:io` File**

```dart
final storageRef = FirebaseStorage.instance.ref('users/$uid/avatar.jpg');

// Загрузить File:
final uploadTask = storageRef.putFile(
  file,
  SettableMetadata(contentType: 'image/jpeg'),
);

// Ожидать завершения:
final snapshot = await uploadTask;
final downloadUrl = await snapshot.ref.getDownloadURL();
```

</details>

---

### Вопрос 2 🟢

Как получить download URL загруженного файла?

- A) `storageRef.url`
- B) `await storageRef.getDownloadURL()` — возвращает строку URL
- C) `FirebaseStorage.getUrl(path)`
- D) URL в объекте `UploadTask`

<details>
<summary>Ответ</summary>

**B) `await ref.getDownloadURL()`**

```dart
final ref = FirebaseStorage.instance.ref('users/$uid/avatar.jpg');

// Получить URL (если файл существует):
final url = await ref.getDownloadURL();

// Использовать для отображения:
Image.network(url);
CachedNetworkImage(imageUrl: url);

// Или сразу после загрузки:
final snapshot = await ref.putFile(file);
final url = await snapshot.ref.getDownloadURL();
```

</details>

---

### Вопрос 3 🟢

Как отслеживать прогресс загрузки файла?

- A) `uploadTask.progress.listen()`
- B) `uploadTask.snapshotEvents.listen((snapshot) => snapshot.bytesTransferred / snapshot.totalBytes)`
- C) `storageRef.onProgress(callback)`
- D) Прогресс недоступен в Firebase SDK

<details>
<summary>Ответ</summary>

**B) `uploadTask.snapshotEvents` — Stream с `TaskSnapshot`**

```dart
final task = storageRef.putFile(file);

task.snapshotEvents.listen((TaskSnapshot snapshot) {
  final progress = snapshot.bytesTransferred / snapshot.totalBytes;
  setState(() => _uploadProgress = progress);

  switch (snapshot.state) {
    case TaskState.running: print('Uploading: ${(progress * 100).toStringAsFixed(1)}%');
    case TaskState.success: print('Upload complete!');
    case TaskState.error: print('Error!');
    case TaskState.canceled: print('Canceled');
    case TaskState.paused: print('Paused');
  }
});
```

</details>

---

### Вопрос 4 🟡

Как загрузить данные из `Uint8List` (байты в памяти), а не из File?

- A) `ref.putFile(File.fromBytes(bytes))`
- B) `await ref.putData(bytes, metadata)` — для `Uint8List` в памяти
- C) `ref.upload(ByteData.view(bytes.buffer))`
- D) Нельзя — только `File`

<details>
<summary>Ответ</summary>

**B) `ref.putData(Uint8List bytes)`**

```dart
// Загрузить изображение после обрезки (без записи на диск):
final Uint8List croppedBytes = await cropImage(imageFile);

final task = await storageRef.putData(
  croppedBytes,
  SettableMetadata(contentType: 'image/jpeg'),
);
final url = await task.ref.getDownloadURL();

// Также: ref.putString(base64String, PutStringFormat.base64)
// Для Web с XFile:
final bytes = await xFile.readAsBytes();
await ref.putData(bytes);
```

</details>

---

### Вопрос 5 🟡

Как удалить файл из Firebase Storage?

- A) `storageRef.remove()`
- B) `await storageRef.delete()`
- C) `FirebaseStorage.instance.delete(path)`
- D) `storageRef.unlink()`

<details>
<summary>Ответ</summary>

**B) `await ref.delete()`**

```dart
final ref = FirebaseStorage.instance.ref('users/$uid/avatar.jpg');

try {
  await ref.delete();

  // Также обновить Firestore:
  await userDoc.update({'photoURL': FieldValue.delete()});
} on FirebaseException catch (e) {
  if (e.code == 'object-not-found') {
    print('Файл уже не существует');
  }
}
```

</details>

---

### Вопрос 6 🟡

Как перечислить файлы в папке Firebase Storage?

- A) `storageRef.folder.files`
- B) `await storageRef.listAll()` — возвращает `ListResult` с `items` и `prefixes` (подпапки)
- C) `FirebaseStorage.list(folder)`
- D) Листинг недоступен в Firebase Storage

<details>
<summary>Ответ</summary>

**B) `ref.listAll()` → `ListResult.items` / `prefixes`**

```dart
final ref = FirebaseStorage.instance.ref('users/$uid');

final result = await ref.listAll();

// Файлы в текущей папке:
for (final item in result.items) {
  print('File: ${item.name}');
  final url = await item.getDownloadURL();
}

// Подпапки:
for (final prefix in result.prefixes) {
  print('Folder: ${prefix.name}');
}

// Постраничный листинг (для больших папок):
final page = await ref.list(ListOptions(maxResults: 10));
final nextPage = await ref.list(ListOptions(
  maxResults: 10,
  pageToken: page.nextPageToken,
));
```

</details>

---

### Вопрос 7 🟡

Как настроить Storage Security Rules для разрешения доступа только к своим файлам?

- A) Настраивается только в Flutter коде
- B) `allow read, write: if request.auth.uid == <uid из пути> && request.resource.size < 5 * 1024 * 1024`
- C) `StorageRules.allow('users/{uid}/**')`
- D) Security Rules не применяются к Storage

<details>
<summary>Ответ</summary>

**B) Rules с `request.auth.uid` из wildcard пути**

```javascript
service firebase.storage {
  match /b/{bucket}/o {
    match /users/{userId}/{allPaths=**} {
      // Только владелец, файл < 5MB, только изображения:
      allow read: if request.auth != null && request.auth.uid == userId;
      allow write: if request.auth.uid == userId
                   && request.resource.size < 5 * 1024 * 1024
                   && request.resource.contentType.matches('image/.*');
      allow delete: if request.auth.uid == userId;
    }

    // Публичные аватары (читать всем):
    match /public/{allPaths=**} {
      allow read;
      allow write: if request.auth != null;
    }
  }
}
```

</details>

---

### Вопрос 8 🔴

Как реализовать resumable upload для больших файлов?

- A) Firebase SDK не поддерживает возобновление
- B) Firebase SDK автоматически поддерживает паузу/возобновление через `task.pause()` / `task.resume()`
- C) Использовать Multipart upload API вручную
- D) Разбить файл на чанки и загружать отдельно

<details>
<summary>Ответ</summary>

**B) `task.pause()` / `task.resume()` / `task.cancel()` — встроены в `UploadTask`**

```dart
late UploadTask _uploadTask;

void startUpload(File file) {
  _uploadTask = storageRef.putFile(file);
  _uploadTask.snapshotEvents.listen(_onProgress);
}

void pauseUpload() {
  _uploadTask.pause();
}

void resumeUpload() {
  _uploadTask.resume();
}

void cancelUpload() {
  _uploadTask.cancel();
}

// Состояние:
void _onProgress(TaskSnapshot snap) {
  setState(() {
    _isPaused = snap.state == TaskState.paused;
    _progress = snap.bytesTransferred / snap.totalBytes;
  });
}
```

</details>

---

### Вопрос 9 🔴

Как скачать файл из Firebase Storage на устройство?

- A) `File file = await ref.download()`
- B) `await ref.writeToFile(localFile)` — потоковая запись; или `await ref.getData()` для `Uint8List`
- C) `http.get(await ref.getDownloadURL())`
- D) `ref.copyToLocalPath(path)`

<details>
<summary>Ответ</summary>

**B) `ref.writeToFile(localFile)` — рекомендуется; `getData()` — для маленьких файлов**

```dart
// Скачать на диск (для больших файлов — без загрузки в память):
final localFile = File('${docsDir.path}/document.pdf');
await storageRef.writeToFile(localFile);

// Скачать в память (только для маленьких файлов):
const maxSize = 10 * 1024 * 1024; // 10 MB
final Uint8List? data = await storageRef.getData(maxSize);

// С прогрессом:
final task = storageRef.writeToFile(localFile);
task.snapshotEvents.listen((snap) {
  final progress = snap.bytesTransferred / snap.totalBytes;
});
```

</details>

---

### Вопрос 10 🔴

Как работают кастомные metadata и как обновить metadata после загрузки?

- A) Metadata нельзя изменить после загрузки
- B) `await ref.updateMetadata(SettableMetadata(...))` — обновляет metadata; `await ref.getMetadata()` — читает
- C) Metadata только в Security Rules
- D) `ref.metadata = SettableMetadata(...)`

<details>
<summary>Ответ</summary>

**B) `updateMetadata()` / `getMetadata()`**

```dart
// Обновить metadata:
await storageRef.updateMetadata(SettableMetadata(
  contentType: 'image/webp',
  customMetadata: {
    'uploadedBy': userId,
    'processedAt': DateTime.now().toIso8601String(),
    'originalName': file.path.split('/').last,
  },
  cacheControl: 'public, max-age=3600',
));

// Прочитать:
final metadata = await storageRef.getMetadata();
print(metadata.contentType);
print(metadata.customMetadata?['uploadedBy']);
print(metadata.timeCreated);
print(metadata.size);
```

</details>
