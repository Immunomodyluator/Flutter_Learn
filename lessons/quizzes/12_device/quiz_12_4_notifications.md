# Квиз: Локальные уведомления

**Тема:** 12.4 — Local Notifications  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой пакет используется для локальных уведомлений в Flutter?

- A) `firebase_messaging` — только для push
- B) `flutter_local_notifications` — для локальных уведомлений напрямую с устройства
- C) `local_push` пакет
- D) `dart:notification`

<details>
<summary>Ответ</summary>

**B) `flutter_local_notifications`**

```dart
final FlutterLocalNotificationsPlugin notifications =
    FlutterLocalNotificationsPlugin();

// Инициализация:
await notifications.initialize(
  InitializationSettings(
    android: AndroidInitializationSettings('@mipmap/ic_launcher'),
    iOS: DarwinInitializationSettings(
      requestAlertPermission: true,
      requestBadgePermission: true,
      requestSoundPermission: true,
    ),
  ),
  onDidReceiveNotificationResponse: (NotificationResponse response) {
    _handleNotificationTap(response.payload);
  },
);
```

</details>

---

### Вопрос 2 🟢

Как показать простое уведомление?

- A) `Notification.show('title', 'body')`
- B) `await notifications.show(id, title, body, notificationDetails, payload: payload)`
- C) `NotificationService.push(title, body)`
- D) `Toast.show(title, body)`

<details>
<summary>Ответ</summary>

**B) `notifications.show(id, title, body, details)`**

```dart
const AndroidNotificationDetails androidDetails = AndroidNotificationDetails(
  'default_channel', // channel id
  'Default Notifications', // channel name
  channelDescription: 'General notifications',
  importance: Importance.high,
  priority: Priority.high,
  icon: '@mipmap/ic_launcher',
);

await notifications.show(
  42, // уникальный id
  'Привет!',
  'Это локальное уведомление',
  const NotificationDetails(android: androidDetails),
  payload: '{"screen": "home"}', // произвольная строка
);
```

</details>

---

### Вопрос 3 🟢

Как запланировать уведомление на конкретное время?

- A) `notifications.schedule(time, ...)`
- B) `await notifications.zonedSchedule(id, title, body, scheduledTime, details)` с `TZDateTime`
- C) `Future.delayed(duration, () => notifications.show(...))`
- D) Только с Firebase Cloud Messaging

<details>
<summary>Ответ</summary>

**B) `zonedSchedule()` с `TZDateTime` (пакет `timezone`)**

```dart
import 'package:timezone/timezone.dart' as tz;
import 'package:timezone/data/latest.dart' as tz;

// Инициализировать timezone:
tz.initializeTimeZones();

// Запланировать на завтра в 9:00:
await notifications.zonedSchedule(
  1,
  'Напоминание',
  'Пора выпить воды!',
  tz.TZDateTime.now(tz.local).add(const Duration(days: 1)).copyWith(hour: 9, minute: 0, second: 0),
  notificationDetails,
  uiLocalNotificationDateInterpretation:
      UILocalNotificationDateInterpretation.absoluteTime,
  matchDateTimeComponents: DateTimeComponents.time, // каждый день в 9:00
);
```

</details>

---

### Вопрос 4 🟡

Как показать уведомление с кнопками действий (actions)?

- A) Кнопки недоступны в локальных уведомлениях
- B) `AndroidNotificationDetails(actions: [AndroidNotificationAction(id, title)])` — Android; `DarwinNotificationDetails(categoryIdentifier: ...)` — iOS
- C) `NotificationAction.buttons([...])`
- D) Только через нативный код

<details>
<summary>Ответ</summary>

**B) `AndroidNotificationAction` для Android; `category` для iOS**

```dart
const androidDetails = AndroidNotificationDetails(
  'channel_id', 'Channel',
  actions: [
    AndroidNotificationAction('mark_read', 'Прочитать', showsUserInterface: false),
    AndroidNotificationAction('reply', 'Ответить', showsUserInterface: true,
        inputs: [AndroidNotificationActionInput(label: 'Введите ответ')]),
  ],
);

// Обработка нажатия:
onDidReceiveNotificationResponse: (response) {
  switch (response.actionId) {
    case 'mark_read': markAsRead(response.id);
    case 'reply': sendReply(response.input ?? '');
  }
}
```

</details>

---

### Вопрос 5 🟡

Как отменить запланированное или активное уведомление?

- A) `notifications.dismiss(id)`
- B) `await notifications.cancel(id)` — одно; `await notifications.cancelAll()` — все
- C) `notifications.remove(id)`
- D) Уведомления нельзя отменить после показа

<details>
<summary>Ответ</summary>

**B) `cancel(id)` / `cancelAll()`**

```dart
// Отменить конкретное:
await notifications.cancel(42);

// Отменить все:
await notifications.cancelAll();

// Получить список активных:
final pending = await notifications.pendingNotificationRequests();
for (final notification in pending) {
  print('ID: ${notification.id}, Title: ${notification.title}');
}

// Получить активные (уже показанные):
final active = await notifications.getActiveNotifications();
```

</details>

---

### Вопрос 6 🟡

Как показать уведомление с progress bar (прогресс загрузки)?

- A) Только через нативный код
- B) `AndroidNotificationDetails(showProgress: true, maxProgress: 100, progress: current)` — пересоздавать уведомление с тем же ID
- C) `ProgressNotification(progress: 0.5)`
- D) Прогресс в уведомлениях недоступен

<details>
<summary>Ответ</summary>

**B) Обновлять уведомление с тем же ID через `show()`**

