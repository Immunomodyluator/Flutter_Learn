# 12.4 Локальные Push-уведомления

## 1. Суть

`flutter_local_notifications` — пакет для показа уведомлений без Firebase. Полезен для:

- напоминаний и таймеров,
- уведомлений о завершении фоновых задач,
- повторяющихся событий (каждый день в 9:00).

```yaml
dependencies:
  flutter_local_notifications: ^17.0.0
  timezone: ^0.9.4 # для планирования по времени
```

---

## 2. Настройка

**Android** (`AndroidManifest.xml`):

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"/>
<!-- Для точных будильников (Android 12+): -->
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"/>

<!-- Внутри <application>: -->
<receiver android:exported="false"
  android:name="com.dexterous.flutterlocalnotifications.ScheduledNotificationReceiver"/>
```

**iOS** — разрешение запрашивается программно (см. ниже).

---

## 3. Инициализация

```dart
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:timezone/data/latest_all.dart' as tz;
import 'package:timezone/timezone.dart' as tz;

class NotificationService {
  static final _plugin = FlutterLocalNotificationsPlugin();

  static Future<void> init() async {
    tz.initializeTimeZones();
    tz.setLocalLocation(tz.getLocation('Europe/Moscow'));

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

  static void _onTap(NotificationResponse response) {
    // response.payload — данные, переданные при создании уведомления
    debugPrint('Нажали на уведомление: ${response.payload}');
  }
}
```

```dart
// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await NotificationService.init();
  runApp(const MyApp());
}
```

---

## 4. Показ уведомления (немедленно)

```dart
Future<void> showNotification({
  required int id,
  required String title,
  required String body,
  String? payload,
}) async {
  const androidDetails = AndroidNotificationDetails(
    'channel_id',      // уникальный ID канала
    'Основные',        // название канала (видит пользователь)
    channelDescription: 'Уведомления приложения',
    importance: Importance.high,
    priority: Priority.high,
    icon: '@mipmap/ic_launcher',
  );

  const iosDetails = DarwinNotificationDetails(
    presentAlert: true,
    presentBadge: true,
    presentSound: true,
  );

  await _plugin.show(
    id,
    title,
    body,
    const NotificationDetails(android: androidDetails, iOS: iosDetails),
    payload: payload,
  );
}
```

---

## 5. Запланированное уведомление

```dart
// Через 5 минут
Future<void> scheduleNotification({
  required int id,
  required String title,
  required String body,
  required Duration delay,
}) async {
  final scheduledDate = tz.TZDateTime.now(tz.local).add(delay);

  await _plugin.zonedSchedule(
    id,
    title,
    body,
    scheduledDate,
    const NotificationDetails(
      android: AndroidNotificationDetails('reminders', 'Напоминания'),
      iOS: DarwinNotificationDetails(),
    ),
    androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
    uiLocalNotificationDateInterpretation:
        UILocalNotificationDateInterpretation.absoluteTime,
    payload: 'some_data',
  );
}

// Каждый день в 9:00
Future<void> scheduleDailyAt9({required int id}) async {
  final now = tz.TZDateTime.now(tz.local);
  var scheduledDate = tz.TZDateTime(tz.local, now.year, now.month, now.day, 9);
  if (scheduledDate.isBefore(now)) {
    scheduledDate = scheduledDate.add(const Duration(days: 1));
  }

  await _plugin.zonedSchedule(
    id,
    'Доброе утро!',
    'Не забудь про ежедневную цель',
    scheduledDate,
    const NotificationDetails(
      android: AndroidNotificationDetails('daily', 'Ежедневные'),
    ),
    androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
    uiLocalNotificationDateInterpretation:
        UILocalNotificationDateInterpretation.absoluteTime,
    matchDateTimeComponents: DateTimeComponents.time, // повторять ежедневно
  );
}
```

---

## 6. Управление уведомлениями

```dart
// Отмена одного уведомления
await _plugin.cancel(id);

// Отмена всех
await _plugin.cancelAll();

// Список запланированных
final pending = await _plugin.pendingNotificationRequests();
for (final n in pending) {
  print('ID: ${n.id}, Title: ${n.title}');
}

// Запрос разрешения (Android 13+)
final androidPlugin = _plugin
    .resolvePlatformSpecificImplementation<
        AndroidFlutterLocalNotificationsPlugin>();
await androidPlugin?.requestNotificationsPermission();
```

---

## 7. Реальный пример — напоминание о приёме лекарств

```dart
class MedicationReminder {
  final _notifications = NotificationService();

  Future<void> setReminder({
    required int medicationId,
    required String medicationName,
    required TimeOfDay time,
  }) async {
    final now = tz.TZDateTime.now(tz.local);
    var scheduled = tz.TZDateTime(
      tz.local, now.year, now.month, now.day,
      time.hour, time.minute,
    );
    if (scheduled.isBefore(now)) {
      scheduled = scheduled.add(const Duration(days: 1));
    }

    await NotificationService._plugin.zonedSchedule(
      medicationId,
      'Время принять лекарство',
      'Не забудь: $medicationName',
      scheduled,
      const NotificationDetails(
        android: AndroidNotificationDetails(
          'medications', 'Напоминания о лекарствах',
          importance: Importance.max,
          priority: Priority.high,
        ),
        iOS: DarwinNotificationDetails(
          sound: 'default',
          presentAlert: true,
          presentBadge: true,
        ),
      ),
      androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
      uiLocalNotificationDateInterpretation:
          UILocalNotificationDateInterpretation.absoluteTime,
      matchDateTimeComponents: DateTimeComponents.time,
      payload: 'medication:$medicationId',
    );
  }

  Future<void> cancelReminder(int medicationId) async {
    await NotificationService._plugin.cancel(medicationId);
  }
}
```

---

## 8. Под капотом

- На Android используются `NotificationChannel` (API 26+) и `AlarmManager` для точных уведомлений.
- На iOS — `UNUserNotificationCenter`.
- `payload` — произвольная строка, которая приходит в `onDidReceiveNotificationResponse`.
- `AndroidScheduleMode.exactAllowWhileIdle` — работает даже в режиме Doze.

---

## 9. Типичные ошибки

| Ошибка                         | Причина                           | Решение                                                              |
| ------------------------------ | --------------------------------- | -------------------------------------------------------------------- |
| Уведомление не появляется      | Нет разрешения                    | Запросить `requestNotificationsPermission`                           |
| Crash на Android 12+           | Нет `SCHEDULE_EXACT_ALARM`        | Добавить разрешение в манифест                                       |
| Звук не воспроизводится        | Не указан sound канала            | Указать `sound: RawResourceAndroidNotificationSound('notification')` |
| Не работает после перезагрузки | `AlarmManager` сбрасывается       | Слушать `BOOT_COMPLETED` broadcast                                   |
| `timezone` не инициализирован  | `initializeTimeZones()` не вызван | Вызывать в `NotificationService.init()`                              |

---

## 10. Рекомендации

1. **Инициализируй `timezone`** перед первым `zonedSchedule`.
2. **Уникальные ID уведомлений** — иначе новые уведомления перезапишут старые.
3. **`matchDateTimeComponents: DateTimeComponents.time`** — для ежедневных уведомлений.
4. **`cancelAll()`** при разлогине пользователя — чтобы персональные напоминания не оставались.
5. **Тестируй на реальном устройстве** — в эмуляторах уведомления ведут себя иначе.
