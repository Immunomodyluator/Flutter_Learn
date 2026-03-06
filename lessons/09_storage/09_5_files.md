# 09.5 Работа с файлами

## 1. Суть

Flutter предоставляет доступ к файловой системе через `dart:io` + `path_provider`. Используется для: загрузки файлов, сохранения PDF/изображений, кеширования, экспорта данных.

```yaml
# pubspec.yaml
dependencies:
  path_provider: ^2.1.2 # пути к папкам системы
```

---

## 2. Директории path_provider

```dart
import 'package:path_provider/path_provider.dart';
import 'dart:io';

// Документы приложения — не очищаются системой (iOS/Android)
final docsDir = await getApplicationDocumentsDirectory();
// → /data/user/0/com.example/files (Android)
// → /Documents (iOS)

// Временные файлы — могут быть удалены системой
final tempDir = await getTemporaryDirectory();
// → /data/user/0/com.example/cache

// Внешнее хранилище Android (SD-карта / Downloads)
final extDir = await getExternalStorageDirectory(); // только Android

// Поддержка (iOS) — синхронизируется iCloud
final supportDir = await getApplicationSupportDirectory();
```

---

## 3. Чтение и запись файлов

```dart
import 'dart:io';
import 'package:path_provider/path_provider.dart';
import 'package:path/path.dart' as p;

// Запись строки
Future<File> writeTextFile(String filename, String content) async {
  final dir = await getApplicationDocumentsDirectory();
  final file = File(p.join(dir.path, filename));
  return file.writeAsString(content);
}

// Чтение строки
Future<String> readTextFile(String filename) async {
  final dir = await getApplicationDocumentsDirectory();
  final file = File(p.join(dir.path, filename));
  if (!await file.exists()) return '';
  return file.readAsString();
}

// Запись байт (изображения, PDF)
Future<File> writeBytesFile(String filename, Uint8List bytes) async {
  final dir = await getApplicationDocumentsDirectory();
  final file = File(p.join(dir.path, filename));
  return file.writeAsBytes(bytes);
}

// Удаление
Future<void> deleteFile(String filename) async {
  final dir = await getApplicationDocumentsDirectory();
  final file = File(p.join(dir.path, filename));
  if (await file.exists()) await file.delete();
}

// Список файлов в директории
Future<List<FileSystemEntity>> listFiles(String subdir) async {
  final dir = await getApplicationDocumentsDirectory();
  final folder = Directory(p.join(dir.path, subdir));
  if (!await folder.exists()) return [];
  return folder.listSync().toList();
}
```

---

## 4. Реальный пример — кеш изображений вручную

```dart
class ImageCacheService {
  Future<File?> getCachedImage(String url) async {
    final file = await _getCacheFile(url);
    if (await file.exists()) return file;
    return null;
  }

  Future<File> downloadAndCache(String url, Uint8List bytes) async {
    final file = await _getCacheFile(url);
    await file.parent.create(recursive: true);
    return file.writeAsBytes(bytes);
  }

  Future<void> clearCache() async {
    final dir = await getTemporaryDirectory();
    final cacheDir = Directory('${dir.path}/img_cache');
    if (await cacheDir.exists()) await cacheDir.delete(recursive: true);
  }

  Future<int> getCacheSizeBytes() async {
    final dir = await getTemporaryDirectory();
    final cacheDir = Directory('${dir.path}/img_cache');
    if (!await cacheDir.exists()) return 0;

    int total = 0;
    await for (final entity in cacheDir.list(recursive: true)) {
      if (entity is File) total += await entity.length();
    }
    return total;
  }

  Future<File> _getCacheFile(String url) async {
    final dir = await getTemporaryDirectory();
    // Хеш URL как имя файла
    final hash = url.hashCode.abs().toString();
    final ext = p.extension(Uri.parse(url).path);
    return File('${dir.path}/img_cache/$hash$ext');
  }
}
```

---

## 5. Экспорт файла (Share/Save)

```dart
import 'package:share_plus/share_plus.dart';

// Поделиться файлом
Future<void> shareFile(File file) async {
  await Share.shareXFiles(
    [XFile(file.path)],
    text: 'Экспорт данных',
  );
}

// Сохранить в галерею (Android/iOS)
// пакет: image_gallery_saver или gal
```

---

## 6. Потоковая запись больших файлов

```dart
// Чтение большого файла построчно (не грузит всё в память)
Future<void> processLargeFile(String path) async {
  final file = File(path);
  final lines = file.openRead().transform(utf8.decoder).transform(const LineSplitter());

  await for (final line in lines) {
    // обрабатываем строку
    print(line);
  }
}

// Запись большого файла через IOSink
Future<void> writeLargeFile(String path, List<String> lines) async {
  final file = File(path);
  final sink = file.openWrite();

  for (final line in lines) {
    sink.writeln(line);
  }
  await sink.close();
}
```

---

## 7. Под капотом

- `dart:io File` — обёртка над системными вызовами OS.
- `path_provider` — дёргает platform channel для получения путей (документы, кеш, external).
- `writeAsString`/`writeAsBytes` — синхронно с `async` обёрткой через `dart:io` event loop.
- iOS имеет строгую «sandbox» модель — нет прямого доступа к чужим файлам.

---

## 8. Типичные ошибки

| Ошибка                                   | Причина                             | Решение                                                         |
| ---------------------------------------- | ----------------------------------- | --------------------------------------------------------------- |
| `FileSystemException: Permission denied` | Нет разрешения на Android           | Запрашивай `permission_handler`                                 |
| `PathNotFoundException`                  | Папка не создана                    | `await file.parent.create(recursive: true)` перед записью       |
| Большой файл в OOM                       | `readAsBytes()` для 100MB файла     | Используй `openRead()` + stream                                 |
| Файл пропал после обновления             | `getTemporaryDirectory()` очищается | Для постоянного хранения → `getApplicationDocumentsDirectory()` |
| Не работает на Web                       | `dart:io` недоступен во Flutter Web | Используй `universal_io` или `js`-подход                        |

---

## 9. Рекомендации

1. **Временный кеш** → `getTemporaryDirectory()`, постоянные данные → `getApplicationDocumentsDirectory()`.
2. **`path.join()`** — всегда для построения путей, не конкатенация строк.
3. **Проверяй `exists()`** перед чтением файла — особенно в кеше.
4. **Большие файлы** читай потоком (`openRead()`), а не через `readAsBytes()`.
5. **Запрашивай разрешения** через `permission_handler` на Android для внешнего хранилища.
6. **Web** не поддерживает `dart:io` — используй `dart:html` или условный импорт.
