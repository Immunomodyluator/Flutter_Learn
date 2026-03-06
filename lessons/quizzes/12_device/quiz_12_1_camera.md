# Квиз: Камера и медиа

**Тема:** 12.1 — Camera / Image Picker  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как инициализировать `CameraController` для использования камеры?

- A) `Camera.start()`
- B) `availableCameras()` → выбрать камеру → `CameraController(camera, resolution)` → `await controller.initialize()`
- C) `CameraWidget.init()`
- D) Камера доступна сразу через `Image.camera()`

<details>
<summary>Ответ</summary>

**B) `availableCameras()` → `CameraController` → `initialize()`**

```dart
class _CameraState extends State<CameraScreen> {
  late CameraController _controller;

  @override
  void initState() async {
    super.initState();
    final cameras = await availableCameras();
    final frontCamera = cameras.firstWhere(
      (c) => c.lensDirection == CameraLensDirection.front,
      orElse: () => cameras.first,
    );

    _controller = CameraController(
      frontCamera,
      ResolutionPreset.high,
      enableAudio: false,
    );
    await _controller.initialize();
    if (mounted) setState(() {});
  }

  @override
  void dispose() {
    _controller.dispose(); // ОБЯЗАТЕЛЬНО
    super.dispose();
  }
}
```

</details>

---

### Вопрос 2 🟢

Как сделать фото с камеры?

- A) `_controller.capture()`
- B) `final XFile photo = await _controller.takePicture()`
- C) `_controller.snap().then((file) => ...)`
- D) `CameraCapture.take(_controller)`

<details>
<summary>Ответ</summary>

**B) `await _controller.takePicture()` → `XFile`**

```dart
Future<void> _takePicture() async {
  if (!_controller.value.isInitialized) return;
  if (_controller.value.isTakingPicture) return;

  try {
    final XFile photo = await _controller.takePicture();
    final File imageFile = File(photo.path);

    // Показать превью:
    Navigator.push(ctx, MaterialPageRoute(
      builder: (_) => PreviewScreen(imagePath: photo.path),
    ));
  } on CameraException catch (e) {
    print('Camera error: ${e.code}: ${e.description}');
  }
}
```

</details>

---

### Вопрос 3 🟢

Как открыть галерею для выбора изображения через `image_picker`?

- A) `ImagePicker.gallery()`
- B) `final XFile? image = await ImagePicker().pickImage(source: ImageSource.gallery)`
- C) `Gallery.pickImage()`
- D) `File.fromGallery()`

<details>
<summary>Ответ</summary>

**B) `ImagePicker().pickImage(source: ImageSource.gallery)`**

```dart
final picker = ImagePicker();

// Одно изображение:
final XFile? image = await picker.pickImage(
  source: ImageSource.gallery,
  maxWidth: 1920,
  maxHeight: 1080,
  imageQuality: 85, // 0-100, сжатие JPEG
);

if (image != null) {
  final file = File(image.path);
  setState(() => _selectedImage = file);
}

// Несколько изображений:
final List<XFile> images = await picker.pickMultiImage();

// Видео:
final XFile? video = await picker.pickVideo(source: ImageSource.gallery);
```

</details>

---

### Вопрос 4 🟡

Как показать preview камеры в UI?

- A) `Camera.preview()`
- B) `_controller.value.isInitialized ? CameraPreview(_controller) : CircularProgressIndicator()`
- C) `Image.camera()`
- D) `VideoPlayer(_controller.stream)`

<details>
<summary>Ответ</summary>

**B) `CameraPreview(_controller)` после инициализации**

```dart
@override
Widget build(BuildContext ctx) {
  if (!_controller.value.isInitialized) {
    return const CircularProgressIndicator();
  }

  return Scaffold(
    body: Stack(
      fit: StackFit.expand,
      children: [
        CameraPreview(_controller), // превью камеры
        Align(
          alignment: Alignment.bottomCenter,
          child: Padding(
            padding: const EdgeInsets.only(bottom: 32),
            child: FloatingActionButton(
              onPressed: _takePicture,
              child: const Icon(Icons.camera),
            ),
          ),
        ),
      ],
    ),
  );
}
```

</details>

---

### Вопрос 5 🟡

Какие разрешения нужно запросить для камеры и галереи?

- A) Никаких — камера работает без разрешений
- B) Android: `CAMERA`, `READ_MEDIA_IMAGES` (API 33+); iOS: `NSCameraUsageDescription`, `NSPhotoLibraryUsageDescription` в Info.plist
- C) Только `CAMERA` для обоих платформ
- D) Разрешения автоматически запрашиваются пакетом `camera`

<details>
<summary>Ответ</summary>

**B) Разные разрешения для Android и iOS**

```xml
<!-- AndroidManifest.xml: -->
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/> <!-- API 33+ -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
                 android:maxSdkVersion="32"/> <!-- API <33 -->
```

```xml
<!-- iOS Info.plist: -->
<key>NSCameraUsageDescription</key>
<string>Нужен доступ к камере для съёмки фото</string>
<key>NSPhotoLibraryUsageDescription</key>
<string>Нужен доступ к галерее для выбора фото</string>
<key>NSMicrophoneUsageDescription</key>
<string>Нужен микрофон для записи видео</string>
```

```dart
// permission_handler:
await Permission.camera.request();
```

</details>

---

### Вопрос 6 🟡

Как записать видео с камеры?

