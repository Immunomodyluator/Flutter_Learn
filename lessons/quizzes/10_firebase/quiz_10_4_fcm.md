# Квиз: Firebase Cloud Messaging (FCM)

**Тема:** 10.4 — FCM Push Notifications  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое FCM token и зачем он нужен?

- A) Токен аутентификации пользователя Firebase Auth
- B) Уникальный идентификатор устройства/установки приложения для отправки push-уведомлений
- C) API ключ для Firebase проекта
- D) Шифрованный токен для Firestore Security Rules

<details>
<summary>Ответ</summary>

**B) Уникальный идентификатор устройства для таргетированных push-уведомлений**

```dart
// Получить FCM token:
final token = await FirebaseMessaging.instance.getToken();
print('FCM Token: $token');

// Сохранить на сервере для отправки уведомлений этому устройству:
await userDoc.update({'fcmToken': token});

// Token обновляется — необходимо слушать:
FirebaseMessaging.instance.onTokenRefresh.listen((newToken) {
  userDoc.update({'fcmToken': newToken});
});
```

</details>

---

### Вопрос 2 🟢

Как запросить разрешение на показ уведомлений на iOS?

- A) iOS автоматически запрашивает разрешение
- B) `await FirebaseMessaging.instance.requestPermission(alert: true, badge: true, sound: true)`
- C) В `Info.plist` — `NSNotificationsUsageDescription`
- D) `NotificationService.requestPermission()`

<details>
<summary>Ответ</summary>

**B) `FirebaseMessaging.instance.requestPermission()`**

```dart
final settings = await FirebaseMessaging.instance.requestPermission(
  alert: true,
  announcement: false,
  badge: true,
  carPlay: false,
  criticalAlert: false,
  provisional: false,
  sound: true,
);

switch (settings.authorizationStatus) {
  case AuthorizationStatus.authorized: print('Granted');
  case AuthorizationStatus.provisional: print('Provisional');
  case AuthorizationStatus.denied: print('Denied');
  case AuthorizationStatus.notDetermined: print('Not determined');
}
```

На Android 13+ также нужно: `POST_NOTIFICATIONS` permission через `permission_handler`.

</details>

---

### Вопрос 3 🟢

Как обработать уведомление когда приложение открыто (foreground)?

- A) Уведомления в foreground отображаются автоматически
- B) `FirebaseMessaging.onMessage.listen(handler)` — получает `RemoteMessage`; на iOS не показывает UI автоматически
- C) `FCM.onForeground(callback)`
- D) `FirebaseMessaging.background.listen(handler)`

<details>
<summary>Ответ</summary>

**B) `FirebaseMessaging.onMessage.listen()` для foreground**

```dart
FirebaseMessaging.onMessage.listen((RemoteMessage message) {
  print('Foreground message: ${message.messageId}');
  print('Title: ${message.notification?.title}');
  print('Body: ${message.notification?.body}');
  print('Data: ${message.data}');

  // Показать локальное уведомление вручную (для Android/iOS foreground):
  flutterLocalNotifications.show(
    message.hashCode,
    message.notification?.title,
    message.notification?.body,
    notificationDetails,
    payload: jsonEncode(message.data),
  );
});
```

</details>

---

### Вопрос 4 🟡

Как обработать клик по уведомлению когда приложение было в фоне?

- A) `FirebaseMessaging.onMessage.listen()` также ловит background tap
- B) `FirebaseMessaging.onMessageOpenedApp.listen(handler)` — вызывается когда пользователь открыл уведомление из background/terminated
- C) `FirebaseMessaging.onBackgroundMessage(handler)`
- D) `FirebaseMessaging.instance.getInitialMessage()` только

<details>
<summary>Ответ</summary>

**B) `onMessageOpenedApp` — background; `getInitialMessage()` — terminated state**

```dart
// Приложение было в фоне:
FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
  _handleNotificationTap(message.data);
});

// Приложение было полностью закрыто (terminated):
final initialMessage = await FirebaseMessaging.instance.getInitialMessage();
if (initialMessage != null) {
  _handleNotificationTap(initialMessage.data);
}

void _handleNotificationTap(Map<String, dynamic> data) {
  if (data['screen'] == 'chat') {
    router.go('/chat/${data['chatId']}');
  }
}
```

</details>

---

### Вопрос 5 🟡

Что такое `@pragma('vm:entry-point')` и зачем нужно при `onBackgroundMessage`?

- A) Оптимизация производительности
- B) Аннотация чтобы Flutter не удалил функцию при tree-shaking — обязательна для background message handler
- C) Маркировка async функций
- D) Указание типа изоляты

<details>
<summary>Ответ</summary>

**B) Предотвращает удаление функции tree-shaking компилятором**

```dart
// background handler должен быть top-level функцией с этой аннотацией:
@pragma('vm:entry-point')
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  // Важно: отдельная изолята — НЕТ доступа к Flutter UI/BuildContext
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  print('Background message: ${message.messageId}');
  // Можно обновить локальную базу данных
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
  await Firebase.initializeApp(...);
  runApp(MyApp());
}
```

</details>

---

### Вопрос 6 🟡

Как подписаться на topics для групповых уведомлений?

- A) `FCM.addTopic('news')`
- B) `await FirebaseMessaging.instance.subscribeToTopic('news')` — устройство получает уведомления для этого topic
- C) `FirebaseMessaging.topics.add('news')`
- D) Подписка только через Firebase Console

