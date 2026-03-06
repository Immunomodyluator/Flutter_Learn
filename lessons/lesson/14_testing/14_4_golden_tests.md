# 14.4 Golden-тесты

## 1. Суть

**Golden-тест** (визуальный регрессионный тест) сохраняет эталонный скриншот виджета в PNG-файл (golden file) и при каждом запуске сравнивает актуальный рендеринг с эталоном. Если пиксели отличаются — тест падает.

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  golden_toolkit: ^0.15.0
```

---

## 2. Базовый golden-тест

```dart
// test/golden/user_avatar_golden_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  testWidgets('UserAvatar golden test', (tester) async {
    await tester.pumpWidget(
      MaterialApp(
        home: Scaffold(
          body: UserAvatar(
            imageUrl: 'https://example.com/avatar.jpg',
            name: 'Ivan',
            size: 64,
          ),
        ),
      ),
    );

    // Создать/сравнить с golden-файлом
    await expectLater(
      find.byType(UserAvatar),
      matchesGoldenFile('goldens/user_avatar.png'),
    );
  });
}
```

```bash
# Первый запуск — создаёт эталонные PNG
flutter test --update-goldens test/golden/

# Последующие запуски — сравниваются с эталоном
flutter test test/golden/
```

---

## 3. golden_toolkit — расширенные возможности

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:golden_toolkit/golden_toolkit.dart';

void main() {
  // Загрузить все шрифты перед тестами (важно!)
  setUpAll(() async {
    await loadAppFonts();
  });

  testGoldens('UserCard — все сценарии', (tester) async {
    final builder = GoldenBuilder.column()
      ..addScenario(
        'Обычный пользователь',
        UserCard(user: User(id: '1', name: 'Ivan', isPremium: false)),
      )
      ..addScenario(
        'Премиум пользователь',
        UserCard(user: User(id: '2', name: 'Anna', isPremium: true)),
      )
      ..addScenario(
        'Длинное имя',
        UserCard(user: User(id: '3', name: 'Очень Длинное Имя Пользователя', isPremium: false)),
      );

    await tester.pumpWidgetBuilder(builder.build());
    await screenMatchesGolden(tester, 'user_card_scenarios');
  });
}
```

---

## 4. Несколько размеров экрана

```dart
testGoldens('LoginScreen на разных устройствах', (tester) async {
  await tester.pumpWidgetBuilder(
    const LoginScreen(),
    wrapper: materialAppWrapper(theme: AppTheme.light),
  );

  await multiScreenGolden(
    tester,
    'login_screen',
    devices: [
      Device.phone,
      Device.tabletLandscape,
      const Device(
        name: 'large_phone',
        size: Size(414, 896),
        devicePixelRatio: 3.0,
      ),
    ],
  );
});
```

---

## 5. Светлая и тёмная тема

```dart
testGoldens('Тема: светлая и тёмная', (tester) async {
  final builder = GoldenBuilder.column()
    ..addScenario(
      'Светлая',
      Theme(data: ThemeData.light(), child: const MyWidget()),
    )
    ..addScenario(
      'Тёмная',
      Theme(data: ThemeData.dark(), child: const MyWidget()),
    );

  await tester.pumpWidgetBuilder(builder.build());
  await screenMatchesGolden(tester, 'my_widget_themes');
});
```

---

## 6. Работа с сетевыми изображениями

```dart
// Мок HTTP для изображений
testGoldens('Аватар с изображением', (tester) async {
  // Использовать локальный asset или мок NetworkImage
  await tester.pumpWidgetBuilder(
    UserAvatar(imageUrl: 'test_asset/avatar.png'),
    wrapper: materialAppWrapper(),
  );
  await screenMatchesGolden(tester, 'user_avatar_with_image');
});
```

---

## 7. Структура golden-файлов

```
test/
  golden/
    goldens/                          ← PNG-файлы, коммитятся в git
      user_avatar.png
      user_card_scenarios.png
      login_screen_phone.png
      login_screen_tablet_landscape.png
    user_avatar_golden_test.dart
    user_card_golden_test.dart
    login_screen_golden_test.dart
```

---

## 8. Golden-тесты в CI

```yaml
# GitHub Actions: сравнение, не генерация
- name: Run golden tests
  run: flutter test test/golden/
  # НЕ добавляй --update-goldens в CI!

# Если тест упал — скачать diff-изображение как артефакт
- name: Upload failure artifacts
  if: failure()
  uses: actions/upload-artifact@v3
  with:
    name: golden-failures
    path: test/golden/failures/
```

---

## 9. Обновление эталона

```bash
# Обновить все golden-файлы
flutter test --update-goldens test/golden/

# Обновить конкретный файл
flutter test --update-goldens test/golden/user_avatar_golden_test.dart
```

Обновление нужно коммитить отдельно с описанием "chore: update golden files after redesign".

---

## 10. Под капотом

- Flutter рендерит виджет в off-screen canvas и сохраняет PNG.
- Сравнение — попиксельное.
- `matchesGoldenFile` при первом запуске с несуществующим файлом — создаёт его.
- **Проблема**: golden-файлы зависят от платформы и шрифтов. Разные ОС дают разный рендеринг.

---

## 11. Решение проблемы кросс-платформенности

```bash
# Все в команде должны генерировать goldens на одной платформе
# Или использовать Docker:

# Генерация goldens в Linux-контейнере
docker run -v $(pwd):/app flutter/flutter flutter test --update-goldens

# CI всегда запускает на Linux — нет расхождений
```

---

## 12. Типичные ошибки

| Ошибка                           | Причина                    | Решение                                        |
| -------------------------------- | -------------------------- | ---------------------------------------------- |
| Тест падает после смены шрифта   | Пиксели изменились         | Запустить `--update-goldens` и закоммитить     |
| Разные результаты на Mac и Linux | Разный рендеринг платформ  | Генерировать goldens только на одной ОС / в CI |
| Golden не создаётся              | Нет папки `goldens/`       | Создать папку или добавить `.gitkeep`          |
| Шрифт не загружается             | Не вызван `loadAppFonts()` | Вызвать в `setUpAll`                           |

---

## 13. Рекомендации

1. **Golden-тесты для дизайн-системы**: кнопки, карточки, типографика — не для каждого экрана.
2. **`loadAppFonts()`** всегда в `setUpAll` — иначе Flutter использует fallback-шрифт.
3. **Коммить goldens в git** — это документация внешнего вида компонентов.
4. **Обновлять отдельным PR** с визуальным сравнением.
5. **Одна платформа для генерации** — зафикси в README команды (`флаттер тест --update-goldens` только на Linux/CI).
