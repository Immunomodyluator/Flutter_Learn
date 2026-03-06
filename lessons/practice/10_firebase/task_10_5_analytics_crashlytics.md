# 10.5: Analytics и Crashlytics — мониторинг приложения

> Project: FitMenu | Глава 10 — Firebase

### 10.5: AnalyticsService с Firebase Analytics и Crashlytics

🎯 **Цель шага:** Подключить Firebase Analytics для трекинга пользовательских действий в FitMenu (добавление рецепта, выполнение цели) и Crashlytics для автоматического сбора crash-репортов.

---

📝 **Техническое задание:**

**Зависимости:**
```yaml
dependencies:
  firebase_analytics: ^10.8.0
  firebase_crashlytics: ^3.4.8
```

**AnalyticsService:**
```dart
class AnalyticsService {
  final _analytics   = FirebaseAnalytics.instance;
  final _crashlytics = FirebaseCrashlytics.instance;

  // Инициализация Crashlytics
  Future<void> initialize() async { ... }

  // Кастомные события
  Future<void> logRecipeViewed(String recipeId, String title) async { ... }
  Future<void> logMealAdded(String mealType, double calories) async { ... }
  Future<void> logDailyGoalAchieved(double targetCalories) async { ... }
  Future<void> logSearch(String query) async { ... }

  // Установить свойства пользователя
  Future<void> setUserProperties(String dietType, int targetCalories) async { ... }

  // Crashlytics
  void logError(Object error, StackTrace? stackTrace, {String? reason}) { ... }
  void setUser(String userId) { ... }
}
```

**Инициализация Crashlytics:**
```dart
Future<void> initialize() async {
  // Перехватывать Flutter ошибки
  FlutterError.onError = _crashlytics.recordFlutterFatalError;

  // Перехватывать Dart зоновые ошибки
  PlatformDispatcher.instance.onError = (error, stack) {
    _crashlytics.recordError(error, stack, fatal: true);
    return true;
  };
}
```

**Navigator Observer для автоматического трекинга экранов:**
```dart
MaterialApp.router(
  routerConfig: appRouter,
  navigatorObservers: [
    FirebaseAnalyticsObserver(analytics: FirebaseAnalytics.instance),
  ],
)
```

**Логирование кастомного события:**
```dart
await _analytics.logEvent(
  name: 'recipe_viewed',
  parameters: {'recipe_id': recipeId, 'recipe_title': title},
);
```

---

✅ **Критерии приёмки:**
- [ ] `FlutterError.onError` переключён на Crashlytics
- [ ] `PlatformDispatcher.instance.onError` перехватывает зоновые ошибки
- [ ] `FirebaseAnalyticsObserver` добавлен в `navigatorObservers`
- [ ] Кастомные события логируются через `logEvent`
- [ ] `setUserId` вызывается после авторизации
- [ ] В debug-режиме `await _crashlytics.setCrashlyticsCollectionEnabled(false)` отключает отправку

---

💡 **Подсказка:** Crashlytics собирает только fatal ошибки по умолчанию — используй `recordError(e, s, fatal: false)` для нефатальных. `kDebugMode` — отключай сбор в разработке чтобы не засорять дашборд. `FirebaseAnalytics.instance.setAnalyticsCollectionEnabled(false)` — GDPR отключение.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/analytics/analytics_service.dart

import 'package:firebase_analytics/firebase_analytics.dart';
import 'package:firebase_crashlytics/firebase_crashlytics.dart';
import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

class AnalyticsService {
  final _analytics   = FirebaseAnalytics.instance;
  final _crashlytics = FirebaseCrashlytics.instance;

  Future<void> initialize() async {
    // В debug отключаем сбор
    if (kDebugMode) {
      await _crashlytics.setCrashlyticsCollectionEnabled(false);
      await _analytics.setAnalyticsCollectionEnabled(false);
      return;
    }

    // Перехват Flutter ошибок
    FlutterError.onError = _crashlytics.recordFlutterFatalError;

    // Перехват Dart зоновых ошибок
    PlatformDispatcher.instance.onError = (error, stack) {
      _crashlytics.recordError(error, stack, fatal: true);
      return true;
    };
  }

  // Установить ID пользователя после логина
  Future<void> setUser(String userId) async {
    await _analytics.setUserId(id: userId);
    await _crashlytics.setUserIdentifier(userId);
  }

  // --- Кастомные события ---

  Future<void> logRecipeViewed(String recipeId, String title) =>
      _analytics.logEvent(
        name: 'recipe_viewed',
        parameters: {'recipe_id': recipeId, 'recipe_title': title},
      );

  Future<void> logMealAdded(String mealType, double calories) =>
      _analytics.logEvent(
        name: 'meal_added',
        parameters: {'meal_type': mealType, 'calories': calories},
      );

  Future<void> logDailyGoalAchieved(double targetCalories) =>
      _analytics.logEvent(
        name: 'daily_goal_achieved',
        parameters: {'target_calories': targetCalories},
      );

  Future<void> logSearch(String query) =>
      _analytics.logSearch(searchTerm: query);

  // --- Свойства пользователя ---

  Future<void> setUserProperties({
    required String dietType,
    required int targetCalories,
  }) async {
    await _analytics.setUserProperty(name: 'diet_type', value: dietType);
    await _analytics.setUserProperty(
        name: 'target_calories', value: targetCalories.toString());
  }

  // --- Crashlytics ---

  void logError(Object error, StackTrace? stackTrace, {String? reason}) {
    _crashlytics.recordError(
      error,
      stackTrace,
      reason: reason,
      fatal: false, // нефатальная ошибка
    );
  }

  void addBreadcrumb(String message) {
    _crashlytics.log(message);
  }
}
```

</details>