<details>
<summary>Ответ</summary>

**B) `subscribeToTopic()` / `unsubscribeFromTopic()`**

```dart
// Подписаться:
await FirebaseMessaging.instance.subscribeToTopic('breaking-news');
await FirebaseMessaging.instance.subscribeToTopic('flutter-updates');

// Отписаться:
await FirebaseMessaging.instance.unsubscribeFromTopic('breaking-news');

// Отправка с сервера на topic (Admin SDK):
const message = {
  topic: 'breaking-news',
  notification: { title: 'Новость', body: 'Важное сообщение' },
  android: { priority: 'high' },
};
await admin.messaging().send(message);
```

Topics — удобны для широковещательных уведомлений без хранения токенов.

</details>

---

### Вопрос 7 🟡

Как создать notification channel на Android для кастомного звука/важности?

- A) В `AndroidManifest.xml` — `<notification-channel>`
- B) `flutter_local_notifications` — `AndroidNotificationChannel` + `createNotificationChannel()` при инициализации
- C) `FCM.setChannel(name, importance)`
- D) Каналы настраиваются только в Firebase Console

<details>
<summary>Ответ</summary>

**B) `flutter_local_notifications` + `AndroidNotificationChannel`**

```dart
const channel = AndroidNotificationChannel(
  'high_importance_channel', // id
  'High Importance Notifications', // name (пользователь видит)
  description: 'For critical alerts',
  importance: Importance.max,
  sound: RawResourceAndroidNotificationSound('notification_sound'),
);

final flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();

// Создать канал (нужно один раз при запуске):
await flutterLocalNotificationsPlugin
    .resolvePlatformSpecificImplementation<
        AndroidFlutterLocalNotificationsPlugin>()
    ?.createNotificationChannel(channel);
```

</details>

---

### Вопрос 8 🔴

Как реализовать data-only message и обработать его в background?

- A) Data-only = `notification: null` — только `data` поле; обрабатывается только в foreground
- B) Data-only message (`notification: null`, только `data:`) всегда доставляется в background handler на обоих платформах — не показывает UI автоматически
- C) Data-only messages не поддерживаются FCM
- D) Идентично notification message

<details>
<summary>Ответ</summary>

**B) Data-only = без notification payload, всегда приходит в handler**

```json
// FCM payload (сервер):
{
  "to": "device_token",
  "data": {
    "type": "sync",
    "userId": "123",
    "action": "refresh_messages"
  }
  // notification: отсутствует!
}
```

```dart
@pragma('vm:entry-point')
Future<void> _backgroundHandler(RemoteMessage message) async {
  await Firebase.initializeApp(...);
  // message.notification == null (data-only)
  // message.data содержит payload
  if (message.data['type'] == 'sync') {
    await localDb.syncMessages(message.data['userId']);
  }
}
```

Преимущество: полный контроль над показом уведомления и поведением.

</details>

---

### Вопрос 9 🔴

Как реализовать тихое (silent) обновление данных через FCM на iOS?

- A) `priority: 'low'` в FCM payload
- B) `content-available: 1` в APNs payload без `alert` — iOS запустит приложение в фоне на до 30 секунд
- C) `silent: true` в data payload
- D) Background refresh в iOS невозможен через FCM

<details>
<summary>Ответ</summary>

**B) `content-available: 1` + `requestPermission(provisional: true)` для фона**

```dart
// requestPermission с background delivery:
await FirebaseMessaging.instance.requestPermission(
  alert: false, // тихое
  sound: false,
  badge: false,
  provisional: true, // iOS: provisional authorization
);

// Настроить iOS foreground options:
await FirebaseMessaging.instance.setForegroundNotificationPresentationOptions(
  alert: false,
  badge: false,
  sound: false,
);
```

```json
// FCM payload для silent push:
{
  "to": "apns_token",
  "apns": {
    "headers": { "apns-push-type": "background", "apns-priority": "5" },
    "payload": { "aps": { "content-available": 1 } }
  },
  "data": { "action": "sync" }
}
```

</details>

---

### Вопрос 10 🔴

Как валидировать FCM уведомления на клиенте для предотвращения spoof атак?

- A) FCM автоматически защищает от подделки
- B) Проверять дополнительную подпись или nonce в `data` payload; критичные действия выполнять только после серверной верификации (не доверять payload напрямую)
- C) Использовать только notification payload (не data)
- D) Валидация FCM токена достаточна

<details>
<summary>Ответ</summary>

**B) Не доверять payload напрямую для критичных действий — верифицировать на сервере**

Принципы безопасности FCM:

1. **Не выполнять критичные действия** (удаление данных, платёж) только по data payload — верифицировать на бэкенде
2. **Подпись**: добавить HMAC подпись в data, проверять на клиенте
3. **Nonce/timestamp**: предотвратить replay attacks
4. **Лимиты на сервере**: rate limiting для отправки уведомлений

```dart
bool _validateNotificationPayload(Map<String, dynamic> data) {
  final timestamp = int.tryParse(data['timestamp'] ?? '');
  if (timestamp == null) return false;
  final age = DateTime.now().millisecondsSinceEpoch - timestamp;
  if (age > 60000) return false; // старше 1 минуты — отклонить
  // Дополнительно: проверить подпись
  return true;
}
```

</details>
