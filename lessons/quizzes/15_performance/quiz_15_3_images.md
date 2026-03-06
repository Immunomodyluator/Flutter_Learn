# Квиз: Image Optimization

**Тема:** 15.3 — Image Performance  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Почему `cached_network_image` лучше `Image.network`?

- A) `cached_network_image` поддерживает GIF
- B) Автоматически кэширует изображения на диск и в памяти — не загружает повторно при скролле/rebuild
- C) `Image.network` не поддерживает HTTPS
- D) `cached_network_image` быстрее скачивает

<details>
<summary>Ответ</summary>

**B) Кэширование на диск и в память**

```dart
// Image.network — каждый раз загружает с сети:
Image.network('https://example.com/photo.jpg')

// cached_network_image — disk + memory cache:
CachedNetworkImage(
  imageUrl: 'https://example.com/photo.jpg',
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(Icons.error),
  // Дополнительные параметры:
  fadeInDuration: const Duration(milliseconds: 300),
  memCacheWidth: 200,  // ограничить размер в memory cache
  maxWidthDiskCache: 800,
)

// Предзагрузка:
await precachePicture(
  CachedNetworkImageProvider('https://example.com/next.jpg'),
  context,
);
```

</details>

---

### Вопрос 2 🟢

Что такое `precacheImage` и когда его использовать?

- A) `precacheImage` — сжатие изображений перед отображением
- B) Загружает изображение в image cache заранее, до того как виджет его запросит — убирает задержку первого отображения
- C) `precacheImage` — только для локальных asset'ов
- D) `precacheImage` блокирует UI thread

<details>
<summary>Ответ</summary>

**B) Предварительная загрузка в кэш**

```dart
// В initState или при навигации на предыдущий экран:
@override
void initState() {
  super.initState();
  WidgetsBinding.instance.addPostFrameCallback((_) {
    // Предзагрузить следующие изображения:
    for (final product in products.take(10)) {
      precacheImage(
        CachedNetworkImageProvider(product.imageUrl),
        context,
      );
    }
  });
}

// Перед переходом на детальный экран:
Future<void> _openProduct(Product product) async {
  await precacheImage(
    CachedNetworkImageProvider(product.largeImageUrl),
    context,
  );
  router.push('/product/${product.id}');
}

// Для asset изображений:
await precacheImage(const AssetImage('assets/images/hero.jpg'), context);
```

</details>

---

### Вопрос 3 🟢

Как формат WebP помогает производительности Flutter?

- A) WebP поддерживается только на Android
- B) WebP даёт ~25–35% меньший размер файла по сравнению с JPEG/PNG при том же качестве — меньше трафика, быстрее загрузка
- C) WebP единственный формат с поддержкой анимации
- D) WebP не поддерживается во Flutter

<details>
<summary>Ответ</summary>

**B) Меньший размер при том же качестве**

```
Сравнение форматов для одного изображения 1000×1000:
- PNG: ~500 KB (без потерь)
- JPEG: ~80 KB (с потерями, без прозрачности)
- WebP lossless: ~350 KB
- WebP lossy: ~55 KB ← оптимальный выбор
- AVIF: ~40 KB (новый, не везде поддерживается)

Flutter поддерживает WebP на всех платформах.
```

```bash
# Конвертировать изображения в WebP:
cwebp -q 80 image.png -o image.webp

# В CI — автоматическая конвертация:
find assets/images -name "*.jpg" -exec \
  cwebp -q 80 {} -o {}.webp \;
```

Для иконок используйте SVG через `flutter_svg` или векторные assets.

</details>

---

### Вопрос 4 🟡

Как `cacheWidth` и `cacheHeight` влияют на использование памяти?

- A) Это размер disk cache
- B) Ограничивают декодированный размер изображения в памяти — 4000×3000 px JPEG занимает 46 MB, с `cacheWidth: 400` только 460 KB
- C) `cacheWidth`/`cacheHeight` масштабируют изображение на экране
- D) Работают только с `Image.asset`

<details>
<summary>Ответ</summary>

**B) Ограничение decoded размера в RAM**

