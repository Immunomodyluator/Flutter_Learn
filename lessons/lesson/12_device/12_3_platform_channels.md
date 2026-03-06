# 12.3 Platform Channels

## 1. Суть

**Platform Channels** — механизм взаимодействия Dart-кода с нативным кодом (Kotlin/Java на Android, Swift/Objective-C на iOS). Используется, когда нужна функциональность, которой нет в стандартных пакетах: биометрия, нативные API, нативные библиотеки.

```
Dart          ←→  MethodChannel  ←→  Kotlin/Swift
(UI логика)        (мост)              (нативный код)
```

Виды каналов:

- `MethodChannel` — вызов методов (запрос / ответ).
- `EventChannel` — поток событий из нативной стороны.
- `BasicMessageChannel` — произвольный двусторонний обмен сообщениями.

---

## 2. MethodChannel — базовый синтаксис

```dart
// --- Dart сторона ---
import 'package:flutter/services.dart';

class BatteryService {
  static const _channel = MethodChannel('com.myapp/battery');

  Future<int> getBatteryLevel() async {
    try {
      final level = await _channel.invokeMethod<int>('getBatteryLevel');
      return level ?? -1;
    } on PlatformException catch (e) {
      debugPrint('Ошибка получения заряда: ${e.message}');
      return -1;
    }
  }
}
```

```kotlin
// --- Android (Kotlin): MainActivity.kt ---
import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import io.flutter.plugin.common.MethodChannel

class MainActivity : FlutterActivity() {
    private val CHANNEL = "com.myapp/battery"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
            .setMethodCallHandler { call, result ->
                when (call.method) {
                    "getBatteryLevel" -> {
                        val level = getBatteryLevel()
                        if (level != -1) result.success(level)
                        else result.error("UNAVAILABLE", "Уровень заряда недоступен", null)
                    }
                    else -> result.notImplemented()
                }
            }
    }

    private fun getBatteryLevel(): Int {
        val batteryManager = getSystemService(BATTERY_SERVICE) as android.os.BatteryManager
        return batteryManager.getIntProperty(android.os.BatteryManager.BATTERY_PROPERTY_CAPACITY)
    }
}
```

```swift
// --- iOS (Swift): AppDelegate.swift ---
import UIKit
import Flutter

@UIApplicationMain
class AppDelegate: FlutterAppDelegate {
    override func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        let controller = window?.rootViewController as! FlutterViewController
        let channel = FlutterMethodChannel(
            name: "com.myapp/battery",
            binaryMessenger: controller.binaryMessenger
        )

        channel.setMethodCallHandler { [weak self] call, result in
            guard call.method == "getBatteryLevel" else {
                result(FlutterMethodNotImplemented)
                return
            }
            self?.receiveBatteryLevel(result: result)
        }

        return super.application(application, didFinishLaunchingWithOptions: launchOptions)
    }

    private func receiveBatteryLevel(result: FlutterResult) {
        UIDevice.current.isBatteryMonitoringEnabled = true
        let level = UIDevice.current.batteryLevel
        if level < 0 {
            result(FlutterError(code: "UNAVAILABLE",
                                message: "Уровень заряда недоступен",
                                details: nil))
        } else {
            result(Int(level * 100))
        }
    }
}
```

---

## 3. EventChannel — поток событий из нативной стороны

```dart
// --- Dart ---
class AccelerometerService {
  static const _channel = EventChannel('com.myapp/accelerometer');

  Stream<Map<String, double>> get stream =>
      _channel.receiveBroadcastStream().map((event) {
        final map = Map<String, dynamic>.from(event as Map);
        return {
          'x': (map['x'] as num).toDouble(),
          'y': (map['y'] as num).toDouble(),
          'z': (map['z'] as num).toDouble(),
        };
      });
}
```

