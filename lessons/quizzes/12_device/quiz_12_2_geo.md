# Квиз: Геолокация

**Тема:** 12.2 — Geolocation / Maps  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой пакет используется для получения геолокации в Flutter?

- A) `dart:location`
- B) `geolocator` — кроссплатформенный пакет для GPS
- C) `flutter_location`
- D) `geo_plugin`

<details>
<summary>Ответ</summary>

**B) `geolocator`**

```dart
import 'package:geolocator/geolocator.dart';

// Получить текущую позицию:
final Position position = await Geolocator.getCurrentPosition(
  desiredAccuracy: LocationAccuracy.high,
);
print('Lat: ${position.latitude}, Lon: ${position.longitude}');
```

</details>

---

### Вопрос 2 🟢

Как проверить и запросить разрешение на геолокацию?

- A) Разрешение запрашивается автоматически
- B) `Geolocator.checkPermission()` → если `denied` → `Geolocator.requestPermission()`
- C) `Permission.location.request()` из `permission_handler`
- D) Только в AndroidManifest/Info.plist — без кода

<details>
<summary>Ответ</summary>

**B) `checkPermission()` + `requestPermission()`**

```dart
Future<bool> _handleLocationPermission() async {
  bool serviceEnabled = await Geolocator.isLocationServiceEnabled();
  if (!serviceEnabled) {
    // Геолокация отключена в настройках устройства
    await Geolocator.openLocationSettings();
    return false;
  }

  LocationPermission permission = await Geolocator.checkPermission();
  if (permission == LocationPermission.denied) {
    permission = await Geolocator.requestPermission();
    if (permission == LocationPermission.denied) return false;
  }
  if (permission == LocationPermission.deniedForever) {
    await Geolocator.openAppSettings(); // направить в настройки
    return false;
  }
  return true;
}
```

</details>

---

### Вопрос 3 🟢

Как подписаться на обновления позиции в реальном времени?

- A) `Geolocator.position` (polling)
- B) `Geolocator.getPositionStream(settings)` — возвращает `Stream<Position>`
- C) `LocationTracker.listen(callback)`
- D) `navigator.geolocation.watchPosition()` — только для веб

<details>
<summary>Ответ</summary>

**B) `Geolocator.getPositionStream()` → `Stream<Position>`**

```dart
final locationSettings = LocationSettings(
  accuracy: LocationAccuracy.high,
  distanceFilter: 10, // обновлять только при изменении на 10+ метров
);

StreamSubscription<Position>? _positionStream;

void _startTracking() {
  _positionStream = Geolocator.getPositionStream(
    locationSettings: locationSettings,
  ).listen(
    (Position position) => setState(() => _currentPosition = position),
    onError: (error) => print('Location error: $error'),
  );
}

@override
void dispose() {
  _positionStream?.cancel();
  super.dispose();
}
```

</details>

---

### Вопрос 4 🟡

Как вычислить расстояние между двумя координатами?

- A) Формула Хаверсина вручную
- B) `Geolocator.distanceBetween(lat1, lon1, lat2, lon2)` — возвращает метры
- C) `Position.distanceTo(other)`
- D) `GeoUtils.distance(pos1, pos2)`

<details>
<summary>Ответ</summary>

**B) `Geolocator.distanceBetween()` — метры**

```dart
final distanceInMeters = Geolocator.distanceBetween(
  55.7558, 37.6173, // Москва
  59.9343, 30.3351, // Санкт-Петербург
);
print('${(distanceInMeters / 1000).toStringAsFixed(0)} км'); // ~633 км

// Bearing (азимут) между точками:
final bearing = Geolocator.bearingBetween(lat1, lon1, lat2, lon2);
print('Bearing: $bearing°');
```

</details>

---

### Вопрос 5 🟡

Как показать Google Maps в Flutter?

- A) `GoogleMapsWidget(lat, lon)`
- B) `GoogleMap` виджет из пакета `google_maps_flutter` с `onMapCreated` callback
- C) `MapView.google(apiKey: key)`
- D) `WebView(url: 'maps.google.com?...')`

<details>
<summary>Ответ</summary>

**B) `GoogleMap` виджет с `GoogleMapController`**

```dart
GoogleMap(
  onMapCreated: (GoogleMapController controller) {
    _mapController = controller;
  },
  initialCameraPosition: CameraPosition(
    target: LatLng(55.7558, 37.6173),
    zoom: 12,
  ),
  markers: {
    Marker(
      markerId: const MarkerId('home'),
      position: const LatLng(55.7558, 37.6173),
      infoWindow: const InfoWindow(title: 'Москва', snippet: 'Столица'),
      icon: BitmapDescriptor.defaultMarkerWithHue(BitmapDescriptor.hueBlue),
    ),
  },
  myLocationEnabled: true,
  myLocationButtonEnabled: true,
)
```

Нужен API ключ в AndroidManifest и Info.plist.

</details>

---

### Вопрос 6 🟡

Как геокодировать адрес в координаты и обратно?

- A) Только через Google Maps API (HTTP)
- B) Пакет `geocoding` — `locationFromAddress(address)` и `placemarkFromCoordinates(lat, lon)`
- C) `Geolocator.geocode(address)`
- D) Геокодирование встроено в `geolocator`

<details>
<summary>Ответ</summary>

**B) `geocoding` пакет — `locationFromAddress` / `placemarkFromCoordinates`**

