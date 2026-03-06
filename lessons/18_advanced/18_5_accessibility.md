# 18.5 Accessibility (Доступность)

## 1. Суть

**Accessibility (a11y)** — способность приложения быть использованным людьми с ограниченными возможностями:

- слабовидящими или незрячими (скринридеры TalkBack / VoiceOver)
- людьми с ограниченной моторикой (большой текст, переключение фокуса)
- людьми с цветовой слепотой (контрастность)

Flutter поддерживает a11y через семантическое дерево, параллельное дереву виджетов.

---

## 2. Семантика Flutter

```
Flutter Widget Tree
       ↓
Semantics Tree (SemanticsNode)
       ↓
Accessibility Services (TalkBack, VoiceOver)
```

Большинство стандартных Flutter-виджетов уже предоставляют корректную семантику. Кастомные виджеты требуют ручного добавления.

---

## 3. Виджет Semantics

```dart
Semantics(
  label: 'Кнопка отправки формы',     // что озвучит скринридер
  hint: 'Нажмите дважды для отправки', // подсказка действия
  button: true,                         // роль элемента
  enabled: isFormValid,                 // состояние
  onTap: _submitForm,
  child: CustomButton(label: 'Отправить'),
)
```

### Параметры Semantics

```dart
Semantics(
  // Текстовое содержимое
  label: 'Описание',          // основной текст для скринридера
  value: '42 из 100',         // текущее значение (для Slider, ProgressBar)
  hint: 'Подсказка действия', // дополнительная инструкция

  // Роли
  button: true,       // кнопка
  link: true,         // ссылка
  image: true,        // изображение
  header: true,       // заголовок
  textField: true,    // поле ввода
  checked: false,     // CheckBox / Switch
  selected: true,     // Radio, Tab

  // Состояние
  enabled: true,
  focused: false,
  readOnly: false,
  obscured: false,    // скрытый текст (пароль)

  // Действия
  onTap: () {},
  onLongPress: () {},
  onScrollUp: () {},
  onScrollDown: () {},

  child: widget,
)
```

---

## 4. semanticLabel для изображений

```dart
// ПЛОХО — изображение без описания
Image.asset('assets/avatar.png')

// ХОРОШО
Image.asset(
  'assets/avatar.png',
  semanticLabel: 'Аватар пользователя Иван Иванов',
)

// Декоративное изображение — исключить из семантики
Image.asset(
  'assets/background.png',
  semanticLabel: '', // пустая строка — skipped by screen reader
  excludeFromSemantics: true,
)
```

---

## 5. ExcludeSemantics — убрать из дерева

```dart
// Декоративные элементы, дублирующие текст
ExcludeSemantics(
  child: Icon(Icons.star), // иконка рядом с текстом "Избранное"
)

// Весь контейнер без семантики
ExcludeSemantics(
  child: AnimatedBackground(), // декоративная анимация
)
```

---

## 6. MergeSemantics — объединить в один элемент

```dart
// Карточка с несколькими элементами → один семантический блок
MergeSemantics(
  child: Row(
    children: [
      Icon(Icons.email),
      const SizedBox(width: 8),
      Text('ivan@example.com'),
    ],
  ),
)
// Скринридер озвучит: "ivan@example.com" (не "email ivan@example.com")
```

---

## 7. Текстовый масштаб — поддержка крупного текста

```dart
// ПЛОХО — фиксированный размер контейнера
SizedBox(
  height: 48,
  child: Text('Кнопка'),
)

// ХОРОШО — гибкий размер
Padding(
  padding: const EdgeInsets.all(12),
  child: Text('Кнопка'),
)

// Ограничить максимальный масштаб (если нужно)
Text(
  'Заголовок',
  style: const TextStyle(fontSize: 24),
  textScaler: TextScaler.linear(
    MediaQuery.textScalerOf(context).scale(1.0).clamp(1.0, 1.5),
  ),
)
```

---

## 8. Контрастность цветов

```dart
// Минимальный контраст по WCAG AA: 4.5:1 для обычного текста
// Использовать ColorScheme с достаточным контрастом

// ПЛОХО — серый текст на белом
Text('Заголовок', style: TextStyle(color: Color(0xFFCCCCCC)))

// ХОРОШО — тёмный на светлом
Text('Заголовок', style: TextStyle(color: Color(0xFF212121)))

// Проверить контраст программно
double contrastRatio(Color bg, Color fg) {
  final bgLum = bg.computeLuminance();
  final fgLum = fg.computeLuminance();
  final lighter = max(bgLum, fgLum);
  final darker = min(bgLum, fgLum);
  return (lighter + 0.05) / (darker + 0.05);
}
// ratio >= 4.5 → AA, >= 7.0 → AAA
```

