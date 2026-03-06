# Квиз: Firebase Analytics и Crashlytics

**Тема:** 10.5 — Analytics / Crashlytics / Performance  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как залогировать кастомное событие в Firebase Analytics?

- A) `Analytics.log('event_name')`
- B) `await FirebaseAnalytics.instance.logEvent(name: 'event_name', parameters: {...})`
- C) `Firebase.analytics.track('event_name')`
- D) `Analytics.instance.send(name, params)`

<details>
<summary>Ответ</summary>

**B) `FirebaseAnalytics.instance.logEvent()`**

```dart
// Кастомное событие:
await FirebaseAnalytics.instance.logEvent(
  name: 'purchase',
  parameters: {
    'item_id': 'prod_123',
    'item_name': 'Flutter Course',
    'price': 29.99,
    'currency': 'USD',
  },
);

// Предустановленные события (рекомендуются — лучше аналитика):
await FirebaseAnalytics.instance.logPurchase(
  currency: 'USD',
  value: 29.99,
  items: [...],
);
await FirebaseAnalytics.instance.logScreenView(screenName: 'HomeScreen');
```

</details>

---

### Вопрос 2 🟢

Как записать ошибку в Firebase Crashlytics?

- A) `Crashlytics.log(error)`
- B) `await FirebaseCrashlytics.instance.recordError(error, stackTrace)`
- C) `Firebase.crashlytics.report(e)`
- D) `CrashlyticsService.send(error, stack)`

<details>
<summary>Ответ</summary>

**B) `FirebaseCrashlytics.instance.recordError(error, stackTrace)`**

```dart
// В FlutterError handler (в main):
FlutterError.onError = FirebaseCrashlytics.instance.recordFlutterFatalError;

// Для Dart errors:
PlatformDispatcher.instance.onError = (error, stack) {
  FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
  return true;
};

// Вручную в try-catch:
try {
  await riskyOperation();
} catch (e, stack) {
  await FirebaseCrashlytics.instance.recordError(
    e, stack,
    reason: 'Failed during checkout',
    fatal: false, // non-fatal = warning
  );
}
```

</details>

---

### Вопрос 3 🟢

Как установить User ID для аналитики и Crashlytics?

- A) Автоматически — Firebase Auth UID
- B) `await FirebaseAnalytics.instance.setUserId(id: userId)` и `FirebaseCrashlytics.instance.setUserIdentifier(userId)`
- C) `Firebase.setUser(userId)`
- D) Только в Firebase Console вручную

<details>
<summary>Ответ</summary>

**B) `setUserId()` для Analytics и `setUserIdentifier()` для Crashlytics**

```dart
void onLogin(User user) async {
  // Analytics:
  await FirebaseAnalytics.instance.setUserId(id: user.uid);
  await FirebaseAnalytics.instance.setUserProperty(
    name: 'subscription_tier',
    value: user.subscriptionTier,
  );

  // Crashlytics:
  await FirebaseCrashlytics.instance.setUserIdentifier(user.uid);
  await FirebaseCrashlytics.instance.setCustomKey('email', user.email);

  // При logout — очистить:
  // setUserId(id: null)
  // setUserIdentifier('')
}
```

</details>

---

### Вопрос 4 🟡

Как автоматически логировать переходы между экранами в аналитике?

- A) Вызывать `logScreenView()` вручную в каждом виджете
- B) Создать `FirebaseAnalyticsObserver` и добавить его в `NavigatorObservers` — автологирование при навигации
- C) `FirebaseAnalytics.autoTrack = true`
- D) Использовать кастомный `RouteAware` mixin

<details>
<summary>Ответ</summary>

**B) `FirebaseAnalyticsObserver` в `navigatorObservers`**

```dart
final analytics = FirebaseAnalytics.instance;
final observer = FirebaseAnalyticsObserver(analytics: analytics);

MaterialApp(
  navigatorObservers: [observer],
  // Экраны автологируются при переходах
);

// Для GoRouter:
GoRouter(
  observers: [observer],
  routes: [...],
);

// Кастомное имя экрана через routeSettings:
Navigator.push(
  ctx,
  MaterialPageRoute(
    settings: RouteSettings(name: 'product_detail'),
    builder: (_) => ProductDetailScreen(),
  ),
);
```

</details>

---

### Вопрос 5 🟡

Как использовать Crashlytics logs для отладки крашей?

- A) `print()` — автоматически попадает в Crashlytics
- B) `FirebaseCrashlytics.instance.log('message')` — хлебные крошки (breadcrumbs) до краша
- C) `debugPrint()` в Crashlytics режиме
- D) Логи недоступны в Crashlytics

<details>
<summary>Ответ</summary>

**B) `crashlytics.log()` — breadcrumbs; `setCustomKey()` — контекст**

```dart
class CartService {
  Future<void> checkout(Cart cart) async {
    FirebaseCrashlytics.instance.log('Starting checkout for ${cart.items.length} items');
    FirebaseCrashlytics.instance.setCustomKey('cart_total', cart.total);
    FirebaseCrashlytics.instance.setCustomKey('payment_method', cart.paymentMethod);

    try {
      await paymentService.charge(cart);
      FirebaseCrashlytics.instance.log('Payment successful');
    } catch (e, stack) {
      FirebaseCrashlytics.instance.log('Payment failed: $e');
      await FirebaseCrashlytics.instance.recordError(e, stack);
      rethrow;
    }
  }
}
```

</details>

---

### Вопрос 6 🟡

Как измерить производительность конкретного кода с Firebase Performance Monitoring?

- A) `Performance.measure(code)`
- B) `FirebasePerformance.instance.newTrace('trace_name')` + `trace.start()` / `trace.stop()`
- C) `PerformanceTimer.record(() => code)`
- D) Автоматически меряет всё

