# Квиз: Platform Channels

**Тема:** 12.3 — Platform Channels  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое Platform Channel и зачем он нужен?

- A) Канал между двумя Flutter виджетами
- B) Коммуникационный мост между Dart кодом и нативным кодом (Android Kotlin/Java, iOS Swift/ObjC)
- C) HTTP канал для сетевых запросов
- D) Канал между изолятами Dart

<details>
<summary>Ответ</summary>

**B) Мост Dart ↔ нативный код**

Нужен когда Flutter SDK не предоставляет API для:

- Работы с Bluetooth, NFC
- Кастомного нативного SDK сторонней библиотеки
- Системных функций не покрытых Flutter пакетами
- Оптимизированных нативных операций

Типы: `MethodChannel` (вызов метода), `EventChannel` (поток событий), `BasicMessageChannel` (двунаправленные сообщения).

</details>

---

### Вопрос 2 🟢

Как вызвать нативный метод через `MethodChannel`?

- A) `NativeCode.call('method')`
- B) `await const MethodChannel('channel.name').invokeMethod('methodName', args)`
- C) `Platform.invoke('method', args)`
- D) `Bridge.call(method: 'name')`

<details>
<summary>Ответ</summary>

**B) `MethodChannel.invokeMethod()`**

```dart
// Dart сторона:
static const _channel = MethodChannel('com.example.app/device');

Future<String> getBatteryLevel() async {
  try {
    final int level = await _channel.invokeMethod('getBatteryLevel');
    return '$level%';
  } on PlatformException catch (e) {
    return 'Battery level unavailable: ${e.message}';
  }
}
```

</details>

---

### Вопрос 3 🟢

Как реализовать обработчик метода на Android (Kotlin)?

- A) `Channel.handle(method) { ... }`
- B) `channel.setMethodCallHandler { call, result -> ... }` в `MainActivity`
- C) `override fun onMethodCall(call: MethodCall)`
- D) `NativePlugin.register(channel, handler)`

<details>
<summary>Ответ</summary>

**B) `setMethodCallHandler` в `FlutterActivity` / `MethodCallHandler`**

```kotlin
// MainActivity.kt:
class MainActivity : FlutterActivity() {
  private val CHANNEL = "com.example.app/device"

  override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    super.configureFlutterEngine(flutterEngine)
    MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
      .setMethodCallHandler { call, result ->
        when (call.method) {
          "getBatteryLevel" -> {
            val batteryLevel = getBatteryLevel()
            if (batteryLevel != -1) result.success(batteryLevel)
            else result.error("UNAVAILABLE", "Battery level unavailable", null)
          }
          else -> result.notImplemented()
        }
      }
  }

  private fun getBatteryLevel(): Int {
    val batteryManager = getSystemService(BATTERY_SERVICE) as BatteryManager
    return batteryManager.getIntProperty(BatteryManager.BATTERY_PROPERTY_CAPACITY)
  }
}
```

</details>

---

### Вопрос 4 🟡

Как реализовать обработчик метода на iOS (Swift)?

- A) `override func application(_ app: UIApplication ...)`
- B) `FlutterMethodChannel(name: channelName, binaryMessenger: controller)` + `setMethodCallHandler` в `AppDelegate`
- C) `PlatformChannel.register(self)` в Swift
- D) `@objc func handleFlutterCall()`

<details>
<summary>Ответ</summary>

**B) `FlutterMethodChannel` + `setMethodCallHandler` в `AppDelegate`**

```swift
// AppDelegate.swift:
@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    let controller = window?.rootViewController as! FlutterViewController
    let channel = FlutterMethodChannel(
      name: "com.example.app/device",
      binaryMessenger: controller.binaryMessenger
    )

    channel.setMethodCallHandler { [weak self] call, result in
      switch call.method {
      case "getBatteryLevel":
        self?.receiveBatteryLevel(result: result)
      default:
        result(FlutterMethodNotImplemented)
      }
    }
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }

  private func receiveBatteryLevel(result: FlutterResult) {
    UIDevice.current.isBatteryMonitoringEnabled = true
    if UIDevice.current.batteryState == .unknown {
      result(FlutterError(code: "UNAVAILABLE", message: "Battery info unavailable", details: nil))
    } else {
      result(Int(UIDevice.current.batteryLevel * 100))
    }
  }
}
```

</details>

---

### Вопрос 5 🟡

Что такое `EventChannel` и когда его использовать?

- A) Альтернативное название `MethodChannel`
- B) Для непрерывных потоков событий от нативного кода к Dart (акселерометр, Bluetooth события, сенсоры)
- C) Для двусторонней передачи сообщений
- D) Только для одноразовых событий

<details>
<summary>Ответ</summary>

**B) `EventChannel` — поток событий нативный → Dart**

```dart
// Dart:
const _channel = EventChannel('com.example.app/accelerometer');
Stream<dynamic> get _accelerometerStream => _channel.receiveBroadcastStream();

// Использование:
_subscription = _accelerometerStream.listen(
  (dynamic event) {
    final data = event as List<dynamic>;
    print('x=${data[0]}, y=${data[1]}, z=${data[2]}');
  },
  onError: (error) => print('Error: $error'),
);
```

```kotlin
// Android — EventChannel handler:
EventChannel(binaryMessenger, CHANNEL).setStreamHandler(object : EventChannel.StreamHandler {
  override fun onListen(arguments: Any?, events: EventChannel.EventSink) {
    // начать отправку событий
    sensorManager.registerListener(object : SensorEventListener {
      override fun onSensorChanged(event: SensorEvent) {
        events.success(listOf(event.values[0], event.values[1], event.values[2]))
      }
      override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {}
    }, sensor, SensorManager.SENSOR_DELAY_NORMAL)
  }
  override fun onCancel(arguments: Any?) {
    sensorManager.unregisterListener(this)
  }
})
```

