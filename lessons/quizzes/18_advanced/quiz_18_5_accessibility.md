# Квиз: Accessibility

**Тема:** 18.5 — Accessibility (a11y)  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое `Semantics` виджет в Flutter?

- A) Виджет для добавления UI эффектов
- B) Виджет для предоставления семантической информации экранным читалкам (TalkBack/VoiceOver) — описание, действия, свойства элемента
- C) Система для управления темой приложения
- D) `Semantics` — только для тестов

<details>
<summary>Ответ</summary>

**B) Семантическая информация для screen readers**

```dart
// Добавить описание к иконке:
Semantics(
  label: 'Добавить в избранное',
  hint: 'Дважды нажмите для добавления',
  button: true,
  child: IconButton(
    icon: const Icon(Icons.favorite_border),
    onPressed: () => addToFavorites(),
  ),
)

// Изображение с описанием:
Semantics(
  label: 'Фото продукта: красные кроссовки Nike Air Max',
  image: true,
  child: Image.network(product.imageUrl),
)

// Скрыть от screen reader (декоративный элемент):
Semantics(
  excludeSemantics: true,
  child: const Icon(Icons.star, color: Colors.amber), // декоративная
)

// Переопределить текст (например показать "$42.00" как "42 доллара"):
Semantics(
  value: 'Цена: сорок два доллара',
  child: Text('\$42.00'),
)
```

</details>

---

### Вопрос 2 🟢

Как работают TalkBack (Android) и VoiceOver (iOS)?

- A) Это одно и то же приложение
- B) Screen readers — зачитывают текст и описания элементов; навигация свайпами; двойной тап для активации; Flutter автоматически интегрируется с ними
- C) TalkBack только для слабовидящих
- D) VoiceOver требует специальный плагин

<details>
<summary>Ответ</summary>

**B) Screen readers с навигацией через свайпы**

```
TalkBack (Android) жесты:
- Свайп вправо/влево: следующий/предыдущий элемент
- Двойной тап: активировать элемент
- Свайп вниз-вправо: открыть меню действий
- Удержание с двумя пальцами: скролл

VoiceOver (iOS) жесты:
- Одиночный свайп вправо/влево: следующий/предыдущий
- Двойной тап: активировать
- Роторный жест двумя пальцами: изменить режим навигации

Тестирование в Flutter:
- Android Emulator: Settings → Accessibility → TalkBack
- iOS Simulator: Settings → Accessibility → VoiceOver
- DevTools Widget Inspector: включить semantics view
```

```bash
# Запустить с семантическим деревом видимым:
flutter run --enable-software-rendering
```

</details>

---

### Вопрос 3 🟢

Что такое `excludeSemantics` и `MergeSemantics`?

- A) Они одинаковые
- B) `excludeSemantics: true` — скрыть виджет от screen reader; `MergeSemantics` — объединить семантику нескольких дочерних в один элемент
- C) `MergeSemantics` — только для ListView
- D) `excludeSemantics` удаляет виджет из дерева

<details>
<summary>Ответ</summary>

**B) Скрытие и объединение семантики**

```dart
// excludeSemantics: скрыть декоративный элемент:
Semantics(
  excludeSemantics: true,
  child: DecorativeIcon(),
)

// Или через ExcludeSemantics виджет:
ExcludeSemantics(
  child: DecorativeImage(),
)

// MergeSemantics: объединить кнопку и иконку в один элемент:
MergeSemantics(
  child: ListTile(
    leading: const Icon(Icons.email),  // будет объединено
    title: const Text('Отправить'),    // с этим текстом
    onTap: sendEmail,
  ),
)
// Screen reader прочитает: "Отправить, кнопка" (не "email, Отправить, кнопка")

// BlockSemantics: скрыть другие элементы когда этот активен (для диалогов):
BlockSemantics(
  blocking: isDialogOpen,
  child: mainContent,
)
```

</details>

---

### Вопрос 4 🟡

Как адаптировать UI под `textScaleFactor` / `textScaler`?

- A) Фиксировать размеры текста, игнорируя настройки системы
- B) Не использовать фиксированные высоты для контейнеров с текстом; тестировать с `textScaleFactor: 2.0`; использовать `MediaQuery.textScalerOf`
- C) `Text(style: TextStyle(fontSize: 16))` автоматически масштабируется
- D) `textScaleFactor` только для Android

<details>
<summary>Ответ</summary>

**B) Адаптивные контейнеры + тестирование с увеличенным текстом**

```dart
// ПЛОХО: фиксированная высота ломается при увеличении текста:
Container(
  height: 48,  // ← сломается при textScale = 2.0
  child: Text('Длинная надпись'),
)

// ХОРОШО: ограничение снизу + padding:
Container(
  constraints: const BoxConstraints(minHeight: 48),
  padding: const EdgeInsets.symmetric(vertical: 8),
  child: Text('Длинная надпись'),
)

// Ограничить максимальный масштаб (осторожно!):
Text(
  'Заголовок',
  textScaler: TextScaler.linear(
    MediaQuery.textScalerOf(context).scale(1.0).clamp(1.0, 2.0),
  ),
)

// Тестирование:
MediaQuery(
  data: MediaQuery.of(context).copyWith(
    textScaler: const TextScaler.linear(2.0),
  ),
  child: MyWidget(),
)
```