```dart
// Изображение 4000×3000 px = 4000*3000*4 bytes = ~46 MB в RAM!

// Ограничить decoded размер:
Image.network(
  'https://example.com/large_photo.jpg',
  cacheWidth: 400,   // декодировать в 400px ширину
  cacheHeight: 300,  // декодировать в 300px высоту
)
// Теперь занимает: 400*300*4 = ~480 KB в RAM ✅

// ResizeImage — для Image.asset и Image.file:
Image(
  image: ResizeImage(
    AssetImage('assets/images/banner.jpg'),
    width: 800,
    height: 400,
  ),
)

// Правило: cacheWidth/cacheHeight = размер отображаемого виджета × devicePixelRatio
// Пример: Image виджет 200×150 логических пикселей, DPR = 3.0:
// cacheWidth = 200*3 = 600, cacheHeight = 150*3 = 450
```

</details>

---

### Вопрос 5 🟡

Как управлять image cache во Flutter?

- A) Image cache управляется автоматически, нельзя настраивать
- B) `PaintingBinding.instance.imageCache` — размер, очистка; `imageCache.maximumSize`, `imageCache.maximumSizeBytes`
- C) `ImageCache.clear()` — статический метод
- D) Только через `cached_network_image`

<details>
<summary>Ответ</summary>

**B) `PaintingBinding.instance.imageCache`**

```dart
// Доступ к image cache:
final cache = PaintingBinding.instance.imageCache;

// Текущее состояние:
print('Кэш: ${cache.currentSize} изображений');
print('Размер: ${cache.currentSizeBytes / 1024 / 1024} MB');

// Настроить максимальный размер (по умолчанию: 1000 images, 100 MB):
cache.maximumSize = 200;                    // максимум 200 изображений
cache.maximumSizeBytes = 50 * 1024 * 1024; // максимум 50 MB

// Очистить весь кэш (например при low memory):
cache.clear();

// Очистить конкретное изображение:
cache.evict(NetworkImage('https://example.com/old.jpg'));

// При low memory — автоматически очищается через:
// WidgetsBinding.instance.addObserver(lifecycleObserver)
// override didHaveMemoryPressure()
```

</details>

---

### Вопрос 6 🟡

Как оптимизировать загрузку изображений в `GridView` или `ListView`?

- A) Загружать все изображения сразу при инициализации
- B) Использовать `ListView.builder` + `cached_network_image` + `cacheWidth`; реализовать pagination; использовать `placeholder`/`errorWidget`
- C) `Image.network` с `gaplessPlayback: true` достаточно
- D) Загружать в `Isolate`

<details>
<summary>Ответ</summary>

**B) `ListView.builder` + lazy loading + кэш**

```dart
GridView.builder(
  gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
    crossAxisCount: 3,
    childAspectRatio: 1,
  ),
  itemCount: photos.length,
  itemBuilder: (context, index) {
    final photo = photos[index];
    return CachedNetworkImage(
      imageUrl: photo.thumbnailUrl, // thumbnail, не полный размер!
      cacheKey: photo.id,           // уникальный ключ
      // Ограничить decoded размер:
      memCacheWidth: 200,
      memCacheHeight: 200,
      // Placeholder пока загружается:
      placeholder: (_, __) => const ColoredBox(color: Colors.grey),
      // Плавное появление:
      fadeInDuration: const Duration(milliseconds: 200),
      fit: BoxFit.cover,
    );
  },
)
```

</details>

---

### Вопрос 7 🟡

Что такое `gaplessPlayback` в `Image` виджете?

- A) Убирает паузу между кадрами GIF
- B) При смене `imageProvider` показывает старое изображение пока загружается новое — избегает мигания пустого места
- C) Отключает анимацию загрузки
- D) Только для APNG формата

<details>
<summary>Ответ</summary>

**B) Показывает старое изображение при смене источника**

```dart
// БЕЗ gaplessPlayback — при смене URL кратко показывает placeholder:
Image.network(user.avatarUrl)

// С gaplessPlayback — плавная смена без мигания:
Image.network(
  user.avatarUrl,
  gaplessPlayback: true,  // показывать старое фото пока грузится новое
)

// Полезно когда:
// - Пользователь меняет аватар
// - Карусель изображений
// - Слайдер с live preview

// Для CachedNetworkImage — используй fadeInDuration вместо:
CachedNetworkImage(
  imageUrl: user.avatarUrl,
  fadeOutDuration: Duration.zero,
  fadeInDuration: const Duration(milliseconds: 300),
)
```

</details>

---

### Вопрос 8 🔴

