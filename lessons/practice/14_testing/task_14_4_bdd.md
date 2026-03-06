# 14.4: BDD — тесты через Gherkin сценарии

> Project: FitMenu | Глава 14 — Testing

### 14.4: BDD тесты с flutter_gherkin

🎯 **Цель шага:** Написать BDD-сценарий (Behaviour-Driven Development) для функции поиска рецептов на языке Gherkin — Given/When/Then описание поведения понятное всей команде.

---

📝 **Техническое задание:**

**Зависимость:** `flutter_gherkin: ^3.0.0`

Создай файл `test_driver/features/search_recipes.feature`:

```gherkin
Feature: Поиск рецептов
  Как пользователь FitMenu
  Я хочу искать рецепты
  Чтобы найти подходящее блюдо для плана питания

  Scenario: Успешный поиск рецепта
    Given приложение запущено
    And пользователь на экране рецептов
    When пользователь вводит "курица" в поле поиска
    Then отображается список рецептов с "курица" в названии
    And каждый рецепт показывает калорийность

  Scenario: Пустой результат поиска
    Given приложение запущено
    And пользователь на экране рецептов
    When пользователь вводит "xyz123abc" в поле поиска
    Then отображается сообщение "Ничего не найдено"

  Scenario: Поиск с пустой строкой
    Given приложение запущено
    And пользователь на экране рецептов
    When пользователь очищает поле поиска
    Then отображается список популярных рецептов
```

**Step Definitions:**
```dart
// test_driver/steps/search_steps.dart

class AppIsRunningStep extends GivenWithWorld<FlutterWidgetTesterWorld> {
  @override
  Future<void> executeStep() async {
    await world.pumpWidget(const FitMenuApp());
    await world.pumpAndSettle();
  }
  @override
  RegExp get pattern => RegExp(r'приложение запущено');
}

class UserTypesInSearchStep extends WhenWithWorld<FlutterWidgetTesterWorld> {
  @override
  Future<void> executeStep(String query) async {
    await world.enterText(find.byType(TextField), query);
    await world.pumpAndSettle(const Duration(seconds: 1));
  }
  @override
  RegExp get pattern => RegExp(r'пользователь вводит "(.+)" в поле поиска');
}
```

---

✅ **Критерии приёмки:**
- [ ] `.feature` файл написан на русском в формате Gherkin (Given/When/Then)
- [ ] Каждый шаг имеет соответствующий Step Definition
- [ ] `RegExp` паттерн правильно захватывает параметры
- [ ] Сценарии покрывают happy path, empty state и edge case
- [ ] BDD тесты запускаются командой `flutter drive`

---

💡 **Подсказка:** BDD — это про коммуникацию, а не про инструменты. `feature` файлы читаются бизнесом и тестировщиками. `Given` — предусловие, `When` — действие, `Then` — ожидаемый результат. `And` продолжает предыдущий шаг того же типа.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```gherkin
# test_driver/features/search_recipes.feature

Feature: Поиск рецептов
  Как пользователь FitMenu
  Я хочу искать рецепты по названию
  Чтобы быстро находить нужное блюдо

  Background:
    Given приложение запущено
    And пользователь авторизован

  Scenario: Успешный поиск по названию
    Given пользователь находится на экране рецептов
    When пользователь вводит "куриц" в поле поиска
    Then список рецептов обновляется
    And каждый результат содержит "куриц" в названии

  Scenario Outline: Поиск с различными запросами
    Given пользователь находится на экране рецептов
    When пользователь вводит "<запрос>" в поле поиска
    Then количество результатов равно <количество>

    Examples:
      | запрос  | количество |
      | суп     | 5          |
      | xyz999  | 0          |
```

```dart
// test_driver/steps/search_steps.dart

import 'package:flutter_gherkin/flutter_gherkin.dart';
import 'package:flutter_test/flutter_test.dart';

StepDefinitionGeneric AppIsRunning() {
  return given<FlutterWidgetTesterWorld>(
    'приложение запущено',
    (world) async {
      await world.appDriver.waitForAppToSettle();
    },
  );
}

StepDefinitionGeneric UserTypesInSearch() {
  return when1<String, FlutterWidgetTesterWorld>(
    RegExp(r'пользователь вводит "(.+)" в поле поиска'),
    (query, world) async {
      final searchField = find.byKey(const Key('search_field'));
      await world.appDriver.tap(searchField);
      await world.appDriver.enterText(searchField, query);
      await world.appDriver.waitForAppToSettle(
        timeout: const Duration(seconds: 2),
      );
    },
  );
}

StepDefinitionGeneric SearchResultsNotEmpty() {
  return then<FlutterWidgetTesterWorld>(
    'список рецептов обновляется',
    (world) async {
      final list = find.byType(RecipeTile);
      await world.appDriver.waitFor(list);
      expect(list.evaluate().isNotEmpty, isTrue);
    },
  );
}
```

</details>