</details>

---

### Вопрос 5 🟡

Как обеспечить минимальный размер touch target?

- A) Любой размер виджета — достаточен
- B) Минимум 48×48 dp (WCAG) / 44×44 pt (Apple HIG); использовать `kMinInteractiveDimension`; `InkWell`/`GestureDetector` может быть больше кнопки
- C) Flutter автоматически увеличивает зоны нажатия
- D) Минимальный размер — 24×24 dp

<details>
<summary>Ответ</summary>

**B) Минимум 48×48 dp; `kMinInteractiveDimension`**

```dart
// kMinInteractiveDimension = 48.0

// IconButton по умолчанию 48×48:
IconButton(
  onPressed: () {},
  icon: const Icon(Icons.add, size: 24), // иконка 24, зона 48×48
)

// Маленькая кнопка с большой зоной нажатия:
SizedBox(
  width: kMinInteractiveDimension,
  height: kMinInteractiveDimension,
  child: Center(
    child: SmallButton(size: 24),
  ),
)

// GestureDetector расширить:
GestureDetector(
  onTap: onTap,
  behavior: HitTestBehavior.opaque, // захватывать весь размер
  child: Padding(
    padding: const EdgeInsets.all(12), // увеличить зону
    child: const Icon(Icons.close, size: 24),
  ),
)

// Проверить в DevTools:
// Widget Inspector → Enable highlight oversized images
```

</details>

---

### Вопрос 6 🟡

Как добеспечить достаточный контраст цветов (WCAG)?

- A) Любой контраст приемлем
- B) WCAG AA: 4.5:1 для обычного текста, 3:1 для крупного (>= 18pt); WCAG AAA: 7:1 и 4.5:1
- C) Контраст — только для цветных слепых
- D) Flutter автоматически проверяет контраст

<details>
<summary>Ответ</summary>

**B) WCAG AA (4.5:1 для текста) как минимальный стандарт**

```dart
// Проверить контраст программно:
double getContrastRatio(Color foreground, Color background) {
  final l1 = foreground.computeLuminance();
  final l2 = background.computeLuminance();
  final lighter = math.max(l1, l2);
  final darker = math.min(l1, l2);
  return (lighter + 0.05) / (darker + 0.05);
}

// Пример:
// Чёрный на белом: 21:1 ✅ (AAA)
// #767676 на белом: 4.54:1 ✅ (AA)
// #949494 на белом: 3.03:1 ❌ (не проходит AA для мелкого текста)

// Тест контраста в widget test:
testWidgets('виджет соответствует WCAG AA', (tester) async {
  await tester.pumpWidget(
    MaterialApp(home: MyWidget()),
  );
  await expectLater(
    tester,
    meetsGuideline(textContrastGuideline),  // WCAG AA
  );
});

// Использовать онлайн инструменты:
// WebAIM Contrast Checker: https://webaim.org/resources/contrastchecker/
// Material Theme Builder проверяет автоматически
```

</details>

---

### Вопрос 7 🟡

Как добавить `Semantics` для кастомных интерактивных виджетов?

- A) Кастомные виджеты не нуждаются в семантике
- B) Добавить `Semantics` с правильными свойствами: `button`, `slider`, `checked`, `selected`, `enabled`; переопределить `debugFillProperties`
- C) Только GestureDetector автоматически добавляет семантику
- D) `CustomWidget(semantics: 'label')`

<details>
<summary>Ответ</summary>

**B) `Semantics` с точными свойствами для роли виджета**

```dart
// Кастомный toggle button:
class CustomToggle extends StatelessWidget {
  final bool isOn;
  final VoidCallback onChanged;
  final String label;

  const CustomToggle({required this.isOn, required this.onChanged,
      required this.label});

  @override
  Widget build(BuildContext context) {
    return Semantics(
      label: label,
      toggled: isOn,      // для toggle/switch
      button: true,
      onTap: onChanged,   // добавить действие
      // Для slider:
      // slider: true,
      // value: '50%',
      // increasedValue: '55%',
      // decreasedValue: '45%',
      // onIncrease: increaseValue,
      // onDecrease: decreaseValue,
      child: GestureDetector(
        onTap: onChanged,
        child: AnimatedContainer(
          duration: const Duration(milliseconds: 200),
          color: isOn ? Colors.green : Colors.grey,
          padding: const EdgeInsets.all(16),
          child: Text(label),
        ),
      ),
    );
  }
}
```

</details>

---

### Вопрос 8 🔴

Как правильно управлять фокусом клавиатуры для accessibility?

