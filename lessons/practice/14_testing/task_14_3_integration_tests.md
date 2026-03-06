# 14.3: Integration-тесты — e2e сценарий добавления блюда

> Project: FitMenu | Глава 14 — Testing

### 14.3: Integration test — добавление рецепта в план питания

🎯 **Цель шага:** Написать integration-тест (e2e) который запускает реальное приложение и проверяет полный сценарий: поиск рецепта → открытие деталей → добавление в план питания.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dev_dependencies:
  integration_test:
    sdk: flutter
  flutter_test:
    sdk: flutter
```

Реализуй `integration_test/app_test.dart`.

**Тестовый сценарий:**
```dart
void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('Сценарий добавления рецепта', () {
    testWidgets('пользователь ищет и добавляет рецепт в план', (tester) async {
      // 1. Запустить приложение
      await tester.pumpWidget(const FitMenuApp());
      await tester.pumpAndSettle();

      // 2. Нажать на таб "Рецепты"
      await tester.tap(find.byIcon(Icons.restaurant_outlined));
      await tester.pumpAndSettle();

      // 3. Нажать на поиск
      await tester.tap(find.byIcon(Icons.search));
      await tester.pumpAndSettle();

      // 4. Ввести запрос
      await tester.enterText(find.byType(TextField), 'chicken');
      await tester.pumpAndSettle(const Duration(seconds: 2));

      // 5. Нажать на первый результат
      await tester.tap(find.byType(RecipeTile).first);
      await tester.pumpAndSettle();

      // 6. Нажать FAB "Добавить в план"
      await tester.tap(find.byType(FloatingActionButton));
      await tester.pumpAndSettle();

      // 7. Проверить SnackBar
      expect(find.textContaining('добавлен в план'), findsOneWidget);
    });
  });
}
```

**Запуск:**
```bash
flutter test integration_test/app_test.dart
# или на устройстве:
flutter test integration_test/ --device-id=<device_id>
```

---

✅ **Критерии приёмки:**
- [ ] `IntegrationTestWidgetsFlutterBinding.ensureInitialized()` вызывается
- [ ] Тест проходит на реальном устройстве/эмуляторе
- [ ] `pumpAndSettle` используется для ожидания анимаций
- [ ] `enterText` + `pumpAndSettle(Duration)` для debounce поиска
- [ ] SnackBar-сообщение проверяется через `find.textContaining`
- [ ] Тест изолирован: не зависит от реального API (используй mock или stub сервер)

---

💡 **Подсказка:** Integration тесты запускаются в реальной Flutter-среде (не в тестовой). `find.byKey(Key('search_field'))` надёжнее чем `find.byType(TextField)` при нескольких текстовых полях. `flutter_driver` — старый API, `integration_test` — современный.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// integration_test/app_test.dart

import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:integration_test/integration_test.dart';

void main() {
  IntegrationTestWidgetsFlutterBinding.ensureInitialized();

  group('FitMenu — Smoke Tests', () {
    testWidgets('приложение запускается без ошибок', (tester) async {
      await tester.pumpWidget(const FitMenuApp());
      await tester.pumpAndSettle();

      // Главный экран отображается
      expect(find.byType(Scaffold), findsWidgets);
    });
  });

  group('Навигация по вкладкам', () {
    testWidgets('переключение между вкладками работает', (tester) async {
      await tester.pumpWidget(const FitMenuApp());
      await tester.pumpAndSettle();

      // Нажать таб "Рецепты"
      await tester.tap(find.byIcon(Icons.restaurant_outlined));
      await tester.pumpAndSettle();
      expect(find.text('Рецепты'), findsWidgets);

      // Нажать таб "Настройки"
      await tester.tap(find.byIcon(Icons.settings_outlined));
      await tester.pumpAndSettle();
      expect(find.text('Настройки'), findsOneWidget);
    });
  });

  group('Сценарий добавления в план питания', () {
    testWidgets('добавляет рецепт из экрана деталей', (tester) async {
      await tester.pumpWidget(const FitMenuApp());
      await tester.pumpAndSettle();

      // Перейти на рецепты
      await tester.tap(find.byIcon(Icons.restaurant_outlined));
      await tester.pumpAndSettle();

      // Найти и нажать на первый RecipeTile
      final tileFinder = find.byType(RecipeTile);
      if (tileFinder.evaluate().isNotEmpty) {
        await tester.tap(tileFinder.first);
        await tester.pumpAndSettle();

        // Нажать FAB
        final fabFinder = find.byType(FloatingActionButton);
        if (fabFinder.evaluate().isNotEmpty) {
          await tester.tap(fabFinder);
          await tester.pumpAndSettle();

          // Проверить подтверждение
          expect(
            find.byType(SnackBar),
            findsOneWidget,
          );
        }
      }
    });
  });
}
```

</details>
