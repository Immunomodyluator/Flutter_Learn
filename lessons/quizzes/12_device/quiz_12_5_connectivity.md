# Квиз: Сенсоры и подключение

**Тема:** 12.5 — Sensors / Connectivity  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой пакет используется для проверки сетевого подключения?

- A) `dart:io` — `InternetAddress.lookup()`
- B) `connectivity_plus` — определяет тип соединения (WiFi, мобильный, нет)
- C) `network_status`
- D) `flutter_connectivity`

<details>
<summary>Ответ</summary>

**B) `connectivity_plus`**

```dart
import 'package:connectivity_plus/connectivity_plus.dart';

final connectivity = Connectivity();

// Одноразовая проверка:
final result = await connectivity.checkConnectivity();
print(result); // [ConnectivityResult.wifi] или [ConnectivityResult.mobile] и т.д.

// Подписка на изменения:
connectivity.onConnectivityChanged.listen((results) {
  final isConnected = results.any((r) => r != ConnectivityResult.none);
  print('Connected: $isConnected');
});
```

</details>

---

### Вопрос 2 🟢

Важное замечание: `connectivity_plus` гарантирует наличие интернета?

- A) Да — если WiFi подключён, то интернет есть
- B) Нет — `connectivity_plus` проверяет только тип соединения, не доступность интернета. Captive portal или плохой сигнал = ConnectivityResult.wifi но нет интернета
- C) Да — мобильный интернет всегда работает
- D) Да — пакет пингует Google

<details>
<summary>Ответ</summary>

**B) Только тип соединения — не гарантия интернета**

```dart
// Правильная проверка реального интернета:
Future<bool> hasInternetAccess() async {
  try {
    final result = await InternetAddress.lookup('google.com')
        .timeout(const Duration(seconds: 3));
    return result.isNotEmpty && result[0].rawAddress.isNotEmpty;
  } on SocketException {
    return false;
  } on TimeoutException {
    return false;
  }
}

// Или HTTP запрос к известному эндпоинту:
try {
  final response = await http.head(Uri.parse('https://8.8.8.8'))
      .timeout(const Duration(seconds: 3));
  return response.statusCode == 200;
} catch (_) { return false; }
```

</details>

---

### Вопрос 3 🟢

Как прочитать данные акселерометра?

- A) `Accelerometer.read()`
- B) `sensors_plus` пакет — `accelerometerEventStream().listen(event => ...)` или `userAccelerometerEventStream()`
- C) `Platform.accelerometer`
- D) `device_sensors.getAccelerometer()`

<details>
<summary>Ответ</summary>

**B) `sensors_plus` — `accelerometerEventStream()`**

```dart
import 'package:sensors_plus/sensors_plus.dart';

// Акселерометр (включает гравитацию):
accelerometerEventStream().listen((AccelerometerEvent event) {
  print('x=${event.x}, y=${event.y}, z=${event.z}');
});

// Без гравитации (только движение пользователя):
userAccelerometerEventStream().listen((UserAccelerometerEvent event) {
  print('Movement: x=${event.x}, y=${event.y}, z=${event.z}');
});

// Гироскоп:
gyroscopeEventStream().listen((GyroscopeEvent event) {
  print('Rotation: x=${event.x}, y=${event.y}, z=${event.z}');
});
```

</details>

---

### Вопрос 4 🟡

Как получить информацию о батарее?

- A) `Platform.batteryLevel`
- B) `battery_plus` пакет — `Battery().batteryLevel`, `Battery().batteryState`, `Battery().onBatteryStateChanged`
- C) `device_info_plus` — `batteryInfo`
- D) `SystemInfo.battery`

<details>
<summary>Ответ</summary>

**B) `battery_plus`**