- A) Фокус управляется автоматически Flutter
- B) `FocusNode` + `FocusTraversalGroup` для порядка табуляции; `FocusScope.of(context).requestFocus()` для программного управления; `autofocus: true`
- C) `Tab` всегда обрабатывается операционной системой
- D) Фокус только для веб-приложений

<details>
<summary>Ответ</summary>

**B) `FocusNode` + `FocusTraversalGroup` + программное управление**

```dart
class LoginForm extends StatefulWidget {
  @override State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _emailFocus = FocusNode();
  final _passwordFocus = FocusNode();
  final _submitFocus = FocusNode();

  @override
  void dispose() {
    _emailFocus.dispose();
    _passwordFocus.dispose();
    _submitFocus.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return FocusTraversalGroup(
      // Порядок обхода: сверху вниз, слева направо:
      policy: OrderedTraversalPolicy(),
      child: Column(children: [
        FocusTraversalOrder(
          order: const NumericFocusOrder(1),
          child: TextField(
            focusNode: _emailFocus,
            autofocus: true,  // первый фокус
            textInputAction: TextInputAction.next, // Tab/Next
            onSubmitted: (_) => _passwordFocus.requestFocus(),
          ),
        ),
        FocusTraversalOrder(
          order: const NumericFocusOrder(2),
          child: TextField(
            focusNode: _passwordFocus,
            textInputAction: TextInputAction.done,
            onSubmitted: (_) => _submitFocus.requestFocus(),
          ),
        ),
        FocusTraversalOrder(
          order: const NumericFocusOrder(3),
          child: ElevatedButton(
            focusNode: _submitFocus,
            onPressed: submit,
            child: const Text('Войти'),
          ),
        ),
      ]),
    );
  }
}
```

</details>

---

### Вопрос 9 🔴

Как адаптировать анимации для пользователей с вестибулярными расстройствами?

- A) Анимации всегда нужны для хорошего UX
- B) Проверять `MediaQuery.disableAnimations`; если `true` — убирать/сокращать анимации; использовать `accessibleNavigation` флаг
- C) Только iOS поддерживает "Reduce Motion"
- D) `AnimationController` автоматически адаптируется

<details>
<summary>Ответ</summary>

**B) `MediaQuery.disableAnimations` + адаптация**

```dart
// Проверить настройку "Reduce Motion":
final reduceMotion = MediaQuery.disableAnimationsOf(context);
// или:
final settings = MediaQuery.accessibleNavigationOf(context);

// Адаптивная анимация:
class AdaptiveAnimation extends StatelessWidget {
  final Widget child;

  @override
  Widget build(BuildContext context) {
    final reduceMotion = MediaQuery.disableAnimationsOf(context);

    return AnimatedSwitcher(
      duration: reduceMotion
          ? Duration.zero              // без анимации
          : const Duration(milliseconds: 300), // нормальная
      child: child,
    );
  }
}

// Hero анимация:
@override
Widget build(BuildContext context) {
  final reduceMotion = MediaQuery.disableAnimationsOf(context);

  if (reduceMotion) {
    return child; // без Hero
  }
  return Hero(tag: heroTag, child: child);
}

// Параллакс и сложные motion effects:
// Полностью отключать при reduceMotion = true
// AnimationController.duration = reduceMotion ? Duration.zero : normalDuration
```

</details>

---

### Вопрос 10 🔴

Как провести accessibility audit Flutter приложения?

- A) Только ручное тестирование с TalkBack/VoiceOver
- B) `meetsGuideline()` в widget тестах; `Accessibility Scanner` (Android); `Accessibility Inspector` (Xcode); DevTools semantics tree; ручное тестирование
- C) Audit доступен только в платных инструментах
- D) `flutter analyze --accessibility`

<details>
<summary>Ответ</summary>

**B) Комплексный подход: автотесты + нативные инструменты + ручное тестирование**

```dart
// 1. Widget тесты с accessibility guidelines:
testWidgets('accessibility audit', (tester) async {
  await tester.pumpWidget(const MaterialApp(home: MyScreen()));

  // WCAG AA контраст:
  await expectLater(tester, meetsGuideline(textContrastGuideline));

  // Минимальный размер touch target:
  await expectLater(tester, meetsGuideline(iOSTapTargetGuideline));
  await expectLater(tester, meetsGuideline(androidTapTargetGuideline));

  // Labeled тaps:
  await expectLater(tester, meetsGuideline(labeledTapTargetGuideline));
});

// 2. DevTools — включить семантическое дерево:
// Widget Inspector → Show Semantics Debugger (глаз с иконкой)

// 3. Flutter диагностика:
// flutter run → 's' → showSemanticsDebugger toggle

// 4. Checklist:
// ✅ Все интерактивные элементы имеют Semantics label
// ✅ Изображения имеют alt-текст или excludeSemantics
// ✅ Цвета не единственный носитель информации
// ✅ Touch targets >= 48×48 dp
// ✅ Текст масштабируется (не фиксированные высоты)
// ✅ Логический порядок фокуса
// ✅ Поддержка Reduce Motion
```

</details>
