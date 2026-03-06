# 12.2 Геолокация

## 1. Суть

Flutter предоставляет доступ к GPS через пакет `geolocator`. Можно получить текущее местоположение или подписаться на поток обновлений позиции. `google_maps_flutter` позволяет отобразить карту.

```yaml
dependencies:
  geolocator: ^11.0.0
  google_maps_flutter: ^2.6.0 # если нужна карта
```

---

## 2. Разрешения

**Android** (`AndroidManifest.xml`):

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<!-- Для фонового отслеживания: -->
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION"/>
```

**iOS** (`Info.plist`):

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Для показа ближайших мест</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>Для фонового отслеживания маршрута</string>
```

---

## 3. Основной синтаксис

```dart
import 'package:geolocator/geolocator.dart';

// Проверка и запрос разрешения
Future<bool> checkAndRequestPermission() async {
  bool serviceEnabled = await Geolocator.isLocationServiceEnabled();
  if (!serviceEnabled) return false; // GPS выключен

  LocationPermission permission = await Geolocator.checkPermission();

  if (permission == LocationPermission.denied) {
    permission = await Geolocator.requestPermission();
    if (permission == LocationPermission.denied) return false;
  }

  if (permission == LocationPermission.deniedForever) {
    await Geolocator.openAppSettings();
    return false;
  }

  return true;
}

// Получить текущую позицию (один раз)
final position = await Geolocator.getCurrentPosition(
  desiredAccuracy: LocationAccuracy.high,
  timeLimit: const Duration(seconds: 10),
);

print('Lat: ${position.latitude}, Lng: ${position.longitude}');
print('Точность: ${position.accuracy} м');
print('Высота: ${position.altitude} м');
print('Скорость: ${position.speed} м/с');

// Последняя известная позиция (быстро, менее точно)
final last = await Geolocator.getLastKnownPosition();

// Поток обновлений позиции
final stream = Geolocator.getPositionStream(
  locationSettings: const LocationSettings(
    accuracy: LocationAccuracy.high,
    distanceFilter: 10, // уведомлять при движении на 10+ метров
  ),
);

// Расчёт расстояния между точками (метры)
final distance = Geolocator.distanceBetween(
  55.7558, 37.6173, // Москва
  59.9343, 30.3351, // Санкт-Петербург
);
print('${distance / 1000} км');
```

---

## 4. Реальный пример — отслеживание местоположения

```dart
class LocationService {
  StreamSubscription<Position>? _subscription;
  Position? _lastPosition;

  Stream<Position> get positionStream => Geolocator.getPositionStream(
        locationSettings: const LocationSettings(
          accuracy: LocationAccuracy.high,
          distanceFilter: 5,
        ),
      );

  Future<Position?> getCurrentLocation() async {
    final hasPermission = await checkAndRequestPermission();
    if (!hasPermission) return null;

    return Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.high,
    );
  }

  void startTracking(void Function(Position) onUpdate) {
    _subscription = positionStream.listen(
      (position) {
        _lastPosition = position;
        onUpdate(position);
      },
      onError: (error) => debugPrint('Location error: $error'),
    );
  }

  void stopTracking() {
    _subscription?.cancel();
    _subscription = null;
  }

  Position? get lastPosition => _lastPosition;
}

// --- Виджет с картой ---
class MapScreen extends StatefulWidget {
  const MapScreen({super.key});

  @override
  State<MapScreen> createState() => _MapScreenState();
}

class _MapScreenState extends State<MapScreen> {
  final _locationService = LocationService();
  Position? _position;

  @override
  void initState() {
    super.initState();
    _initLocation();
  }

  Future<void> _initLocation() async {
    final position = await _locationService.getCurrentLocation();
    if (position != null && mounted) {
      setState(() => _position = position);
    }
  }

  @override
  Widget build(BuildContext context) {
    if (_position == null) {
      return const Center(child: CircularProgressIndicator());
    }

    return GoogleMap(
      initialCameraPosition: CameraPosition(
        target: LatLng(_position!.latitude, _position!.longitude),
        zoom: 15,
      ),
      myLocationEnabled: true,
      myLocationButtonEnabled: true,
      markers: {
        Marker(
          markerId: const MarkerId('current'),
          position: LatLng(_position!.latitude, _position!.longitude),
          infoWindow: const InfoWindow(title: 'Вы здесь'),
        ),
      },
    );
  }
}
```

---

## 5. Точность (LocationAccuracy)

```dart
LocationAccuracy.lowest    // ~3 км — минимальный расход батареи
LocationAccuracy.low       // ~1 км
LocationAccuracy.medium    // ~100 м
LocationAccuracy.high      // ~10 м — GPS
LocationAccuracy.best      // максимальная точность GPS
LocationAccuracy.bestForNavigation // для навигации, максимальный расход батареи
```

---

## 6. Под капотом

- `geolocator` использует `FusedLocationProviderClient` (Android) / `CLLocationManager` (iOS).
- `distanceFilter` — движок отправляет обновление только при движении на указанное расстояние.
- `getLastKnownPosition` не запрашивает GPS — возвращает кешированное значение (может быть устаревшим).

---

## 7. Типичные ошибки

| Ошибка                             | Причина                           | Решение                                         |
| ---------------------------------- | --------------------------------- | ----------------------------------------------- |
| `LocationServiceDisabledException` | GPS выключен                      | Вызвать `Geolocator.openLocationSettings()`     |
| `PermissionDeniedException`        | Отказ в разрешении                | Проверяй статус перед вызовом                   |
| Позиция не обновляется             | `distanceFilter` слишком большой  | Уменьши или поставь 0                           |
| Высокий расход батареи             | `LocationAccuracy.best` постоянно | Используй `high` или уменьши частоту обновлений |
| Не работает в фоне                 | Нет `ACCESS_BACKGROUND_LOCATION`  | Добавь разрешение + отдельный запрос            |

---

## 8. Рекомендации

1. **Запрашивай разрешение перед `getCurrentPosition`** — иначе исключение.
2. **`getLastKnownPosition` для быстрого старта** + `getCurrentPosition` для точности.
3. **`distanceFilter`** — снижает расход батареи при continuous tracking.
4. **Фоновое отслеживание** требует отдельных разрешений и `WorkManager` на Android.
5. **Google Maps API Key** — получи в Cloud Console и добавь в `android/local.properties` и `ios/Runner/AppDelegate.swift`.
