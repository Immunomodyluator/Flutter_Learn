# 12. Работа с устройством

## 1. Обзор возможностей

Flutter предоставляет доступ к нативным возможностям устройства через Platform Plugins — пакеты, которые используют Platform Channels для общения с iOS/Android кодом.

| Возможность      | Пакет                                               | Платформы         |
| ---------------- | --------------------------------------------------- | ----------------- |
| Камера / галерея | `camera`, `image_picker`                            | Android, iOS      |
| Геолокация       | `geolocator`, `google_maps_flutter`                 | Android, iOS      |
| Push-уведомления | `flutter_local_notifications`, `firebase_messaging` | Android, iOS      |
| Связь            | `connectivity_plus`                                 | Android, iOS, Web |
| Сенсоры          | `sensors_plus`                                      | Android, iOS      |
| Биометрия        | `local_auth`                                        | Android, iOS      |
| Файлы/Storage    | `permission_handler`                                | Android, iOS      |
| Буфер обмена     | `clipboard` (dart:ui)                               | Все               |
| Батарея          | `battery_plus`                                      | Android, iOS      |
| Устройство       | `device_info_plus`                                  | Все               |

---

## 2. Общий паттерн работы с нативными API

```dart
// 1. Запросить разрешение
// 2. Проверить статус
// 3. Использовать API
// 4. Обработать отказ/permanently denied

import 'package:permission_handler/permission_handler.dart';

Future<bool> requestCameraPermission() async {
  final status = await Permission.camera.request();

  return switch (status) {
    PermissionStatus.granted => true,
    PermissionStatus.denied => false,
    PermissionStatus.permanentlyDenied => {
      // Показать диалог → открыть настройки
      openAppSettings();
      false,
    },
    _ => false,
  };
}
```

---

## 3. Манифесты разрешений

**Android** (`android/app/src/main/AndroidManifest.xml`):

```xml
<uses-permission android:name="android.permission.CAMERA"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.INTERNET"/>
```

**iOS** (`ios/Runner/Info.plist`):

```xml
<key>NSCameraUsageDescription</key>
<string>Приложению нужна камера для создания фото</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>Для отображения ближайших точек</string>
```

---

## 4. Принцип минимальных разрешений

1. Запрашивай разрешения **только при необходимости** — не в начале приложения.
2. **Объясняй зачем** — iOS требует описание в Info.plist, Android 12+ показывает его пользователю.
3. **Обрабатывай `permanentlyDenied`** — предлагай открыть настройки.
4. **Не блокируй UI** при отказе — показывай альтернативный путь.
