# 12.2: Геолокация — ближайшие рестораны на карте

> Project: FitMenu | Глава 12 — Device Features

### 12.2: LocationService для поиска ресторанов рядом

🎯 **Цель шага:** Получить текущее местоположение пользователя через `geolocator` и показать ближайшие рестораны с здоровой едой на экране — с обработкой разрешений.

---

📝 **Техническое задание:**

**Зависимость:** `geolocator: ^11.0.0`

Реализуй `lib/features/explore/services/location_service.dart`.

**LocationService:**
```dart
class LocationService {
  Future<Position?> getCurrentPosition() async { ... }
  Future<bool> requestPermission() async { ... }
  Stream<Position> watchPosition() { ... }
}
```

**Алгоритм getCurrentPosition:**
1. Проверить `LocationPermission` через `Geolocator.checkPermission()`
2. Если `denied` → `requestPermission()`
3. Если `deniedForever` → показать `openAppSettings()`
4. `Geolocator.getCurrentPosition(desiredAccuracy: LocationAccuracy.medium)`

**UI виджет — NearbyRestaurantsScreen:**
- FutureBuilder на `LocationService().getCurrentPosition()`
- При загрузке: `CircularProgressIndicator`
- При ошибке: кнопка "Разрешить геолокацию"
- При успехе: заглушка-карта с координатами (реальная карта в бонусе)

---

✅ **Критерии приёмки:**
- [ ] `LocationPermission.deniedForever` обрабатывается отдельно
- [ ] `Geolocator.openAppSettings()` вызывается при `deniedForever`
- [ ] `desiredAccuracy: LocationAccuracy.medium` (не high — батарея)
- [ ] `watchPosition()` возвращает `Stream<Position>`
- [ ] Разрешения описаны в AndroidManifest / Info.plist

---

💡 **Подсказка:** `Geolocator.isLocationServiceEnabled()` проверяет включен ли GPS на устройстве (отдельно от разрешений). Используй `LocationAccuracy.medium` вместо `high` для экономии батареи. `Geolocator.distanceBetween(lat1, lon1, lat2, lon2)` вычисляет расстояние в метрах.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/explore/services/location_service.dart

import 'package:geolocator/geolocator.dart';

class LocationException implements Exception {
  const LocationException(this.message);
  final String message;
}

class LocationService {
  Future<bool> requestPermission() async {
    var permission = await Geolocator.checkPermission();
    if (permission == LocationPermission.denied) {
      permission = await Geolocator.requestPermission();
    }
    return permission == LocationPermission.always ||
           permission == LocationPermission.whileInUse;
  }

  Future<Position?> getCurrentPosition() async {
    final serviceEnabled = await Geolocator.isLocationServiceEnabled();
    if (!serviceEnabled) throw const LocationException('GPS отключён');

    final permission = await Geolocator.checkPermission();
    if (permission == LocationPermission.deniedForever) {
      await Geolocator.openAppSettings();
      throw const LocationException('Разрешение отклонено навсегда');
    }
    if (permission == LocationPermission.denied) {
      final granted = await requestPermission();
      if (!granted) throw const LocationException('Разрешение не получено');
    }

    return Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.medium,
      timeLimit: const Duration(seconds: 10),
    );
  }

  Stream<Position> watchPosition() => Geolocator.getPositionStream(
        locationSettings: const LocationSettings(
          accuracy: LocationAccuracy.medium,
          distanceFilter: 50, // обновлять каждые 50 метров
        ),
      );
}
```

</details>