```dart
import 'package:battery_plus/battery_plus.dart';

final battery = Battery();

// Уровень (0-100):
final level = await battery.batteryLevel;
print('Battery: $level%');

// Состояние:
final state = await battery.batteryState;
switch (state) {
  case BatteryState.charging: print('Зарядка');
  case BatteryState.discharging: print('Разряд');
  case BatteryState.full: print('Полный');
  case BatteryState.connectedNotCharging: print('Подключён, не заряжается');
  case BatteryState.unknown: print('Неизвестно');
}

// Подписка на изменения:
battery.onBatteryStateChanged.listen((state) {
  // Обновить UI
});
```

</details>

---

### Вопрос 5 🟡

Как получить информацию об устройстве (модель, OS версия)?

- A) `Platform.operatingSystemVersion`
- B) `device_info_plus` пакет — `AndroidDeviceInfo` / `IosDeviceInfo`
- C) `SystemInfo.device`
- D) `DeviceDetails.current`

<details>
<summary>Ответ</summary>

**B) `device_info_plus` — платформо-специфичные объекты**

```dart
import 'package:device_info_plus/device_info_plus.dart';

final deviceInfo = DeviceInfoPlugin();

if (Platform.isAndroid) {
  final androidInfo = await deviceInfo.androidInfo;
  print('Model: ${androidInfo.model}');
  print('SDK: ${androidInfo.version.sdkInt}');
  print('Brand: ${androidInfo.brand}');
  print('Is physical device: ${androidInfo.isPhysicalDevice}');
} else if (Platform.isIOS) {
  final iosInfo = await deviceInfo.iosInfo;
  print('Name: ${iosInfo.name}');
  print('Model: ${iosInfo.model}');
  print('iOS version: ${iosInfo.systemVersion}');
  print('Identifier: ${iosInfo.identifierForVendor}');
}
```

</details>

---

### Вопрос 6 🟡

Как определить тип сети и скорость соединения более детально?

- A) `connectivity_plus` уже предоставляет скорость
- B) `network_info_plus` для WiFi деталей (SSID, BSSID, IP); скорость — только через нативный код или тестовый HTTP запрос
- C) `NetworkSpeed.measure()`
- D) `Connectivity().getNetworkSpeed()`

<details>
<summary>Ответ</summary>

**B) `network_info_plus` для WiFi деталей**

```dart
import 'package:network_info_plus/network_info_plus.dart';

final info = NetworkInfo();

// WiFi информация:
final wifiName = await info.getWifiName();       // 'MyNetwork'
final wifiBSSID = await info.getWifiBSSID();     // 'aa:bb:cc:dd:ee:ff'
final wifiIP = await info.getWifiIP();           // '192.168.1.100'
final wifiGateway = await info.getWifiGatewayIP(); // '192.168.1.1'

// Измерить скорость загрузки:
Future<double> measureDownloadSpeed() async {
  final start = DateTime.now();
  final response = await http.get(
    Uri.parse('https://speed.cloudflare.com/__down?bytes=1000000'), // 1 MB
  );
  final duration = DateTime.now().difference(start).inMilliseconds;
  return (response.bodyBytes.length * 8 / duration) / 1000; // Mbps
}
```

</details>

---

### Вопрос 7 🟡

Как реализовать реактивный NetworkObserver для всего приложения?

- A) Проверять при каждом HTTP запросе
- B) Riverpod `StreamProvider` + `connectivity_plus` — автоматически обновлять состояние во всём приложении
- C) `GlobalKey<NetworkState>()`
- D) `InheritedWidget` с polling

<details>
<summary>Ответ</summary>

**B) `StreamProvider` с connectivity stream**

```dart
// Riverpod:
@riverpod
Stream<bool> isConnected(Ref ref) => Connectivity()
    .onConnectivityChanged
    .map((results) => results.any((r) => r != ConnectivityResult.none));

// В виджете:
final connected = ref.watch(isConnectedProvider);

connected.when(
  data: (isOnline) => isOnline
      ? HomeScreen()
      : OfflineBanner(),
  loading: () => SplashScreen(),
  error: (e, _) => ErrorWidget(e),
)

// Provider (без Riverpod):
class ConnectivityProvider extends ChangeNotifier {
  bool _isConnected = true;

  ConnectivityProvider() {
    Connectivity().onConnectivityChanged.listen((results) {
      _isConnected = results.any((r) => r != ConnectivityResult.none);
      notifyListeners();
    });
  }
}
```

