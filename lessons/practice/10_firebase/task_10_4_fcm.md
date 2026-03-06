# 10.4: FCM — push-уведомления о приёме пищи

> Project: FitMenu | Глава 10 — Firebase

### 10.4: PushNotificationService с Firebase Cloud Messaging

🎯 **Цель шага:** Настроить Firebase Cloud Messaging в FitMenu — запросить разрешения, получать push-уведомления (напоминания о приёме пищи), обрабатывать нажатие на уведомление с открытием нужного экрана.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  firebase_messaging: ^14.7.0
```

Реализуй `lib/core/notifications/push_notification_service.dart`.

**PushNotificationService:**
```dart
class PushNotificationService {
  final _messaging = FirebaseMessaging.instance;

  Future<void> initialize() async {
    await _requestPermission();
    await _getToken();
    _setupForegroundHandler();
    _setupBackgroundHandler();
    await _handleInitialMessage();
  }

  Future<void> _requestPermission() async { ... }
  Future<String?> _getToken() async { ... }
  void _setupForegroundHandler() { ... }
  void _setupBackgroundHandler() { ... }
  Future<void> _handleInitialMessage() async { ... }
}
```

**Запрос разрешений:**
```dart
final settings = await _messaging.requestPermission(
  alert: true, badge: true, sound: true,
);
```

**Foreground handler:**
```dart
FirebaseMessaging.onMessage.listen((message) {
  // Показать локальное уведомление через flutter_local_notifications
  _showLocalNotification(message.notification?.title, message.notification?.body);
});
```

**Background/terminated handler:**
```dart
// Регистрируется на верхнем уровне (top-level функция):
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  // обработка фонового сообщения
}

// В main():
FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
```

**Навигация при нажатии:**
```dart
FirebaseMessaging.onMessageOpenedApp.listen((message) {
  final screen = message.data['screen'];
  if (screen == 'meal_plan') context.go('/meal-plan');
});
```

---

✅ **Критерии приёмки:**
- [ ] Разрешения запрашиваются через `requestPermission()`
- [ ] FCM токен логируется (для тестирования отправки)
- [ ] `onMessage` обрабатывается в foreground
- [ ] `onBackgroundMessage` — top-level функция с `@pragma('vm:entry-point')`
- [ ] `onMessageOpenedApp` навигирует на нужный экран
- [ ] `getInitialMessage()` обрабатывает запуск через уведомление

---

💡 **Подсказка:** Background handler ОБЯЗАН быть top-level функцией (не методом класса). `@pragma('vm:entry-point')` нужен для tree-shaking. В foreground Flutter не показывает уведомления автоматически — нужен `flutter_local_notifications`. `getInitialMessage()` возвращает сообщение если приложение было ЗАПУЩЕНО через нажатие на уведомление (не просто из background).

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/notifications/push_notification_service.dart

import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_messaging/firebase_messaging.dart';
import 'package:flutter/foundation.dart';

// Должна быть top-level функцией
@pragma('vm:entry-point')
Future<void> firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp();
  debugPrint('Background message: ${message.messageId}');
}

class PushNotificationService {
  final _messaging = FirebaseMessaging.instance;

  Future<void> initialize() async {
    await _requestPermission();
    await _logToken();
    _setupForegroundHandler();
    await _handleInitialMessage();
  }

  Future<void> _requestPermission() async {
    final settings = await _messaging.requestPermission(
      alert: true,
      announcement: false,
      badge: true,
      carPlay: false,
      criticalAlert: false,
      provisional: false,
      sound: true,
    );
    debugPrint('FCM Permission: ${settings.authorizationStatus}');
  }

  Future<void> _logToken() async {
    final token = await _messaging.getToken();
    debugPrint('FCM Token: $token');

    // Обновление токена при изменении (например, переустановка)
    _messaging.onTokenRefresh.listen((newToken) {
      debugPrint('FCM Token refreshed: $newToken');
      // TODO: сохранить новый токен в Firestore
    });
  }

  void _setupForegroundHandler() {
    FirebaseMessaging.onMessage.listen((message) {
      debugPrint('Foreground message: ${message.notification?.title}');
      // TODO: показать через flutter_local_notifications
    });

    // Нажатие на уведомление пока приложение в background (не terminated)
    FirebaseMessaging.onMessageOpenedApp.listen(_handleNavigation);
  }

  Future<void> _handleInitialMessage() async {
    // Приложение запущено через нажатие на уведомление
    final initial = await _messaging.getInitialMessage();
    if (initial != null) _handleNavigation(initial);
  }

  void _handleNavigation(RemoteMessage message) {
    final screen = message.data['screen'] as String?;
    debugPrint('Navigate to: $screen');
    // TODO: использовать GoRouter для навигации
    // if (screen == 'meal_plan') appRouter.go('/meal-plan');
  }
}
```

</details>
