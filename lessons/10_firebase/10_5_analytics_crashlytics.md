# 10.5 Firebase Analytics и Crashlytics

## 1. Суть

**Firebase Analytics** — бесплатный инструмент аналитики: события, воронки, аудитории, конверсии. Автоматически собирает базовые события (first_open, session_start и др.).

**Firebase Crashlytics** — система отчётов об ошибках: краши, нефатальные ошибки, кастомные логи. Незаменима для продакшна.

```yaml
dependencies:
  firebase_analytics: ^10.8.9
  firebase_crashlytics: ^3.4.18
```

---

## 2. Crashlytics — настройка

```dart
// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);

  // Перехватываем все Flutter-ошибки
  FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;

  // Перехватываем ошибки вне Flutter (async, isolates)
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  runApp(const MyApp());
}
```

---

## 3. Crashlytics — использование

```dart
import 'package:firebase_crashlytics/firebase_crashlytics.dart';

final crashlytics = FirebaseCrashlytics.instance;

// Записать нефатальную ошибку (не крашит приложение)
try {
  await someRiskyOperation();
} catch (e, stack) {
  await crashlytics.recordError(e, stack);
}

// Добавить контекст (отображается в отчёте)
crashlytics.setUserId('uid123');
crashlytics.setCustomKey('screen', 'ProductDetail');
crashlytics.setCustomKey('productId', 'prod_456');

// Добавить лог (как хлебные крошки к крашу)
crashlytics.log('Пользователь нажал "Добавить в корзину"');

// Тестовый краш (только для проверки настройки)
// crashlytics.crash(); // NE ISPOLZOVAT V PRODUCTION!

// Отключить в debug
await FirebaseCrashlytics.instance
    .setCrashlyticsCollectionEnabled(!kDebugMode);
```

---

## 4. Analytics — отслеживание событий

```dart
import 'package:firebase_analytics/firebase_analytics.dart';

final analytics = FirebaseAnalytics.instance;

// Предопределённые события (рекомендуется использовать стандартные)
await analytics.logLogin(loginMethod: 'google');
await analytics.logSignUp(signUpMethod: 'email');
await analytics.logSearch(searchTerm: 'flutter widgets');

// E-commerce события
await analytics.logAddToCart(
  currency: 'RUB',
  value: 1990,
  items: [
    AnalyticsEventItem(
      itemId: 'prod_123',
      itemName: 'Flutter Book',
      price: 1990,
    ),
  ],
);
await analytics.logPurchase(
  currency: 'RUB',
  value: 1990,
  transactionId: 'order_456',
);

// Кастомное событие
await analytics.logEvent(
  name: 'share_content',
  parameters: {
    'content_type': 'post',
    'content_id': 'post_123',
    'method': 'whatsapp',
  },
);

// Установить свойства пользователя
await analytics.setUserProperty(name: 'subscription_tier', value: 'premium');
await analytics.setUserId(id: 'uid123');

// Экраны (для воронок)
await analytics.logScreenView(
  screenName: 'ProductDetail',
  screenClass: 'ProductDetailScreen',
);
```

---

## 5. Analytics Observer для GoRouter/Navigator

```dart
// Автоматическое логирование переходов между экранами

// Navigator 1.0
final analytics = FirebaseAnalytics.instance;
MaterialApp(
  navigatorObservers: [FirebaseAnalyticsObserver(analytics: analytics)],
  ...
);

// GoRouter
final router = GoRouter(
  observers: [FirebaseAnalyticsObserver(analytics: FirebaseAnalytics.instance)],
  routes: [...],
);
```

---

## 6. Под капотом

- **Crashlytics**: ошибки буферизуются на устройстве → отправляются при следующем запуске.
- **Analytics**: события отправляются батчами (~30 секунд или при достижении 500 событий).
- В Debug-режиме Firebase Analytics может не отображать события сразу.
- Crashlytics требует подтверждения краша на следующий запуск → для теста нужно перезапустить.

---

## 7. Типичные ошибки

| Ошибка                        | Причина                            | Решение                                                  |
| ----------------------------- | ---------------------------------- | -------------------------------------------------------- |
| Краши не появляются в консоли | ProGuard убирает stack traces      | Загрузи mapping файл через Crashlytics Gradle plugin     |
| Analytics события с задержкой | Нормально — батчинг                | Включи Debug View в консоли для мгновенного просмотра    |
| `_FlutterError` не пойман     | `FlutterError.onError` не настроен | Добавь в `main()` перед `runApp`                         |
| Много лишних событий          | Автоматическое логирование         | Отключи ненужные: `setAnalyticsCollectionEnabled(false)` |

---

## 8. Рекомендации

1. **Crashlytics обязателен для продакшна** — без него знаешь о крашах только от пользователей.
2. **`setCrashlyticsCollectionEnabled(!kDebugMode)`** — не засоряй данные в debug.
3. **`setUserId`** в Crashlytics и Analytics — для контекста кто пострадал.
4. **`log()`** — добавляй хлебные крошки перед рискованными операциями.
5. **Стандартные события Analytics** (logLogin, logPurchase) — предпочтительнее кастомных, они автоматически строят воронки.
6. **Debug View** в Firebase Console — для немедленной проверки событий в разработке.
