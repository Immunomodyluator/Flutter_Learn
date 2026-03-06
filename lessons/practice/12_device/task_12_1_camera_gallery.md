# 12.1: Камера и галерея — фото блюда

> Project: FitMenu | Глава 12 — Device Features

### 12.1: ImagePickerService для съёмки и выбора фото блюда

🎯 **Цель шага:** Добавить возможность фотографировать блюдо или выбирать фото из галереи при добавлении приёма пищи — через `image_picker` с обработкой разрешений.

---

📝 **Техническое задание:**

**Зависимость:** `image_picker: ^1.0.0`

Реализуй `lib/features/meal_log/services/image_picker_service.dart`.

**ImagePickerService:**
```dart
class ImagePickerService {
  final _picker = ImagePicker();

  Future<File?> pickFromGallery() async { ... }
  Future<File?> takePhoto() async { ... }
  Future<File?> showPickerDialog(BuildContext context) async { ... }
}
```

- `maxWidth: 1024`, `maxHeight: 1024`, `imageQuality: 85`
- `showPickerDialog`: показывает `showModalBottomSheet` с вариантами "Камера" / "Галерея"
- Обработка `null` (пользователь отменил)

**UI виджет выбора фото:**
```dart
class FoodPhotoPickerWidget extends StatefulWidget {
  // Отображает: пустой контейнер с иконкой камеры → после выбора показывает Image.file
  // Нажатие → showPickerDialog
}
```

---

✅ **Критерии приёмки:**
- [ ] Оба источника (камера/галерея) работают
- [ ] `imageQuality: 85` применяется для сжатия
- [ ] `null` обрабатывается (нет крашей при отмене)
- [ ] `showModalBottomSheet` предлагает выбор источника
- [ ] `Image.file` отображает выбранное фото
- [ ] `CircleAvatar` или `ClipRRect` скругляет превью

---

💡 **Подсказка:** `XFile.path` даёт путь к временному файлу — оберни в `File(xfile.path)`. На iOS нужно добавить NSCameraUsageDescription в Info.plist. `ImageSource.camera` / `ImageSource.gallery` — два источника.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/meal_log/services/image_picker_service.dart

import 'dart:io';
import 'package:flutter/material.dart';
import 'package:image_picker/image_picker.dart';

class ImagePickerService {
  final _picker = ImagePicker();

  Future<File?> _pick(ImageSource source) async {
    final xfile = await _picker.pickImage(
      source: source,
      maxWidth: 1024,
      maxHeight: 1024,
      imageQuality: 85,
    );
    return xfile != null ? File(xfile.path) : null;
  }

  Future<File?> pickFromGallery() => _pick(ImageSource.gallery);
  Future<File?> takePhoto()       => _pick(ImageSource.camera);

  Future<File?> showPickerDialog(BuildContext context) async {
    ImageSource? source;
    await showModalBottomSheet<void>(
      context: context,
      builder: (_) => SafeArea(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            ListTile(
              leading: const Icon(Icons.photo_library),
              title: const Text('Выбрать из галереи'),
              onTap: () { source = ImageSource.gallery; Navigator.pop(context); },
            ),
            ListTile(
              leading: const Icon(Icons.camera_alt),
              title: const Text('Сделать фото'),
              onTap: () { source = ImageSource.camera; Navigator.pop(context); },
            ),
          ],
        ),
      ),
    );
    if (source == null) return null;
    return _pick(source!);
  }
}

// Виджет
class FoodPhotoPickerWidget extends StatefulWidget {
  const FoodPhotoPickerWidget({super.key, this.onPhotoSelected});
  final void Function(File)? onPhotoSelected;

  @override
  State<FoodPhotoPickerWidget> createState() => _FoodPhotoPickerWidgetState();
}

class _FoodPhotoPickerWidgetState extends State<FoodPhotoPickerWidget> {
  File? _photo;
  final _service = ImagePickerService();

  Future<void> _pick() async {
    final file = await _service.showPickerDialog(context);
    if (file != null) {
      setState(() => _photo = file);
      widget.onPhotoSelected?.call(file);
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _pick,
      child: ClipRRect(
        borderRadius: BorderRadius.circular(12),
        child: Container(
          width: 100, height: 100,
          color: Theme.of(context).colorScheme.surfaceVariant,
          child: _photo != null
              ? Image.file(_photo!, fit: BoxFit.cover)
              : const Icon(Icons.add_a_photo, size: 32),
        ),
      ),
    );
  }
}
```

</details>
