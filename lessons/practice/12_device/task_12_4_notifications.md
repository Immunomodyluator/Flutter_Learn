# 12.4: Локальные уведомления — напоминания о еде

> Project: FitMenu | Глава 12 — Device Features

### 12.4: MealReminderService с flutter_local_notifications

🎯 **Цель шага:** Настроить локальные напоминания о приёмах пищи через `flutter_local_notifications` — пользователь получает пуш в 8:00, 13:00 и 19:00 с напоминанием записать завтрак/обед/ужин.

---

📝 **Техническое задание:**

**Зависимость:** `flutter_local_notifications: ^17.0.0`

Реализуй `lib/core/notifications/meal_reminder_service.dart`.

**Инициализация:**
```dart
class MealReminderService {
  final _notifications = FlutterLocalNotificationsPlugin();

  Future<void> initialize() async {
    const android = AndroidInitializationSettings('@mipmap/ic_launcher');
    const ios     = DarwinInitializationSettings(requestAlertPermission: true);
    await _notifications.initialize(
      const InitializationSettings(android: android, iOS: ios),
      onDidReceiveNotificationResponse: _onNotificationTap,
    );
  }
}
```

**Расписание напоминаний:**
```dart
Future<void> scheduleMealReminders() async {
  await cancelAll();
  await _scheduleAt(id: 1, hour: 8,  title: 'Время завтрака! 🌅', body: 'Запиши что ты ел на завтрак');
  await _scheduleAt(id: 2, hour: 13, title: 'Время обеда! 🍽️',   body: 'Не забудь записать обед');
  await _scheduleAt(id: 3, hour: 19, title: 'Время ужина! 🌙',    body: 'Запиши ужин в дневник');
}
```

**_scheduleAt (ежедневное расписание):**
```dart
Future<void> _scheduleAt({required int id, required int hour, required String title, required String body}) async {
  await _notifications.zonedSchedule(
    id, title, body,
    _nextInstanceOfTime(hour, 0),
    _notificationDetails(),
    androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
    uiLocalNotificationDateInterpretation: UILocalNotificationDateInterpretation.absoluteTime,
    matchDateTimeComponents: DateTimeComponents.time, // повторять ежедневно
  );
}
```

---

✅ **Критерии приёмки:**
- [ ] `initialize()` настраивает и Android и iOS
- [ ] `zonedSchedule` с `matchDateTimeComponents: DateTimeComponents.time` — ежедневный повтор
- [ ] `cancelAll()` отменяет все уведомления
- [ ] `_onNotificationTap` навигирует на экран плана питания
- [ ] Зависимость `timezone` подключена для работы с временными зонами

---

💡 **Подсказка:** `flutter_local_notifications` требует пакет `timezone` для `zonedSchedule`. `tz.TZDateTime.now(tz.local)` — текущее время в локальной зоне. `matchDateTimeComponents: DateTimeComponents.time` — повторять уведомление каждый день в указанное время.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/notifications/meal_reminder_service.dart

import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/timezone.dart' as tz;
import 'package:timezone/data/latest.dart' as tz;

class MealReminderService {
  final _plugin = FlutterLocalNotificationsPlugin();

  Future<void> initialize() async {
    tz.initializeTimeZones();

    const android = AndroidInitializationSettings('@mipmap/ic_launcher');
    const ios = DarwinInitializationSettings(
      requestAlertPermission: true,
      requestBadgePermission: true,
      requestSoundPermission: true,
    );
    await _plugin.initialize(
      const InitializationSettings(android: android, iOS: ios),
      onDidReceiveNotificationResponse: _onTap,
    );
  }

  Future<void> scheduleMealReminders() async {
    await cancelAll();
    await _scheduleDaily(id: 1, hour: 8,  minute: 0,  title: 'Время завтрака! 🌅', body: 'Запиши что ты ел на завтрак');
    await _scheduleDaily(id: 2, hour: 13, minute: 0,  title: 'Время обеда! 🍽️',   body: 'Не забудь записать обед');
    await _scheduleDaily(id: 3, hour: 19, minute: 0,  title: 'Время ужина! 🌙',    body: 'Запиши ужин в дневник питания');
  }

  Future<void> _scheduleDaily({
    required int id,
    required int hour,
    required int minute,
    required String title,
    required String body,
  }) async {
    await _plugin.zonedSchedule(
      id,
      title,
      body,
      _nextInstance(hour, minute),
      _details(),
      androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
      uiLocalNotificationDateInterpretation:
          UILocalNotificationDateInterpretation.absoluteTime,
      matchDateTimeComponents: DateTimeComponents.time,
    );
  }

  tz.TZDateTime _nextInstance(int hour, int minute) {
    final now = tz.TZDateTime.now(tz.local);
    var scheduled = tz.TZDateTime(tz.local, now.year, now.month, now.day, hour, minute);
    if (scheduled.isBefore(now)) {
      scheduled = scheduled.add(const Duration(days: 1));
    }
    return scheduled;
  }

  NotificationDetails _details() => const NotificationDetails(
        android: AndroidNotificationDetails(
          'meal_reminders',
          'Напоминания о питании',
          channelDescription: 'Ежедневные напоминания записывать приёмы пищи',
          importance: Importance.high,
          priority: Priority.high,
        ),
        iOS: DarwinNotificationDetails(),
      );

  Future<void> cancelAll() => _plugin.cancelAll();
  Future<void> cancel(int id) => _plugin.cancel(id);

  void _onTap(NotificationResponse response) {
    // TODO: GoRouter navigate to /meal-plan
  }
}
```

</details>
