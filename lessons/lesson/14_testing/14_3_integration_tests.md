# 14.3 Интеграционные тесты

## 1. Суть

**Интеграционный тест** запускает полное приложение на реальном устройстве или эмуляторе и автоматически выполняет пользовательские сценарии. Проверяет взаимодействие всех слоёв — UI, бизнес-логику, сеть, БД.

```yaml
dev_dependencies:
  integration_test:
    sdk: flutter

  # patrol — более мощный фреймворк
  patrol: ^3.0.0
```

---

## 2. Базовая настройка

```dart
// integration_test/app_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';
import 'package:myapp/main.dart' as app;

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('E2E: Авторизация', () {
    testWidgets('пользователь может войти в приложение', (tester) async {
      app.main();
      await tester.pumpAndSettle();

      // Ввести email
      await tester.enterText(
        find.byKey(const Key('email_field')),
        'test@example.com',
      );
      await tester.pumpAndSettle();

      // Ввести пароль
      await tester.enterText(
        find.byKey(const Key('password_field')),
        'password123',
      );
      await tester.pumpAndSettle();

      // Нажать кнопку входа
      await tester.tap(find.byKey(const Key('login_button')));
      await tester.pumpAndSettle(const Duration(seconds: 5));

      // Проверить, что перешли на главный экран
      expect(find.byKey(const Key('home_screen')), findsOneWidget);
    });
  });
}
```

---

## 3. Запуск интеграционных тестов

```bash
# На подключённом устройстве/эмуляторе
flutter test integration_test/app_test.dart

# На конкретном устройстве
flutter test integration_test/ -d emulator-5554

# На всех файлах
flutter test integration_test/

# Firebase Test Lab
gcloud firebase test android run \
  --type instrumentation \
  --app build/app/outputs/apk/debug/app-debug.apk \
  --test build/app/outputs/apk/androidTest/debug/app-debug-androidTest.apk \
  --device model=Pixel4,version=30
```

---

## 4. Работа с реальными данными — моки на уровне приложения

```dart
// Для интеграционных тестов инжектируй mock-репозитории через DI
// main_test.dart
void main() {
  setUp(() {
    // Регистрируем тестовые зависимости вместо реальных
    getIt.registerSingleton<UserRepository>(FakeUserRepository());
    getIt.registerSingleton<AuthRepository>(FakeAuthRepository());
  });

  tearDown(() => getIt.reset());
}

// FakeAuthRepository — детерминированный (всегда успех или всегда ошибка)
class FakeAuthRepository implements AuthRepository {
  @override
  Future<User> login(String email, String password) async {
    if (email == 'test@example.com' && password == 'password123') {
      return User(id: '1', name: 'Test User', email: email);
    }
    throw AuthException('Неверные учётные данные');
  }
}
```

---

## 5. patrol — расширенный фреймворк

`patrol` позволяет взаимодействовать с системными диалогами (разрешения, уведомления):

```dart
import 'package:patrol/patrol.dart';

void main() {
  patrolTest(
    'запрашивает разрешение на геолокацию',
    ($) async {
      await $.pumpWidgetAndSettle(const MyApp());

      await $('Включить геолокацию').tap();

      // Обрабатываем системный диалог разрешения
      await $.native.grantPermissionWhenInUse();

      await $.pumpAndSettle();
      expect($('Ваше местоположение'), findsOneWidget);
    },
  );
}
```

```yaml
# pubspec.yaml
dev_dependencies:
  patrol: ^3.0.0

# patrol_test.yaml — конфигурация
android:
  packageName: com.example.myapp
ios:
  bundleId: com.example.myapp
```

```bash
# Установка patrol CLI
dart pub global activate patrol_cli

# Запуск тестов с patrol
patrol test
```

---

## 6. Скриншоты в интеграционных тестах

```dart
import 'package:integration_test/integration_test.dart';

final binding = IntegrationTestWidgetsFlutterBinding.ensureInitialized();

testWidgets('скриншот главного экрана', (tester) async {
  app.main();
  await tester.pumpAndSettle();

  // Сделать скриншот
  await binding.takeScreenshot('home_screen');
});
```

---

## 7. Паттерны организации тестов

```dart
// Page Object Model — инкапсуляция взаимодействий с экраном
class LoginPage {
  final WidgetTester tester;
  const LoginPage(this.tester);

  Future<void> enterEmail(String email) async {
    await tester.enterText(find.byKey(const Key('email_field')), email);
    await tester.pump();
  }

  Future<void> enterPassword(String password) async {
    await tester.enterText(find.byKey(const Key('password_field')), password);
    await tester.pump();
  }

  Future<void> tapLoginButton() async {
    await tester.tap(find.byKey(const Key('login_button')));
    await tester.pumpAndSettle(const Duration(seconds: 5));
  }

  Future<void> loginWith(String email, String password) async {
    await enterEmail(email);
    await enterPassword(password);
    await tapLoginButton();
  }
}

// Использование:
testWidgets('успешный вход', (tester) async {
  app.main();
  await tester.pumpAndSettle();

  final loginPage = LoginPage(tester);
  await loginPage.loginWith('test@example.com', 'password123');

  expect(find.byKey(const Key('home_screen')), findsOneWidget);
});
```

---

## 8. Под капотом

- `IntegrationTestWidgetsFlutterBinding` заменяет стандартный `WidgetsFlutterBinding`.
- Тесты компилируются в тестовый `.apk` (Android) / тестовое приложение (iOS).
- `pumpAndSettle` ждёт до 100 кадров (10 секунд), потом падает — используй `Duration` для долгих операций.
- `patrol` использует нативные тест-фреймворки (UIAutomator / XCTest) для действий вне Flutter.

---

## 9. Типичные ошибки

| Ошибка                          | Причина                                    | Решение                                   |
| ------------------------------- | ------------------------------------------ | ----------------------------------------- |
| Тест зависает                   | Бесконечная анимация или таймер            | `pump(Duration)` вместо `pumpAndSettle()` |
| Виджет не найден                | Нет Key или неверный Key                   | Добавить `Key` в продакшн-код             |
| Нестабильный тест (flaky)       | Зависит от сети/времени                    | Использовать fake-данные                  |
| Тест не видит системные диалоги | `integration_test` не работает вне Flutter | Использовать `patrol`                     |

---

## 10. Рекомендации

1. **Page Object Model** — изолирует тест от деталей UI; рефакторинг экрана = изменение одного класса.
2. **Fake-данные** для интеграционных тестов — производительность и стабильность.
3. **Минимум E2E тестов** — только для критических сценариев (login, checkout, onboarding).
4. **CI/CD** — запускай интеграционные тесты на эмуляторе в пайплайне.
5. **Keys во всех интерактивных элементах** — без них тест хрупкий.
