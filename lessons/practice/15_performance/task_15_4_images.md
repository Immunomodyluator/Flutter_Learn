# 15.4: Оптимизация изображений — cached_network_image

> Project: FitMenu | Глава 15 — Performance

### 15.4: Кэширование и оптимизация изображений рецептов

🎯 **Цель шага:** Подключить `cached_network_image` для фотографий рецептов — с плейсхолдером, обработкой ошибок и контролем размера кэша. Добавить сжатие при загрузке фото с камеры.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  cached_network_image: ^3.3.0
  flutter_cache_manager: ^3.3.0
  image: ^4.1.0      # сжатие изображений
```

**1. CachedNetworkImage в RecipeTile:**
```dart
class RecipeImage extends StatelessWidget {
  const RecipeImage({super.key, required this.imageUrl});
  final String imageUrl;

  @override
  Widget build(BuildContext context) {
    return CachedNetworkImage(
      imageUrl: imageUrl,
      cacheManager: AppCacheManager.instance,  // кастомный кэш
      width: 120, height: 120,
      fit: BoxFit.cover,
      placeholder: (_, __) => const _ShimmerPlaceholder(),
      errorWidget:  (_, __, ___) => const _ErrorPlaceholder(),
    );
  }
}
```

**2. Кастомный CacheManager:**
```dart
class AppCacheManager {
  static const key = 'fitMenuImageCache';

  static CacheManager instance = CacheManager(
    Config(
      key,
      stalePeriod: const Duration(days: 7),
      maxNrOfCacheObjects: 200,
    ),
  );
}
```

**3. Shimmer-плейсхолдер:**
```dart
class _ShimmerPlaceholder extends StatelessWidget {
  const _ShimmerPlaceholder();

  @override
  Widget build(BuildContext context) {
    return ColoredBox(
      color: Theme.of(context).colorScheme.surfaceVariant,
      child: const SizedBox(width: 120, height: 120),
    );
  }
}
```

**4. Сжатие при загрузке с галереи:**
```dart
Future<File?> pickAndCompressImage() async {
  final picker = ImagePicker();
  final picked = await picker.pickImage(
    source: ImageSource.gallery,
    maxWidth:  800,
    maxHeight: 800,
    imageQuality: 85,   // 0-100, JPEG качество
  );
  return picked == null ? null : File(picked.path);
}
```

---

✅ **Критерии приёмки:**
- [ ] `CachedNetworkImage` используется вместо `Image.network` во всех RecipeCard
- [ ] Shimmer или цветной плейсхолдер отображается до загрузки
- [ ] `errorWidget` отображает иконку при ошибке загрузки
- [ ] `CacheManager` ограничивает кэш 200 объектами и 7 днями
- [ ] `imageQuality: 85` используется при загрузке с камеры/галереи
- [ ] Офлайн: ранее загруженные изображения показываются из кэша

---

💡 **Подсказка:** `CachedNetworkImage.evictFromCache(imageUrl)` — удалить один URL из кэша. `DefaultCacheManager().emptyCache()` — очистить весь кэш. Для SVG иконок используй `flutter_svg` — они не требуют кэширования. `ResizeImage` обёртка поможет ограничить размер декодированного изображения в памяти.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/cache/app_cache_manager.dart

import 'package:flutter_cache_manager/flutter_cache_manager.dart';

class AppCacheManager {
  AppCacheManager._();

  static const _cacheKey = 'fitMenuImageCache';

  static final CacheManager instance = CacheManager(
    Config(
      _cacheKey,
      stalePeriod: const Duration(days: 7),
      maxNrOfCacheObjects: 200,
      repo: JsonCacheInfoRepository(databaseName: _cacheKey),
      fileService: HttpFileService(),
    ),
  );

  static Future<void> clearCache() => instance.emptyCache();
  static Future<void> clearImage(String url) =>
      instance.removeFile(url);
}
```

```dart
// lib/features/recipes/presentation/widgets/recipe_image.dart

import 'package:cached_network_image/cached_network_image.dart';
import 'package:flutter/material.dart';

class RecipeImage extends StatelessWidget {
  const RecipeImage({
    super.key,
    required this.imageUrl,
    this.width  = 120,
    this.height = 120,
    this.borderRadius = BorderRadius.zero,
  });

  final String imageUrl;
  final double width;
  final double height;
  final BorderRadius borderRadius;

  @override
  Widget build(BuildContext context) {
    return ClipRRect(
      borderRadius: borderRadius,
      child: CachedNetworkImage(
        imageUrl: imageUrl,
        cacheManager: AppCacheManager.instance,
        width:    width,
        height:   height,
        fit:      BoxFit.cover,
        fadeInDuration: const Duration(milliseconds: 250),
        placeholder:  (_, __) => _Shimmer(width: width, height: height),
        errorWidget:  (_, __, ___) => _ErrorPlaceholder(
          width: width, height: height,
        ),
      ),
    );
  }
}

class _Shimmer extends StatelessWidget {
  const _Shimmer({required this.width, required this.height});
  final double width;
  final double height;

  @override
  Widget build(BuildContext context) {
    return ColoredBox(
      color: Theme.of(context).colorScheme.surfaceContainerHighest,
      child: SizedBox(width: width, height: height),
    );
  }
}

class _ErrorPlaceholder extends StatelessWidget {
  const _ErrorPlaceholder({required this.width, required this.height});
  final double width;
  final double height;

  @override
  Widget build(BuildContext context) {
    return SizedBox(
      width:  width,
      height: height,
      child: Center(
        child: Icon(
          Icons.broken_image_outlined,
          color: Theme.of(context).colorScheme.onSurfaceVariant,
          size: width * 0.4,
        ),
      ),
    );
  }
}
```

</details>
