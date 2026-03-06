# Квиз: Работа с файлами

**Тема:** 09.4 — path_provider / File API / Pickers  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой пакет предоставляет пути к директориям файловой системы (Documents, Temp, Cache)?

- A) `dart:io` — только стандартная библиотека
- B) `path_provider` — кроссплатформенный доступ к стандартным директориям
- C) `file_system` пакет
- D) `flutter/src/foundation/paths.dart`

<details>
<summary>Ответ</summary>

**B) `path_provider`**

Основные методы:

- `getApplicationDocumentsDirectory()` — для настроек/данных пользователя (backup на iOS)
- `getApplicationSupportDirectory()` — для внутренних данных приложения
- `getTemporaryDirectory()` — Cache, может быть удалён ОС
- `getExternalStorageDirectory()` — только Android (SD card, публичные файлы)

</details>

---

### Вопрос 2 🟢

Как записать текст в файл и прочитать его обратно?

- A) `File(path).write(text)` / `File(path).read()`
- B) `await File(path).writeAsString(text)` / `await File(path).readAsString()`
- C) `FileOutput(path).write(text)` / `FileInput(path).read()`
- D) `dart:io.save(path, text)` / `dart:io.load(path)`

<details>
<summary>Ответ</summary>

**B) `writeAsString()` / `readAsString()`**

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

final dir = await getApplicationDocumentsDirectory();
final file = File(p.join(dir.path, 'notes.txt'));

await file.writeAsString('Hello, Flutter!');
final content = await file.readAsString();
```

</details>

---

### Вопрос 3 🟢

Какой пакет позволяет пользователю выбрать изображение из галереи или сделать фото?

- A) `camera`
- B) `image_picker`
- C) `gallery_saver`
- D) `photo_manager`

<details>
<summary>Ответ</summary>

**B) `image_picker`**

```dart
final picker = ImagePicker();
final image = await picker.pickImage(
  source: ImageSource.gallery, // или ImageSource.camera
  maxWidth: 1920,
  imageQuality: 85,
);
if (image != null) {
  final file = File(image.path);
}
```

Требует разрешений: iOS `NSPhotoLibraryUsageDescription`, Android `READ_EXTERNAL_STORAGE`.

</details>

---

### Вопрос 4 🟡

Какое различие между `writeAsBytes(bytes)` и `openWrite()` для записи файлов?

- A) Никакого — они идентичны
- B) `writeAsBytes` заменяет файл целиком; `openWrite()` возвращает `IOSink` для потоковой записи (streaming) больших файлов
- C) `openWrite()` только для бинарных данных
- D) `writeAsBytes` только для текста

<details>
<summary>Ответ</summary>

**B) `writeAsBytes` — заменяет; `openWrite()`/`IOSink` — потоковая запись**

```dart
// Маленький файл — сразу весь:
await file.writeAsBytes(imageBytes);

// Большой файл — потоком (эффективно по памяти):
final sink = file.openWrite();
await for (final chunk in stream) {
  sink.add(chunk);
}
await sink.close();

// Дописать в конец:
final sink = file.openWrite(mode: FileMode.append);
sink.writeln('New line');
await sink.close();
```

</details>

---

### Вопрос 5 🟡

Как проверить существование файла и создать директорию рекурсивно?

- A) `File.exists(path)` / `Dir.make(path)`
- B) `await file.exists()` / `await directory.create(recursive: true)`
- C) `path_provider.check(path)` / `path_provider.mkdir(path)`
- D) `File.stat(path) != null` / `Directory.create(path)`

<details>
<summary>Ответ</summary>

**B) `file.exists()` / `directory.create(recursive: true)`**

```dart
import 'dart:io';

final dir = Directory('/storage/data/subfolder');
if (!await dir.exists()) {
  await dir.create(recursive: true); // создаст все промежуточные папки
}