<details>
<summary>Ответ</summary>

**B) `newTrace()` + `start()` / `stop()`**

```dart
final performance = FirebasePerformance.instance;

Future<List<Product>> loadProducts() async {
  final trace = performance.newTrace('load_products_trace');
  await trace.start();

  try {
    final products = await api.getProducts();
    trace.setMetric('products_count', products.length);
    trace.putAttribute('category', currentCategory);
    return products;
  } finally {
    await trace.stop(); // всегда вызывать
  }
}
```

</details>

---

### Вопрос 7 🟡

Что такое Remote Config и как его использовать для A/B тестирования?

- A) Синоним Crashlytics
- B) Firebase Remote Config — хранит конфигурационные параметры в облаке; можно менять поведение приложения без обновления; A/B Testing через Experiments
- C) Механизм обновления Firebase SDK
- D) Только для хранения API ключей

<details>
<summary>Ответ</summary>

**B) Remote Config — параметры в облаке + A/B эксперименты**

```dart
final remoteConfig = FirebaseRemoteConfig.instance;

// Настроить:
await remoteConfig.setConfigSettings(RemoteConfigSettings(
  fetchTimeout: const Duration(minutes: 1),
  minimumFetchInterval: const Duration(hours: 1),
));

// Дефолтные значения:
await remoteConfig.setDefaults({'show_new_ui': false, 'max_items': 20});

// Загрузить и применить:
await remoteConfig.fetchAndActivate();

// Использовать:
final showNewUI = remoteConfig.getBool('show_new_ui');
final maxItems = remoteConfig.getInt('max_items');
```

</details>

---

### Вопрос 8 🔴

Как обработать необработанные Dart и Flutter ошибки для полного покрытия Crashlytics?

- A) Только `try-catch` в main
- B) `FlutterError.onError` для Flutter framework ошибок + `PlatformDispatcher.instance.onError` для Dart async ошибок + `runZonedGuarded` для зонной изоляции
- C) `Isolate.current.addErrorListener()`
- D) `CatchAllFlutterErrors.install()`

<details>
<summary>Ответ</summary>

**B) Три механизма для полного покрытия**

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(...);

  // 1. Flutter framework errors:
  FlutterError.onError = (details) {
    FirebaseCrashlytics.instance.recordFlutterFatalError(details);
  };

  // 2. Non-fatal Flutter errors (grey screen):
  FlutterError.presentError = (details) {
    FirebaseCrashlytics.instance.recordFlutterError(details, fatal: false);
  };

  // 3. Dart async errors (вне Flutter):
  PlatformDispatcher.instance.onError = (error, stack) {
    FirebaseCrashlytics.instance.recordError(error, stack, fatal: true);
    return true;
  };

  runApp(const MyApp());
}
```

</details>

---

### Вопрос 9 🔴

Как настроить Firebase Analytics для GDPR/CCPA соответствия?

- A) Firebase автоматически соответствует GDPR
- B) `setAnalyticsCollectionEnabled(false)` до получения согласия; хранить согласие в SharedPreferences
- C) Только через Firebase Console
- D) Использовать другой SDK для GDPR-регионов

<details>
<summary>Ответ</summary>

**B) `setAnalyticsCollectionEnabled(false)` + conditional enable после согласия**

```dart
class AnalyticsConsent {
  // Отключить при первом запуске:
  Future<void> initialize() async {
    final hasConsent = prefs.getBool('analytics_consent');
    await FirebaseAnalytics.instance
        .setAnalyticsCollectionEnabled(hasConsent ?? false);
    await FirebaseCrashlytics.instance
        .setCrashlyticsCollectionEnabled(hasConsent ?? false);
  }

  // Пользователь дал согласие:
  Future<void> grantConsent() async {
    await prefs.setBool('analytics_consent', true);
    await FirebaseAnalytics.instance.setAnalyticsCollectionEnabled(true);
    await FirebaseCrashlytics.instance.setCrashlyticsCollectionEnabled(true);
  }

  // Пользователь отозвал согласие:
  Future<void> revokeConsent() async {
    await prefs.setBool('analytics_consent', false);
    await FirebaseAnalytics.instance.setAnalyticsCollectionEnabled(false);
    await FirebaseCrashlytics.instance.setCrashlyticsCollectionEnabled(false);
  }
}
```

</details>

---

### Вопрос 10 🔴

Как создать пользовательскую вороночную аналитику (funnel) через кастомные события?

- A) Firebase автоматически создаёт воронки
- B) Логировать события на каждом шаге воронки с общими параметрами (`session_id`, `user_tier`); настроить Funnel в Firebase Console → Funnels
- C) Использовать только предустановленные события `begin_checkout` / `purchase`
- D) Воронки доступны только в Google Analytics 360

<details>
<summary>Ответ</summary>

**B) Последовательные события с общими параметрами → Funnel в Console**

```dart
class CheckoutFunnelTracker {
  static const _sessionId = 'session_abc';

  static Future<void> viewCart(int itemCount) async {
    await analytics.logEvent(name: 'checkout_step_1_view_cart', parameters: {
      'session_id': _sessionId,
      'item_count': itemCount,
    });
  }

  static Future<void> enterShipping() async {
    await analytics.logEvent(name: 'checkout_step_2_shipping', parameters: {
      'session_id': _sessionId,
    });
  }

  static Future<void> confirmPayment(String method) async {
    await analytics.logEvent(name: 'checkout_step_3_payment', parameters: {
      'session_id': _sessionId,
      'payment_method': method,
    });
  }

  static Future<void> orderComplete(double total) async {
    await analytics.logPurchase(value: total, currency: 'USD');
  }
}
```

</details>
