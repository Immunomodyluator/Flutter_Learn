# 12.3: Platform Channels — нативный шагомер

> Project: FitMenu | Глава 12 — Device Features

### 12.3: MethodChannel для доступа к нативному шагомеру

🎯 **Цель шага:** Создать Platform Channel для получения данных нативного шагомера Android/iOS — научиться писать мост между Dart и нативным кодом.

---

📝 **Техническое задание:**

**Dart сторона:**
```dart
// lib/core/platform/step_counter_channel.dart

class StepCounterChannel {
  static const _channel = MethodChannel('com.fitmenu.app/step_counter');
  static const _eventChannel = EventChannel('com.fitmenu.app/step_counter_stream');

  Future<int> getStepCount() async {
    try {
      return await _channel.invokeMethod<int>('getStepCount') ?? 0;
    } on PlatformException catch (e) {
      throw StepCounterException(e.message ?? 'Ошибка шагомера');
    }
  }

  Stream<int> watchStepCount() {
    return _eventChannel
        .receiveBroadcastStream()
        .map((event) => event as int);
  }
}
```

**Android (Kotlin) сторона** `MainActivity.kt`:
```kotlin
private val CHANNEL = "com.fitmenu.app/step_counter"

override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
    super.configureFlutterEngine(flutterEngine)
    MethodChannel(flutterEngine.dartExecutor.binaryMessenger, CHANNEL)
        .setMethodCallHandler { call, result ->
            when (call.method) {
                "getStepCount" -> result.success(getStepsFromSensor())
                else -> result.notImplemented()
            }
        }
}
```

**iOS (Swift) сторона** `AppDelegate.swift`:
```swift
let controller = window?.rootViewController as! FlutterViewController
let channel = FlutterMethodChannel(name: "com.fitmenu.app/step_counter",
                                    binaryMessenger: controller.binaryMessenger)
channel.setMethodCallHandler { call, result in
    if call.method == "getStepCount" {
        // CMPedometer query
        result(steps)
    } else {
        result(FlutterMethodNotImplemented)
    }
}
```

**UI — StepCounterWidget:**
- Отображает дневной счётчик шагов
- Обновляется в реальном времени через `EventChannel`
- Иконка ноги + число шагов + прогресс к цели 10,000

---

✅ **Критерии приёмки:**
- [ ] Channel имя совпадает на Dart и нативной стороне
- [ ] `PlatformException` обрабатывается в Dart
- [ ] `EventChannel` используется для стрима (не polling)
- [ ] `MethodChannel` — для разовых запросов
- [ ] Fallback значение при недоступности шагомера

---

💡 **Подсказка:** `MethodChannel` — запрос-ответ (Future). `EventChannel` — стрим данных. Channel имя — произвольная строка, но должна совпадать на обеих сторонах. Для шагомера на Android нужно разрешение `ACTIVITY_RECOGNITION` в AndroidManifest.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/platform/step_counter_channel.dart

import 'package:flutter/services.dart';

class StepCounterException implements Exception {
  const StepCounterException(this.message);
  final String message;
}

class StepCounterChannel {
  static const _method = MethodChannel('com.fitmenu.app/step_counter');
  static const _event  = EventChannel('com.fitmenu.app/step_counter_stream');

  Future<int> getStepCount() async {
    try {
      return await _method.invokeMethod<int>('getStepCount') ?? 0;
    } on PlatformException catch (e) {
      throw StepCounterException(e.message ?? 'Ошибка шагомера');
    } on MissingPluginException {
      return 0; // платформа не поддерживает
    }
  }

  Stream<int> watchStepCount() {
    return _event
        .receiveBroadcastStream()
        .map((event) => (event as int?) ?? 0)
        .handleError((e) => 0); // fallback при ошибке
  }
}
```

```dart
// lib/features/home/widgets/step_counter_widget.dart

import 'package:flutter/material.dart';

class StepCounterWidget extends StatelessWidget {
  const StepCounterWidget({super.key});

  static const _goal = 10000;

  @override
  Widget build(BuildContext context) {
    final channel = StepCounterChannel();

    return StreamBuilder<int>(
      stream: channel.watchStepCount(),
      builder: (context, snapshot) {
        final steps = snapshot.data ?? 0;
        final progress = (steps / _goal).clamp(0.0, 1.0);

        return Card(
          child: Padding(
            padding: const EdgeInsets.all(16),
            child: Row(
              children: [
                const Icon(Icons.directions_walk, size: 40, color: Colors.blue),
                const SizedBox(width: 12),
                Expanded(
                  child: Column(
                    crossAxisAlignment: CrossAxisAlignment.start,
                    children: [
                      Text('$steps / $_goal шагов',
                          style: Theme.of(context).textTheme.titleSmall),
                      const SizedBox(height: 4),
                      LinearProgressIndicator(value: progress),
                    ],
                  ),
                ),
              ],
            ),
          ),
        );
      },
    );
  }
}
```

</details>