final file = File('${dir.path}/config.json');
if (await file.exists()) {
  final content = await file.readAsString();
}
```

</details>

---

### Вопрос 6 🟡

Что делает `file_picker` пакет в отличие от `image_picker`?

- A) Они идентичны
- B) `file_picker` позволяет выбирать любые типы файлов (PDF, документы, любые расширения), а не только изображения
- C) `file_picker` только для Desktop
- D) `image_picker` — для файлов, `file_picker` — для изображений

<details>
<summary>Ответ</summary>

**B) `file_picker` — для любых типов файлов**

```dart
final result = await FilePicker.platform.pickFiles(
  type: FileType.custom,
  allowedExtensions: ['pdf', 'doc', 'docx'],
  allowMultiple: true,
);
if (result != null) {
  for (final file in result.files) {
    print('${file.name} — ${file.size} bytes');
    final bytes = file.bytes; // Web
    final path = file.path;   // Mobile/Desktop
  }
}
```

</details>

---

### Вопрос 7 🟡

Как получить список файлов в директории?

- A) `Directory(path).list()` — возвращает Stream<FileSystemEntity>
- B) `Directory(path).files` — List<File>
- C) `path_provider.listDir(path)`
- D) `File.list(directory)`

<details>
<summary>Ответ</summary>

**A) `Directory(path).list()` → `Stream<FileSystemEntity>`**

```dart
final dir = await getApplicationDocumentsDirectory();
final entities = dir.list(recursive: false, followLinks: false);

await for (final entity in entities) {
  if (entity is File) {
    print('File: ${entity.path}');
  } else if (entity is Directory) {
    print('Dir: ${entity.path}');
  }
}

// Синхронно (блокирует поток!):
final list = dir.listSync();
```

</details>

---

### Вопрос 8 🔴

Как сохранить файл в публичную галерею на Android и iOS?

- A) `File.copy('/gallery/${file.path}')`
- B) Использовать `gal` или `image_gallery_saver` пакет — для галереи нужен специальный API каждой платформы
- C) `MediaStore.save(file)` через `dart:io`
- D) `path_provider.getGalleryDirectory()`

<details>
<summary>Ответ</summary>

**B) `gal` пакет (современный) или `image_gallery_saver`**

```dart
import 'package:gal/gal.dart';

// Сохранить фото в галерею:
await Gal.putImage(filePath);

// Сохранить видео:
await Gal.putVideo(videoPath);

// Проверить разрешения:
final hasAccess = await Gal.hasAccess();
if (!hasAccess) {
  await Gal.requestAccess();
}
```

Требует разрешений: Android `WRITE_EXTERNAL_STORAGE` (API<29), iOS `NSPhotoLibraryAddUsageDescription`.

</details>

---

### Вопрос 9 🔴

Как безопасно работать с файлами, чтобы избежать path traversal атак?

- A) Доверять всем путям от `path_provider`
- B) Валидировать и нормализовать пути, убедиться что финальный путь начинается с разрешённой базовой директории
- C) Использовать `File.secure(path)`
- D) Path traversal невозможен в Flutter

<details>
<summary>Ответ</summary>

**B) Нормализовать и проверять что путь — внутри разрешённой директории**

```dart
import 'package:path/path.dart' as p;

Future<File> getSafeFile(String userFileName) async {
  final docsDir = await getApplicationDocumentsDirectory();

  // Нормализовать путь (убрать ../ и т.п.):
  final normalizedName = p.basename(userFileName); // только имя файла!
  final safePath = p.join(docsDir.path, normalizedName);

  // Убедиться что путь внутри разрешённой директории:
  if (!safePath.startsWith(docsDir.path)) {
    throw Exception('Path traversal detected!');
  }

  return File(safePath);
}
```

</details>

---

### Вопрос 10 🔴

Как эффективно скопировать большой файл, не загружая весь в память?

- A) `final bytes = await src.readAsBytes(); await dst.writeAsBytes(bytes);`
- B) `await src.copy(dst.path)` — OS-level copy или `src.openRead().pipe(dst.openWrite())`
- C) `File.transfer(src, dst)`
- D) Разбить на чанки вручную через `readAsBytes()`

<details>
<summary>Ответ</summary>

**B) `src.copy(dst)` — самый эффективный или `pipe()` для трансформаций**

```dart
// Простое копирование (Platform OS handles it):
final newFile = await srcFile.copy(destinationPath);

// Копирование с трансформацией (стриминг, без загрузки в память):
final srcStream = srcFile.openRead();
final dstSink = dstFile.openWrite();
await srcStream.pipe(dstSink); // dstSink автоматически закрывается

// Копирование с прогрессом:
final total = await srcFile.length();
int written = 0;
await srcFile.openRead().forEach((chunk) async {
  dstSink.add(chunk);
  written += chunk.length;
  onProgress(written / total);
});
await dstSink.close();
```

</details>