```dart
// Показать с прогрессом:
Future<void> _showProgressNotification(int progress) async {
  final androidDetails = AndroidNotificationDetails(
    'download_channel', 'Downloads',
    channelDescription: 'File download progress',
    importance: Importance.low,
    priority: Priority.low,
    onlyAlertOnce: true, // не звенеть при обновлении
    showProgress: true,
    maxProgress: 100,
    progress: progress,
    indeterminate: progress == 0, // % неизвестен
  );

  await notifications.show(
    999, // тот же ID — обновит существующее
    'Загрузка файла',
    '$progress% завершено',
    NotificationDetails(android: androidDetails),
  );
}

// Завершить:
if (progress == 100) {
  await notifications.cancel(999);
  await _showCompletedNotification();
}
```

</details>

---

### Вопрос 7 🟡

Как обработать нажатие на уведомление когда приложение было закрыто?

- A) Невозможно — нажатие игнорируется если приложение закрыто
- B) `getNotificationAppLaunchDetails()` при запуске — проверить запущено ли приложение через уведомление
- C) `onDidReceiveNotificationResponse` вызывается автоматически
- D) Только FCM может запускать закрытые приложения

<details>
<summary>Ответ</summary>

**B) `getNotificationAppLaunchDetails()` в `main()` / `initState`**

```dart
@override
void initState() async {
  super.initState();

  // Проверить был ли запуск через уведомление:
  final launchDetails = await notificationsPlugin.getNotificationAppLaunchDetails();

  if (launchDetails?.didNotificationLaunchApp ?? false) {
    final payload = launchDetails!.notificationResponse?.payload;
    if (payload != null) {
      final data = jsonDecode(payload) as Map<String, dynamic>;
      // Навигировать к нужному экрану:
      WidgetsBinding.instance.addPostFrameCallback((_) {
        router.go('/notifications/${data['id']}');
      });
    }
  }
}
```

</details>

---

### Вопрос 8 🔴

Как реализовать повторяющееся уведомление (ежедневно в определённое время)?

- A) `notifications.schedule(repeat: true, ...)`
- B) `notifications.zonedSchedule()` с `matchDateTimeComponents: DateTimeComponents.time` — каждый день в данное время
- C) `Timer.periodic` + `notifications.show()`
- D) `notifications.showDailyAtTime(Time(hour, minute))`

<details>
<summary>Ответ</summary>

**B) `zonedSchedule` с `matchDateTimeComponents`**

```dart
// Ежедневно в 08:00:
await notifications.zonedSchedule(
  id,
  'Доброе утро!',
  'Не забудь о плане на сегодня',
  _nextInstanceOfTime(8, 0), // TZDateTime
  notificationDetails,
  uiLocalNotificationDateInterpretation:
      UILocalNotificationDateInterpretation.absoluteTime,
  matchDateTimeComponents: DateTimeComponents.time, // только время, любой день
  androidScheduleMode: AndroidScheduleMode.exactAllowWhileIdle,
);

TZDateTime _nextInstanceOfTime(int hour, int minute) {
  final now = tz.TZDateTime.now(tz.local);
  var scheduled = tz.TZDateTime(tz.local, now.year, now.month, now.day, hour, minute);
  if (scheduled.isBefore(now)) {
    scheduled = scheduled.add(const Duration(days: 1));
  }
  return scheduled;
}
```

</details>

---

### Вопрос 9 🔴

Как показать уведомление с большим изображением (BigPictureStyle)?

- A) `notification.image = image`
- B) `AndroidNotificationDetails(styleInformation: BigPictureStyleInformation(bigPicture, ...))` где `bigPicture = FilePathAndroidBitmap` или `ByteArrayAndroidBitmap`
- C) `ImageNotification(url: imageUrl)`
- D) Только встроенное изображение из assets

<details>
<summary>Ответ</summary>

**B) `BigPictureStyleInformation` в `AndroidNotificationDetails`**

```dart
// Скачать изображение и показать:
final http.Response response = await http.get(Uri.parse(imageUrl));
final bigPicture = ByteArrayAndroidBitmap(response.bodyBytes);

final androidDetails = AndroidNotificationDetails(
  'channel_id', 'Images',
  styleInformation: BigPictureStyleInformation(
    bigPicture,
    contentTitle: '<b>Новое фото</b>',
    summaryText: 'Нажмите чтобы открыть',
    largeIcon: bigPicture, // иконка = то же изображение
    hideExpandedLargeIcon: false,
  ),
);

await notifications.show(id, title, body, NotificationDetails(android: androidDetails));
```

</details>

---

### Вопрос 10 🔴

Как убедиться что у приложения есть разрешение на точные уведомления (Android 12+)?

- A) Разрешение запрашивается автоматически
- B) `SCHEDULE_EXACT_ALARM` в AndroidManifest + `notifications.resolvePlatformSpecificImplementation<AndroidFlutterLocalNotificationsPlugin>()?.requestExactAlarmsPermission()`
- C) `Permission.notification.request()`
- D) `TARGET_SDK_VERSION < 31` отключает требование

<details>
<summary>Ответ</summary>

**B) `SCHEDULE_EXACT_ALARM` + `requestExactAlarmsPermission()`**

```xml
<!-- AndroidManifest.xml (API 31+): -->
<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM"
    android:maxSdkVersion="32"/>
<!-- API 33+: USE_EXACT_ALARM (не требует запроса разрешения): -->
<uses-permission android:name="android.permission.USE_EXACT_ALARM"/>
```

```dart
final androidPlugin = notifications
    .resolvePlatformSpecificImplementation<AndroidFlutterLocalNotificationsPlugin>();

// Проверить:
final hasExact = await androidPlugin?.canScheduleExactNotifications();

if (hasExact == false) {
  // Запросить разрешение:
  await androidPlugin?.requestExactAlarmsPermission();
  // или открыть настройки:
  await androidPlugin?.requestPermission();
}
```

</details>
