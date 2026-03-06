# 14. Тестирование Flutter-приложений

## 1. Зачем тестировать?

Тесты — страховая сетка при рефакторинге, документация поведения системы и способ найти баги до деплоя.

---

## 2. Пирамида тестирования

```
        ▲ Integration Tests   (медленные, мало)
       ███
      █████ Widget Tests       (средние)
     ███████
    █████████ Unit Tests       (быстрые, много)
```

| Тип             | Что проверяет                      | Скорость     | Инструмент                    |
| --------------- | ---------------------------------- | ------------ | ----------------------------- |
| **Unit**        | Логика класса / функции            | Очень быстро | `dart test`, `mockito`        |
| **Widget**      | UI компонент                       | Быстро       | `flutter test`, `testWidgets` |
| **Integration** | Полный поток в реальном устройстве | Медленно     | `integration_test`, `patrol`  |
| **Golden**      | Визуальный снимок виджета          | Средне       | `golden_toolkit`              |

---

## 3. Зависимости

```yaml
dev_dependencies:
  flutter_test:
    sdk: flutter

  # Мокирование
  mocktail: ^1.0.4 # рекомендуется (без кодогенерации)
  # или mockito: ^5.4.4 (с кодогенерацией)

  # Интеграционные тесты
  integration_test:
    sdk: flutter

  # Golden тесты
  golden_toolkit: ^0.15.0

  # Дополнительно
  fake_async: ^1.3.1 # управление временем в тестах
```

---

## 4. Запуск тестов

```bash
# Unit + Widget тесты
flutter test

# Конкретный файл
flutter test test/unit/user_repository_test.dart

# С coverage
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html

# Интеграционные тесты
flutter test integration_test/

# Только тесты, имя которых содержит 'login'
flutter test --name 'login'
```

---

## 5. Структура тестовой папки

```
test/
  unit/
    repositories/
      user_repository_test.dart
    usecases/
      get_user_use_case_test.dart
    viewmodels/
      profile_view_model_test.dart
  widget/
    screens/
      login_screen_test.dart
    widgets/
      user_avatar_test.dart
  golden/
    goldens/                   ← эталонные PNG (коммитятся в git)
    login_screen_golden_test.dart
  helpers/
    test_helpers.dart          ← общие утилиты и фикстуры
integration_test/
  app_test.dart
```

---

## 6. Обзор тем раздела

| Файл                        | Тема                                                |
| --------------------------- | --------------------------------------------------- |
| `14_1_unit_tests.md`        | Unit-тесты: `test`, `expect`, `mocktail`            |
| `14_2_widget_tests.md`      | Widget-тесты: `testWidgets`, `pumpWidget`, `Finder` |
| `14_3_integration_tests.md` | Интеграционные тесты: `integration_test`, `patrol`  |
| `14_4_golden_tests.md`      | Golden-тесты: визуальное регрессионное тестирование |

---

## 7. Рекомендации

1. **Unit тестов — больше всего**: они быстрые и надёжные.
2. **Widget тесты** — для ключевых UI-компонентов и форм.
3. **Интеграционные** — для критических пользовательских сценариев (логин, checkout).
4. **Tестируй поведение, не реализацию** — тест не должен ломаться при рефакторинге, если поведение не изменилось.
5. **Coverage ≥ 80%** для бизнес-логики (Domain-слой); для UI — достаточно ключевых сценариев.