```dart
import 'package:geocoding/geocoding.dart';

// Адрес → координаты:
final locations = await locationFromAddress('Красная площадь, Москва');
for (final location in locations) {
  print('${location.latitude}, ${location.longitude}');
}

// Координаты → адрес:
final placemarks = await placemarkFromCoordinates(55.7539, 37.6208);
final placemark = placemarks.first;
print('${placemark.street}, ${placemark.locality}, ${placemark.country}');
// Красная площадь, Москва, Россия
```

</details>

---

### Вопрос 7 🟡

Как реализовать background геолокацию (когда приложение свёрнуто)?

- A) `geolocator` автоматически работает в фоне
- B) `background_locator_2` или настройка `AndroidForegroundService` + iOS `NSLocationAlwaysUsageDescription` с `requestAlwaysAuthorization()`
- C) Background геолокация невозможна в Flutter
- D) `Workmanager` + `Geolocator.getCurrentPosition()`

<details>
<summary>Ответ</summary>

**B) Требует отдельной настройки для каждой платформы**

```xml
<!-- Android: Foreground Service -->
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION"/>
<service android:name="...LocationService" android:foregroundServiceType="location"/>

<!-- iOS Info.plist: -->
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Нужна геолокация в фоне для трекинга маршрута</string>
<key>UIBackgroundModes</key>
<array><string>location</string></array>
```

```dart
// geolocator с Always permission:
LocationPermission.always // вместо whileInUse

// Настройки для фонового режима:
final settings = AndroidSettings(
  accuracy: LocationAccuracy.high,
  foregroundNotificationConfig: const ForegroundNotificationConfig(
    notificationText: 'Отслеживание геолокации активно',
    notificationTitle: 'Трекер',
  ),
);
```

</details>

---

### Вопрос 8 🔴

Как обрабатывать GPS недоступность и таймауты корректно?

- A) Просто ждать сколько нужно
- B) `Geolocator.getCurrentPosition()` с `timeLimit` + fallback на последнюю известную позицию через `getLastKnownPosition()`
- C) `try-catch (TimeoutException)`
- D) GPS всегда доступен в современных устройствах

<details>
<summary>Ответ</summary>

**B) `timeLimit` параметр + `getLastKnownPosition()` как fallback**

```dart
Future<Position?> _getPosition() async {
  try {
    // Сначала попробовать быстро получить:
    return await Geolocator.getCurrentPosition(
      desiredAccuracy: LocationAccuracy.high,
      timeLimit: const Duration(seconds: 10),
    );
  } on TimeoutException {
    // Таймаут — вернуть последнюю известную:
    return await Geolocator.getLastKnownPosition();
  } on LocationServiceDisabledException {
    // Геолокация отключена
    return null;
  } on PermissionDeniedException {
    // Нет разрешения
    return null;
  }
}
```

</details>

---

### Вопрос 9 🔴

Как кастомизировать маркер на Google Maps с кастомным bitmap?

- A) Только PNG из assets
- B) `BitmapDescriptor.fromAssetImage()` для assets; `BitmapDescriptor.fromBytes()` для кастомного рисунка через `Canvas`
- C) `Marker(icon: Icon(Icons.location))`
- D) SVG маркеры встроены

<details>
<summary>Ответ</summary>

**B) `BitmapDescriptor.fromAssetImage()` или кастомный `Canvas` → `fromBytes()`**

```dart
// Из assets:
final BitmapDescriptor icon = await BitmapDescriptor.fromAssetImage(
  const ImageConfiguration(devicePixelRatio: 2.5),
  'assets/map_pin.png',
);

// Кастомный рисунок через Canvas:
Future<BitmapDescriptor> _createCustomMarker(String label) async {
  final pictureRecorder = ui.PictureRecorder();
  final canvas = Canvas(pictureRecorder);
  final paint = Paint()..color = Colors.blue;
  canvas.drawCircle(Offset(50, 50), 50, paint);

  final picture = pictureRecorder.endRecording();
  final img = await picture.toImage(100, 100);
  final bytes = await img.toByteData(format: ui.ImageByteFormat.png);

  return BitmapDescriptor.fromBytes(bytes!.buffer.asUint8List());
}
```

</details>

---

### Вопрос 10 🔴

Как реализовать геофенс (geofence) — уведомление при входе/выходе из зоны?

- A) `Geolocator.geofence(zone)` — встроена
- B) `geofence_service` пакет или ручная реализация: `getPositionStream()` + проверка `Geolocator.distanceBetween() <= radius`
- C) Google Maps API Geofencing
- D) Background геофенс невозможен

<details>
<summary>Ответ</summary>

**B) `geofence_service` пакет или ручная реализация через position stream**

```dart
// Ручная реализация:
class GeofenceMonitor {
  final LatLng center;
  final double radiusMeters;
  bool _isInside = false;

  void startMonitoring() {
    Geolocator.getPositionStream().listen((position) {
      final distance = Geolocator.distanceBetween(
        center.latitude, center.longitude,
        position.latitude, position.longitude,
      );

      final nowInside = distance <= radiusMeters;

      if (nowInside && !_isInside) {
        _isInside = true;
        onEnter(); // вошёл в зону
      } else if (!nowInside && _isInside) {
        _isInside = false;
        onExit(); // вышел из зоны
      }
    });
  }
}
```

</details>
