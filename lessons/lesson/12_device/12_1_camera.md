# 12.1 Камера и галерея

## 1. Суть

Flutter предоставляет два подхода:

- **`image_picker`** — простой выбор фото/видео из галереи или камеры (рекомендуется для большинства задач)
- **`camera`** — полный контроль над камерой с предпросмотром в реальном времени

```yaml
dependencies:
  image_picker: ^1.0.7
  camera: ^0.10.5+9 # только если нужен кастомный UI камеры
```

---

## 2. image_picker — выбор фото/видео

```dart
import 'package:image_picker/image_picker.dart';
import 'dart:io';

final picker = ImagePicker();

// Выбор фото из галереи
final XFile? photo = await picker.pickImage(
  source: ImageSource.gallery,
  maxWidth: 1024,      // ограничение размера
  maxHeight: 1024,
  imageQuality: 85,    // сжатие (0-100)
);

if (photo != null) {
  final file = File(photo.path);
  // использовать файл
}

// Съёмка через камеру
final XFile? captured = await picker.pickImage(source: ImageSource.camera);

// Выбор видео
final XFile? video = await picker.pickVideo(
  source: ImageSource.gallery,
  maxDuration: const Duration(minutes: 5),
);

// Несколько фото (iOS 14+, Android)
final List<XFile> photos = await picker.pickMultiImage(
  maxWidth: 1024,
  imageQuality: 85,
);
```

---

## 3. Настройка манифестов

**Android** (`AndroidManifest.xml`):

```xml
<uses-permission android:name="android.permission.CAMERA"/>
<uses-feature android:name="android.hardware.camera" android:required="false"/>
```

**iOS** (`Info.plist`):

```xml
<key>NSCameraUsageDescription</key>
<string>Для съёмки фото профиля</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Для выбора фото из галереи</string>
<key>NSMicrophoneUsageDescription</key>
<string>Для записи видео со звуком</string>
```

---

## 4. Реальный пример — пикер с обрезкой

```dart
import 'package:image_picker/image_picker.dart';
import 'package:image_cropper/image_cropper.dart'; // дополнительный пакет

class ImagePickerService {
  final _picker = ImagePicker();

  Future<File?> pickAndCropAvatar() async {
    // 1. Выбрать источник
    final source = await _showSourceDialog();
    if (source == null) return null;

    // 2. Получить фото
    final photo = await _picker.pickImage(
      source: source,
      maxWidth: 2048,
      maxHeight: 2048,
      imageQuality: 90,
    );
    if (photo == null) return null;

    // 3. Обрезать по кругу
    final cropped = await ImageCropper().cropImage(
      sourcePath: photo.path,
      aspectRatio: const CropAspectRatio(ratioX: 1, ratioY: 1),
      uiSettings: [
        AndroidUiSettings(
          toolbarTitle: 'Обрезать фото',
          lockAspectRatio: true,
        ),
        IOSUiSettings(
          title: 'Обрезать фото',
          aspectRatioLockEnabled: true,
        ),
      ],
    );

    return cropped != null ? File(cropped.path) : null;
  }

  Future<ImageSource?> _showSourceDialog() async {
    // Показывает bottom sheet с выбором: камера / галерея
    return null; // реализация через showModalBottomSheet
  }
}

// --- Виджет ---
class AvatarPicker extends StatefulWidget {
  const AvatarPicker({super.key});

  @override
  State<AvatarPicker> createState() => _AvatarPickerState();
}

class _AvatarPickerState extends State<AvatarPicker> {
  File? _imageFile;
  final _service = ImagePickerService();

  Future<void> _pick() async {
    final file = await _service.pickAndCropAvatar();
    if (file != null && mounted) {
      setState(() => _imageFile = file);
    }
  }

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: _pick,
      child: CircleAvatar(
        radius: 60,
        backgroundImage: _imageFile != null ? FileImage(_imageFile!) : null,
        child: _imageFile == null
            ? const Icon(Icons.add_a_photo, size: 40)
            : null,
      ),
    );
  }
}
```

---

## 5. camera — кастомный UI камеры

```dart
import 'package:camera/camera.dart';

class CameraScreen extends StatefulWidget {
  const CameraScreen({super.key});

  @override
  State<CameraScreen> createState() => _CameraScreenState();
}

class _CameraScreenState extends State<CameraScreen> {
  late final CameraController _controller;
  late final Future<void> _initFuture;

  @override
  void initState() {
    super.initState();
    _initFuture = _initCamera();
  }

  Future<void> _initCamera() async {
    final cameras = await availableCameras();
    _controller = CameraController(
      cameras.first,
      ResolutionPreset.high,
      enableAudio: false,
    );
    await _controller.initialize();
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  Future<void> _takePicture() async {
    if (!_controller.value.isInitialized) return;

    final image = await _controller.takePicture();
    if (mounted) {
      // использовать image.path
    }
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<void>(
      future: _initFuture,
      builder: (context, snapshot) {
        if (snapshot.connectionState != ConnectionState.done) {
          return const Center(child: CircularProgressIndicator());
        }
        return Stack(
          children: [
            CameraPreview(_controller),
            Positioned(
              bottom: 32,
              left: 0,
              right: 0,
              child: IconButton(
                iconSize: 72,
                icon: const Icon(Icons.camera, color: Colors.white),
                onPressed: _takePicture,
              ),
            ),
          ],
        );
      },
    );
  }
}
```

---

## 6. Под капотом

- `image_picker` использует нативный Intent (Android) / UIImagePickerController (iOS).
- `XFile.path` — путь к временному файлу в кеше приложения (не в галерее).
- `camera` использует Camera2 API (Android) / AVFoundation (iOS).
- Фото/видео из галереи доступны только для чтения — скопируй в документы если нужно редактировать.

---

## 7. Типичные ошибки

| Ошибка                         | Причина                       | Решение                                    |
| ------------------------------ | ----------------------------- | ------------------------------------------ |
| `MissingPluginException`       | Не добавлены разрешения       | Добавь в AndroidManifest и Info.plist      |
| Фото null после выбора         | Пользователь отменил          | Всегда проверяй `photo != null`            |
| Большой файл                   | Нет сжатия                    | Используй `maxWidth`/`imageQuality`        |
| Камера не работает в эмуляторе | Виртуальная камера ограничена | Тестируй на реальном устройстве            |
| `CameraController` не dispose  | Утечка ресурса                | Всегда `_controller.dispose()` в dispose() |

---

## 8. Рекомендации

1. **`image_picker`** для большинства задач — не пиши кастомный UI камеры без необходимости.
2. **Сжимай фото** перед загрузкой на сервер — `maxWidth: 1024`, `imageQuality: 85`.
3. **Bottom sheet** для выбора источника — лучший UX, чем два отдельных виджета.
4. **Обрезка** через `image_cropper` — не реализуй canvas-обрезку самостоятельно.
5. **Тестируй на реальном устройстве** — камера в эмуляторе ненадёжна.