```kotlin
// --- Android (Kotlin) ---
import io.flutter.plugin.common.EventChannel

class MainActivity : FlutterActivity() {
    private val EVENT_CHANNEL = "com.myapp/accelerometer"

    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        EventChannel(flutterEngine.dartExecutor.binaryMessenger, EVENT_CHANNEL)
            .setStreamHandler(object : EventChannel.StreamHandler {
                override fun onListen(args: Any?, events: EventChannel.EventSink?) {
                    // Запустить сенсор и отправлять данные через events?.success(data)
                }
                override fun onCancel(args: Any?) {
                    // Остановить сенсор
                }
            })
    }
}
```

---

## 4. Передача аргументов

```dart
// Dart → нативный
final result = await _channel.invokeMethod<String>('greet', {
  'name': 'Ivan',
  'language': 'ru',
});

// Kotlin:
// val name = call.argument<String>("name")
// val language = call.argument<String>("language")
// result.success("Привет, $name!")
```

Поддерживаемые типы (кодек `StandardMessageCodec`):

| Dart        | Kotlin          | Swift                      |
| ----------- | --------------- | -------------------------- |
| `null`      | `null`          | `nil`                      |
| `bool`      | `Boolean`       | `Bool`                     |
| `int`       | `Int`           | `Int`                      |
| `double`    | `Double`        | `Double`                   |
| `String`    | `String`        | `String`                   |
| `List`      | `List<Any>`     | `[Any]`                    |
| `Map`       | `Map<Any, Any>` | `[AnyHashable: Any]`       |
| `Uint8List` | `ByteArray`     | `FlutterStandardTypedData` |

---

## 5. Реальный пример — вызов нативной биометрии

```dart
class BiometricService {
  static const _channel = MethodChannel('com.myapp/biometric');

  Future<bool> authenticate({String reason = 'Подтвердите личность'}) async {
    try {
      final result = await _channel.invokeMethod<bool>(
        'authenticate',
        {'reason': reason},
      );
      return result ?? false;
    } on PlatformException catch (e) {
      if (e.code == 'NOT_AVAILABLE') {
        throw BiometricNotAvailableException();
      }
      rethrow;
    }
  }
}

// Виджет
class SecureScreen extends StatelessWidget {
  const SecureScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: ElevatedButton(
          onPressed: () async {
            final auth = await BiometricService().authenticate(
              reason: 'Войдите в приложение',
            );
            if (auth && context.mounted) {
              context.go('/home');
            }
          },
          child: const Text('Войти по отпечатку'),
        ),
      ),
    );
  }
}
```

---

## 6. Под капотом

- Все данные сериализуются в бинарный формат через `StandardMessageCodec`.
- Вызов `invokeMethod` асинхронный — Dart приостанавливает выполнение до ответа.
- Нативная сторона **должна** вызвать `result.success()`, `result.error()` или `result.notImplemented()` — иначе Dart зависнет.
- Канал привязан к конкретному `FlutterEngine`, а не к `Activity`/`ViewController`.

---

## 7. Типичные ошибки

| Ошибка                   | Причина                                        | Решение                                                  |
| ------------------------ | ---------------------------------------------- | -------------------------------------------------------- |
| `MissingPluginException` | Нет обработчика на нативной стороне            | Добавь `setMethodCallHandler` в `configureFlutterEngine` |
| Зависание Future         | Нативный код не вызвал `result.*()`            | Всегда вызывай один из трёх методов result               |
| Неверные типы            | Dart ожидает `int`, нативный возвращает `long` | Явно кастуй: `(map['val'] as num).toInt()`               |
| Crash на iOS             | Swift не обработал `not implemented`           | Добавь `result(FlutterMethodNotImplemented)` в `else`    |

---

## 8. Рекомендации

1. **Называй канал по схеме** `com.{company}/{feature}` — исключает коллизии.
2. **Оборачивай в сервис-класс** — не вызывай `MethodChannel` напрямую из UI.
3. **`PlatformException`** всегда нужно обрабатывать — нативный код может вернуть ошибку.
4. **`local_auth`** — готовый пакет для биометрии, не нужно делать вручную.
5. **Для сложных каналов** рассмотри `pigeon` — генерирует type-safe API для обоих сторон.