- A) `_controller.captureVideo()`
- B) `await _controller.startVideoRecording()` → `await _controller.stopVideoRecording()` → `XFile`
- C) `VideoRecorder.start(_controller)`
- D) Видео поддерживается только через `image_picker`

<details>
<summary>Ответ</summary>

**B) `startVideoRecording()` / `stopVideoRecording()`**

```dart
bool _isRecording = false;

Future<void> _toggleRecording() async {
  if (_isRecording) {
    final XFile video = await _controller.stopVideoRecording();
    setState(() => _isRecording = false);

    final file = File(video.path);
    print('Video saved: ${file.path} (${await file.length()} bytes)');
  } else {
    await _controller.prepareForVideoRecording(); // iOS требует
    await _controller.startVideoRecording();
    setState(() => _isRecording = true);
  }
}
```

Требует `enableAudio: true` в `CameraController` для звука.

</details>

---

### Вопрос 7 🟡

Как обрезать изображение после выбора из галереи?

- A) `image_picker` поддерживает crop встроенно
- B) Использовать `image_cropper` или `croppr_flutter` пакеты после получения `XFile` из image_picker
- C) `File.crop(rect)`
- D) `dart:ui.Image.crop()`

<details>
<summary>Ответ</summary>

**B) `image_cropper` — отдельный пакет для кроппинга**

```dart
Future<File?> _pickAndCropImage() async {
  // 1. Выбрать:
  final XFile? picked = await ImagePicker().pickImage(source: ImageSource.gallery);
  if (picked == null) return null;

  // 2. Обрезать:
  final CroppedFile? cropped = await ImageCropper().cropImage(
    sourcePath: picked.path,
    uiSettings: [
      AndroidUiSettings(
        toolbarTitle: 'Обрезка',
        aspectRatioPresets: [CropAspectRatioPreset.square, CropAspectRatioPreset.ratio16x9],
      ),
      IOSUiSettings(title: 'Обрезка'),
    ],
  );

  return cropped != null ? File(cropped.path) : null;
}
```

</details>

---

### Вопрос 8 🔴

Как реализовать сканер QR-кода с камерой?

- A) `CameraController.scanQR()`
- B) `mobile_scanner` пакет — `MobileScannerController` + `MobileScanner(onDetect: callback)`
- C) `qr_code_scanner` с `QRView` виджетом
- D) `FirebaseVision.barcodeDetector()`

<details>
<summary>Ответ</summary>

**B) `mobile_scanner` — современный рекомендуемый пакет**

```dart
final _scannerController = MobileScannerController(
  detectionSpeed: DetectionSpeed.noDuplicates,
  facing: CameraFacing.back,
);

MobileScanner(
  controller: _scannerController,
  onDetect: (capture) {
    final barcodes = capture.barcodes;
    for (final barcode in barcodes) {
      print('Found barcode: ${barcode.rawValue}');
      if (barcode.type == BarcodeType.url) {
        print('URL: ${barcode.url?.url}');
      }
    }
  },
)

// Переключить фонарик:
_scannerController.toggleTorch();
// Переключить камеру:
_scannerController.switchCamera();
```

</details>

---

### Вопрос 9 🔴

Как обработать `CameraException` с корректным показом ошибки пользователю?

- A) Игнорировать — камера всегда доступна
- B) `CameraException` имеет `code` и `description`; основные коды: `cameraPermission`, `audioAccessDenied`, `CameraAccessDenied`
- C) Все ошибки имеют одинаковый код
- D) `try-catch (Exception)` достаточно

<details>
<summary>Ответ</summary>

**B) `CameraException.code` для разных типов ошибок**

```dart
Future<void> _initCamera() async {
  try {
    await _controller.initialize();
  } on CameraException catch (e) {
    String message;
    switch (e.code) {
      case 'CameraAccessDenied':
      case 'cameraPermission':
        message = 'Нет разрешения на камеру. Откройте настройки.';
        break;
      case 'AudioAccessDenied':
        message = 'Нет доступа к микрофону.';
        break;
      default:
        message = 'Ошибка камеры: ${e.description}';
    }
    ScaffoldMessenger.of(ctx).showSnackBar(SnackBar(content: Text(message)));

    // Открыть настройки:
    await openAppSettings(); // permission_handler
  }
}
```

</details>

---

### Вопрос 10 🔴

Как реализовать потоковый анализ кадров с камеры (image stream)?

- A) `CameraController.frames.listen()`
- B) `_controller.startImageStream((CameraImage image) { ... })` — получать сырые кадры `YUV420` / `BGRA8888`
- C) `CameraPreview.onFrame`
- D) Только `takePicture()` — потоков нет

<details>
<summary>Ответ</summary>

**B) `startImageStream()` — потоковые сырые кадры**

```dart
@override
void initState() async {
  // ...
  await _controller.initialize();
  await _controller.startImageStream(_processCameraImage);
}

void _processCameraImage(CameraImage image) {
  // image.format.group: ImageFormatGroup.yuv420 (Android) / bgra8888 (iOS)
  // image.planes — цветовые плоскости

  // Обработку выполнять в Isolate (чтобы не блокировать UI):
  compute(_analyzeFrame, image).then((result) {
    if (mounted) setState(() => _detectionResult = result);
  });
}

// При переходе на другой экран:
@override
void dispose() {
  _controller.stopImageStream();
  _controller.dispose();
  super.dispose();
}
```

</details>