</details>

---

### Вопрос 8 🔴

Как работать с магнитометром и компасом?

- A) `Compass.getBearing()`
- B) `sensors_plus` — `magnetometerEventStream()` для сырых данных; bearing вычислить через `atan2(event.y, event.x)`
- C) `magnetometerEventStream()` — возвращает готовый heading
- D) Только через нативный код

<details>
<summary>Ответ</summary>

**B) `magnetometerEventStream()` + `atan2` для heading**

```dart
import 'dart:math' as math;
import 'package:sensors_plus/sensors_plus.dart';

double _heading = 0;

magnetometerEventStream().listen((MagnetometerEvent event) {
  // Вычислить heading (азимут в радианах):
  final radians = math.atan2(event.y, event.x);
  // Преобразовать в градусы (0-360):
  final degrees = (radians * 180 / math.pi + 360) % 360;
  setState(() => _heading = degrees);
});

// Отобразить:
Transform.rotate(
  angle: -_heading * math.pi / 180,
  child: const Icon(Icons.navigation, size: 60),
)
```

Для точного компаса — работать с акселерометром + магнитометром совместно.

</details>

---

### Вопрос 9 🔴

Как определить встряхивание устройства (shake detection)?

- A) `sensors_plus.onShake.listen()`
- B) Читать `accelerometerEventStream()`, вычислять magnitude изменения ускорения, сравнивать с порогом
- C) `ShakeDetector.listen()`
- D) `MotionSensor.shake` встроен в Flutter

<details>
<summary>Ответ</summary>

**B) Вычислять magnitude + пороговое значение; пакеты `shake` или `shake_gesture`**

```dart
import 'dart:math' as math;

class ShakeDetector {
  static const _shakeThreshold = 15.0;
  static const _minTimeBetweenShakes = Duration(milliseconds: 500);
  DateTime? _lastShakeTime;

  StreamSubscription? _subscription;

  void startListening(VoidCallback onShake) {
    _subscription = accelerometerEventStream().listen((event) {
      final magnitude = math.sqrt(
        event.x * event.x + event.y * event.y + event.z * event.z
      );

      if (magnitude > _shakeThreshold) {
        final now = DateTime.now();
        if (_lastShakeTime == null ||
            now.difference(_lastShakeTime!) > _minTimeBetweenShakes) {
          _lastShakeTime = now;
          onShake();
        }
      }
    });
  }

  void stopListening() => _subscription?.cancel();
}
```

</details>

---

### Вопрос 10 🔴

Как получить уникальный идентификатор устройства?

- A) `Platform.deviceId`
- B) `device_info_plus` — `androidInfo.id` (не persistent) или `flutter_flavorizr`; для стабильного ID — `flutter_udid` или `android_id`
- C) `DeviceId.generate()`
- D) Firebase автоматически создаёт deviceId

<details>
<summary>Ответ</summary>

**B) Нет встроенного стабильного device ID — использовать специальные пакеты**

```dart
// Android:
final androidInfo = await DeviceInfoPlugin().androidInfo;
final id = androidInfo.id; // Android ID (меняется при factory reset)

// iOS: identifierForVendor (меняется при удалении всех приложений разработчика)
final iosInfo = await DeviceInfoPlugin().iosInfo;
final id = iosInfo.identifierForVendor;

// Альтернатива — генерировать UUID при первом запуске и сохранять:
Future<String> getInstallationId() async {
  const key = 'installation_id';
  final prefs = await SharedPreferences.getInstance();
  final existing = prefs.getString(key);
  if (existing != null) return existing;

  final newId = const Uuid().v4();
  await prefs.setString(key, newId);
  return newId;
}
// Хранить в flutter_secure_storage для большей стабильности
```

</details>