Как правильно работать с изображениями больших разрешений (например, фото с камеры)?

- A) Отображать в оригинальном разрешении
- B) Ресайзить в Isolate перед отображением: `compute()` + `image` пакет; использовать `cacheWidth`/`cacheHeight`; thumbnail для галереи
- C) Только `Image.file` без трансформаций
- D) Сжимать через platform channel

<details>
<summary>Ответ</summary>

**B) Ресайз в Isolate + правильные cacheWidth/cacheHeight**

```dart
import 'package:image/image.dart' as img;

// Ресайз в Isolate (не блокирует UI):
Future<File> resizeImage(String sourcePath, int maxSize) async {
  return compute(_resizeInIsolate, {'path': sourcePath, 'maxSize': maxSize});
}

File _resizeInIsolate(Map<String, dynamic> params) {
  final path = params['path'] as String;
  final maxSize = params['maxSize'] as int;

  final bytes = File(path).readAsBytesSync();
  final image = img.decodeImage(bytes)!;

  final resized = image.width > image.height
      ? img.copyResize(image, width: maxSize)
      : img.copyResize(image, height: maxSize);

  final output = File('${path}_resized.jpg')
    ..writeAsBytesSync(img.encodeJpg(resized, quality: 85));
  return output;
}

// Для отображения фото с камеры:
Image.file(
  cameraPhoto,
  cacheWidth: 800, // ограничить для gallery view
)
```

</details>

---

### Вопрос 9 🔴

Как реализовать progressive loading изображений (blur hash / low-quality placeholder)?

- A) Загрузить изображение одним запросом
- B) Показать blurhash/LQIP пока грузится полное изображение; использовать `flutter_blurhash` пакет
- C) `Image.network` делает это автоматически
- D) Progressive JPEG поддерживается только на iOS

<details>
<summary>Ответ</summary>

**B) BlurHash + постепенная загрузка**

```dart
// flutter_blurhash:
import 'package:flutter_blurhash/flutter_blurhash.dart';

Stack(children: [
  // Сначала показываем blurred placeholder (мгновенно):
  BlurHash(hash: photo.blurHash),  // 'L6PZfSi_.AyE_3t7t7R**0o#DgR4'

  // Поверх — реальное изображение (с fade in):
  CachedNetworkImage(
    imageUrl: photo.url,
    fadeInDuration: const Duration(milliseconds: 500),
    placeholder: (_, __) => const SizedBox.shrink(),
    fit: BoxFit.cover,
  ),
])

// Генерация BlurHash на сервере:
// npm install blurhash
// const hash = encode(pixels, width, height, 4, 3);

// Или LQIP (Low Quality Image Placeholder) — tiny JPEG:
// Загрузить сначала photo.thumbnailUrl (2-5 KB)
// Потом заменить на photo.fullUrl (500 KB)
```

</details>

---

### Вопрос 10 🔴

Как настроить стратегию кэширования изображений для offline-first приложения?

- A) Кэшировать все изображения бесконечно
- B) Настроить TTL (время жизни), максимальный размер; инвалидировать по URL или ключу; использовать stale-while-revalidate
- C) Только `SharedPreferences` для URL
- D) Offline изображения невозможны во Flutter

<details>
<summary>Ответ</summary>

**B) TTL + размер + стратегия инвалидации**

```dart
// cached_network_image с кастомным CacheManager:
import 'package:flutter_cache_manager/flutter_cache_manager.dart';

class ProductImageCacheManager extends CacheManager with ImageCacheManager {
  static const key = 'productImageCache';

  static final ProductImageCacheManager _instance =
      ProductImageCacheManager._();
  factory ProductImageCacheManager() => _instance;

  ProductImageCacheManager._()
      : super(Config(
          key,
          stalePeriod: const Duration(days: 7),   // TTL: 7 дней
          maxNrOfCacheObjects: 500,                // max 500 файлов
          repo: JsonCacheInfoRepository(databaseName: key),
        ));
}

// Использование:
CachedNetworkImage(
  imageUrl: product.imageUrl,
  cacheManager: ProductImageCacheManager(),
  cacheKey: '${product.id}_${product.imageVersion}', // инвалидация по версии
)

// Ручная инвалидация:
await ProductImageCacheManager().removeFile(product.imageUrl);
await ProductImageCacheManager().emptyCache();
```

</details>