</details>

---

### Вопрос 6 🟡

Какие типы данных можно передавать через Platform Channels?

- A) Только `String` и `int`
- B) Null, bool, int, double, String, Uint8List, Int32List, Int64List, Float64List, List, Map — через StandardMessageCodec
- C) Любые Dart объекты (`Serializable`)
- D) Только JSON строки

<details>
<summary>Ответ</summary>

**B) Базовые типы + коллекции через `StandardMessageCodec`**

```dart
// Передача Map:
final result = await _channel.invokeMethod<Map<dynamic, dynamic>>('getDeviceInfo');
// { 'model': 'Pixel 6', 'os': 'Android', 'version': '13' }

// Передача аргументов:
await _channel.invokeMethod('openFile', {
  'path': '/storage/file.pdf',
  'mimeType': 'application/pdf',
});

// Передача байт — Uint8List:
final Uint8List bytes = await _channel.invokeMethod('compressImage', {
  'path': imagePath,
  'quality': 80,
});
```

Кастомные объекты → сериализовать в Map вручную.

</details>

---

### Вопрос 7 🟡

Как создать Flutter пакет с Platform Channel (plugin)?

- A) `flutter create mypackage --template=package`
- B) `flutter create --template=plugin --platforms=android,ios mypackage` — создаёт структуру с нативными папками
- C) `dart create mypackage --type=plugin`
- D) Плагины создаются только в IDE

<details>
<summary>Ответ</summary>

**B) `flutter create --template=plugin ...`**

```bash
flutter create --template=plugin --platforms=android,ios,web my_plugin
```

Структура:

```
my_plugin/
  lib/my_plugin.dart          # Dart API
  android/src/main/kotlin/... # Android implementation
  ios/Classes/...             # iOS implementation
  test/                       # Unit tests
  example/                    # Example app
  pubspec.yaml                # plugin: platforms:
```

Federated plugins: разделение на `_platform_interface` + `_android`/`_ios` пакеты.

</details>

---

### Вопрос 8 🔴

Как обеспечить безопасную передачу данных через Platform Channels?

- A) Все вызовы безопасны по умолчанию
- B) Валидировать входные данные на нативной стороне; channel name внутри app bundle (не внешнее) — injection невозможен; не передавать sensitive данные через channels без необходимости
- C) Шифровать данные в channel
- D) Использовать `SecureMethodChannel`

<details>
<summary>Ответ</summary>

**B) Валидация на нативной стороне + минимизация sensitive данных**

```kotlin
// Android — валидация:
channel.setMethodCallHandler { call, result ->
  when (call.method) {
    "readFile" -> {
      val path = call.argument<String>("path") ?: run {
        result.error("INVALID_ARGS", "Path required", null)
        return@setMethodCallHandler
      }
      // Валидация пути — только разрешённые директории:
      val safePath = path.trim()
      if (!safePath.startsWith(filesDir.absolutePath)) {
        result.error("SECURITY_ERROR", "Access denied: $path", null)
        return@setMethodCallHandler
      }
      // Продолжить...
    }
  }
}
```

</details>

---

### Вопрос 9 🔴

Как тестировать код использующий `MethodChannel`?

- A) Только на реальном устройстве
- B) `TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger.setMockMethodCallHandler()` — мок нативных ответов в unit tests
- C) `MethodChannel.setMock(handler)`
- D) Извлечь логику от Platform Channel и тестировать её отдельно

<details>
<summary>Ответ</summary>

**B) `setMockMethodCallHandler()` в тестах + рефакторинг для изоляции**

```dart
// В тестах:
void main() {
  TestWidgetsFlutterBinding.ensureInitialized();

  setUp(() {
    TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
        .setMockMethodCallHandler(
      const MethodChannel('com.example.app/device'),
      (MethodCall methodCall) async {
        switch (methodCall.method) {
          case 'getBatteryLevel': return 85;
          default: return null;
        }
      },
    );
  });

  tearDown(() {
    TestDefaultBinaryMessengerBinding.instance.defaultBinaryMessenger
        .setMockMethodCallHandler(
      const MethodChannel('com.example.app/device'), null,
    );
  });

  test('getBatteryLevel returns value', () async {
    final service = DeviceService();
    expect(await service.getBatteryLevel(), '85%');
  });
}
```

</details>

---

### Вопрос 10 🔴

Как вызвать Dart код из нативного (обратный вызов)?

- A) Нельзя — только Dart → нативный
- B) Использовать `invokeMethod()` на нативной стороне через `MethodChannel` объект; или `EventChannel` для потока событий
- C) `DartCallback.call(name, args)`
- D) Только через `EventChannel`

<details>
<summary>Ответ</summary>

**B) Нативный код может вызвать Dart через `channel.invokeMethod()`**

```kotlin
// Android — вызов Dart из Kotlin:
class MainActivity : FlutterActivity() {
  private lateinit var methodChannel: MethodChannel

  override fun configureFlutterEngine(engine: FlutterEngine) {
    super.configureFlutterEngine(engine)
    methodChannel = MethodChannel(engine.dartExecutor.binaryMessenger, CHANNEL)

    // Например, при получении deep link:
    methodChannel.invokeMethod("onDeepLink", mapOf("url" to deepLinkUrl))
  }
}
```

```dart
// Dart — зарегистрировать обработчик:
const _channel = MethodChannel('com.example.app/device');

@override
void initState() {
  super.initState();
  _channel.setMethodCallHandler((MethodCall call) async {
    switch (call.method) {
      case 'onDeepLink':
        final url = call.arguments['url'] as String;
        _handleDeepLink(url);
    }
  });
}
```

</details>
