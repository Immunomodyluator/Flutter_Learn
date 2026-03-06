# 15.3 Оптимизация изображений

## 1. Суть

Изображения — одна из главных причин тормозов и перерасхода памяти. Основные проблемы:

- Декодирование большого изображения в UI-потоке.
- Загрузка оригинального изображения (4K) для маленького виджета (50px).
- Повторные network-запросы без кеширования.

---

## 2. ResizeImage и cacheWidth / cacheHeight

```dart
// ❌ Декодирует изображение в полном разрешении
Image.network('https://example.com/photo.jpg')

// ✅ Декодирует только нужный размер (экономит память и CPU)
Image.network(
  'https://example.com/photo.jpg',
  cacheWidth: 100,   // в логических пикселях
  cacheHeight: 100,
  fit: BoxFit.cover,
)

// Для asset
Image.asset(
  'assets/photo.jpg',
  cacheWidth: 200,
)

// ResizeImage — явно
Image(
  image: ResizeImage(
    const NetworkImage('https://example.com/photo.jpg'),
    width: 100,
    height: 100,
  ),
)
```

---

## 3. cached_network_image

```yaml
dependencies:
  cached_network_image: ^3.3.0
```

```dart
import 'package:cached_network_image/cached_network_image.dart';

// Кеширует изображение на диск — при повторном открытии нет сетевого запроса
CachedNetworkImage(
  imageUrl: 'https://example.com/photo.jpg',
  width: 100,
  height: 100,
  fit: BoxFit.cover,
  placeholder: (context, url) => const CircularProgressIndicator(),
  errorWidget: (context, url, error) => const Icon(Icons.broken_image),
)

// Просто провайдер для Image
Image(
  image: CachedNetworkImageProvider('https://example.com/photo.jpg'),
  width: 100,
  cacheWidth: 100,
)

// Очистить кеш
await CachedNetworkImage.evictFromCache(imageUrl);
await DefaultCacheManager().emptyCache();
```

---

## 4. Предзагрузка изображений

```dart
// Предзагрузить изображение в память заранее
await precacheImage(
  const NetworkImage('https://example.com/hero.jpg'),
  context,
);

// Предзагрузить несколько изображений
final futures = imageUrls.map(
  (url) => precacheImage(NetworkImage(url), context),
);
await Future.wait(futures);
```

---

## 5. SVG — flutter_svg

```yaml
dependencies:
  flutter_svg: ^2.0.9
```

```dart
import 'package:flutter_svg/flutter_svg.dart';

// SVG не масштабируется пиксельно — всегда чёткий
SvgPicture.asset(
  'assets/icons/logo.svg',
  width: 48,
  height: 48,
  colorFilter: const ColorFilter.mode(Colors.white, BlendMode.srcIn),
)

SvgPicture.network(
  'https://example.com/icon.svg',
  placeholderBuilder: (context) => const CircularProgressIndicator(),
)

// Предзагрузить SVG
final loader = SvgAssetLoader('assets/icons/logo.svg');
await svg.cache.putIfAbsent(loader.cacheKey(null), () => loader.loadBytes(null));
```

---

## 6. Форматы изображений

| Формат   | Применение             | Размер                               |
| -------- | ---------------------- | ------------------------------------ |
| **WebP** | Фото, фон              | Меньше JPEG на ~30%                  |
| **JPEG** | Фото                   | Стандарт, потери                     |
| **PNG**  | Иконки с прозрачностью | Без потерь, больше                   |
| **SVG**  | Иконки, логотипы       | Векторный, любой размер              |
| **AVIF** | Современные браузеры   | Меньше WebP, медленное декодирование |

```bash
# Конвертировать PNG → WebP
cwebp input.png -o output.webp -q 80

# Оптимизировать PNG
pngcrush input.png output.png

# Оптимизировать JPEG
jpegoptim --max=85 photo.jpg
```

---

## 7. Оптимизация памяти для списков изображений

```dart
// ❌ Держит все изображения в памяти
GridView.builder(
  itemBuilder: (context, index) => Image.network(items[index].imageUrl),
)

// ✅ Автоматически выгружает невидимые изображения из kеша
GridView.builder(
  addAutomaticKeepAlives: false,
  addRepaintBoundaries: true,
  itemBuilder: (context, index) => CachedNetworkImage(
    imageUrl: items[index].imageUrl,
    cacheWidth: 150, // адаптируй под размер ячейки в GridView
    memCacheWidth: 150,
  ),
)
```

---

## 8. ImageCache — управление кешем в памяти

```dart
// Размер in-memory кеша (по умолчанию 100 изображений / 100 МБ)
PaintingBinding.instance.imageCache.maximumSize = 50;
PaintingBinding.instance.imageCache.maximumSizeBytes = 50 * 1024 * 1024; // 50 МБ

// Очистить кеш
PaintingBinding.instance.imageCache.clear();
PaintingBinding.instance.imageCache.clearLiveImages();

// Текущее состояние
final cache = PaintingBinding.instance.imageCache;
print('В кеше: ${cache.currentSize} / ${cache.maximumSize}');
print('Размер: ${cache.currentSizeBytes} байт');
```

---

## 9. Загрузка изображений из assets — оптимально

```yaml
# pubspec.yaml — указывай конкретные файлы, не папки
flutter:
  assets:
    - assets/images/hero.webp
    - assets/icons/logo.svg
    # НЕ: - assets/  (включает всё, раздувает APK)
```

```dart
// Для разных плотностей экранов — папки 1.5x и 2.0x
assets/
  images/
    background.png       (1x)
    1.5x/
      background.png     (1.5x)
    2.0x/
      background.png     (2x)
    3.0x/
      background.png     (3x)
```

---

## 10. Типичные ошибки

| Ошибка                            | Симптом                         | Решение                                          |
| --------------------------------- | ------------------------------- | ------------------------------------------------ |
| Оригинальный размер для thumbnail | OOM, лаги прокрутки             | `cacheWidth`/`cacheHeight` или resize на сервере |
| Без кеширования                   | Повторные запросы при прокрутке | `cached_network_image`                           |
| PNG для фотографий                | Большой APK                     | Конвертировать в JPEG/WebP                       |
| Все assets в одной папке          | Медленная сборка                | Разбить по типам, указывать конкретные файлы     |

---

## 11. Рекомендации

1. **`cacheWidth`/`cacheHeight`** для каждого `Image.network` с известным размером.
2. **`cached_network_image`** — замена стандартному `Image.network` в 99% случаев.
3. **WebP** вместо PNG/JPEG для всех растровых изображений в assets.
4. **Resize изображений на сервере** — не отдавай 2000px для thumbnail.
5. **SVG для иконок** — всегда чёткие, маленький размер файла.