---

## 9. Focus Management — управление фокусом

```dart
// Явное управление фокусом при навигации
class LoginForm extends StatefulWidget {
  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _emailFocus = FocusNode();
  final _passwordFocus = FocusNode();

  @override
  void dispose() {
    _emailFocus.dispose();
    _passwordFocus.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        TextField(
          focusNode: _emailFocus,
          decoration: const InputDecoration(labelText: 'Email'),
          textInputAction: TextInputAction.next,
          onSubmitted: (_) => _passwordFocus.requestFocus(), // перейти к паролю
        ),
        TextField(
          focusNode: _passwordFocus,
          decoration: const InputDecoration(labelText: 'Пароль'),
          obscureText: true,
          textInputAction: TextInputAction.done,
          onSubmitted: (_) => _submit(),
        ),
      ],
    );
  }
}
```

---

## 10. Semantics для кастомных виджетов

```dart
// Кастомный Slider
class AccessibleSlider extends StatelessWidget {
  final double value;
  final ValueChanged<double> onChanged;

  const AccessibleSlider({super.key, required this.value, required this.onChanged});

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: 'Громкость',
      value: '${(value * 100).round()}%',
      hint: 'Проведите вверх или вниз для изменения',
      onIncrease: () => onChanged((value + 0.1).clamp(0.0, 1.0)),
      onDecrease: () => onChanged((value - 0.1).clamp(0.0, 1.0)),
      child: CustomSliderPainter(value: value, onChanged: onChanged),
    );
  }
}
```

---

## 11. Тестирование доступности

```dart
// Запустить Semantics Debugger
void main() {
  runApp(
    SemanticsDebugger(
      child: MyApp(),
    ),
  );
}

// Unit-тест семантики
testWidgets('кнопка имеет корректный semantic label', (tester) async {
  await tester.pumpWidget(
    MaterialApp(
      home: Scaffold(
        body: ElevatedButton(
          onPressed: () {},
          child: const Text('Отправить'),
        ),
      ),
    ),
  );

  expect(
    tester.getSemantics(find.byType(ElevatedButton)),
    matchesSemantics(
      label: 'Отправить',
      isButton: true,
      hasTapAction: true,
    ),
  );
});
```

```bash
# Проверить на реальном устройстве
# Android: включить TalkBack (Настройки → Accessibility → TalkBack)
# iOS: включить VoiceOver (Настройки → Accessibility → VoiceOver)
```

---

## 12. Accessibility Inspector (DevTools)

```bash
# Запустить с Semantics Inspector
flutter run --enable-software-rendering
# В DevTools → Inspector → Show Semantics Nodes
```

---

## 13. Чеклист доступности

| Критерий                               | Как проверить          |
| -------------------------------------- | ---------------------- |
| Все интерактивные элементы имеют label | SemanticsDebugger      |
| Изображения имеют semanticLabel        | Code review            |
| Контраст >= 4.5:1                      | Contrast ratio tool    |
| Поддержка крупного текста (x2)         | iOS: Настройки → Текст |
| Порядок фокуса логичен                 | TalkBack/VoiceOver     |
| Декоративные элементы исключены        | ExcludeSemantics       |

---

## 14. Типичные ошибки

| Ошибка                                          | Исправление                                            |
| ----------------------------------------------- | ------------------------------------------------------ |
| Icon без текста и без label                     | `Semantics(label: ..., child: Icon(...))`              |
| Несколько `Text` в Row озвучиваются отдельно    | `MergeSemantics` вокруг Row                            |
| Фиксированная высота обрезает увеличенный текст | Убрать `SizedBox(height: ...)`, использовать `Padding` |
| Декоративная анимация отвлекает скринридер      | `ExcludeSemantics`                                     |
| Пустое изображение без `excludeFromSemantics`   | Добавить `excludeFromSemantics: true`                  |

---

## 15. Рекомендации

1. **Тестировать с TalkBack/VoiceOver** на реальных устройствах — автотесты не заменят живое взаимодействие.
2. **`SemanticsDebugger`** в debug-режиме — быстрая проверка семантического дерева.
3. **Роли важны** — `button: true`, `header: true` меняют поведение скринридера.
4. **Порядок фокуса** должен быть логичным → задать через `FocusTraversalGroup` при необходимости.
5. **WCAG 2.1 AA** — целевой уровень соответствия для большинства продуктов.
