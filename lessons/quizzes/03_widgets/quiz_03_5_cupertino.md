# Квиз: Cupertino Widgets

**Тема:** 03.5 — Cupertino Widgets  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой виджет является Cupertino аналогом `MaterialApp`?

- A) `IosApp`
- B) `CupertinoApp`
- C) `AppleApp`
- D) `CupertinoWidget`

<details>
<summary>Ответ</summary>

**B) `CupertinoApp`**

`CupertinoApp` — корневой виджет iOS-стиль приложения. Задаёт `CupertinoThemeData` вместо `ThemeData`. Можно использовать `MaterialApp` + отдельные Cupertino виджеты или полностью на Cupertino.

</details>

---

### Вопрос 2 🟢

Что такое `CupertinoNavigationBar`?

- A) Нижняя панель вкладок (tab bar)
- B) iOS-стиль верхняя навигационная панель с заголовком
- C) Боковое меню
- D) Переход между экранами

<details>
<summary>Ответ</summary>

**B) iOS-стиль верхняя навигационная панель с заголовком**

`CupertinoNavigationBar` — аналог `AppBar` в iOS стиле: blur-фон, шрифт SF Pro, кнопка «назад» со стрелкой. `CupertinoSliverNavigationBar` — с large title (коллапсирует при скролле), как в iOS Settings.

</details>

---

### Вопрос 3 🟢

Чем `CupertinoSwitch` отличается от Material `Switch`?

- A) `CupertinoSwitch` зелёного цвет, `Switch` — синего
- B) `CupertinoSwitch` стилизован под iOS (pill-shape, нет ripple); `Switch` — Material стиль
- C) `CupertinoSwitch` однонаправленный
- D) Только эстетическое различие

<details>
<summary>Ответ</summary>

**B) `CupertinoSwitch` стилизован под iOS (pill-shape, нет ripple); `Switch` — Material стиль**

`CupertinoSwitch` — точная копия iOS toggle: зелёный цвет по умолчанию, скруглённая форма, плавная анимация без Material ink ripple. `Switch.adaptive()` автоматически рендерит нужный вариант по платформе.

</details>

---

### Вопрос 4 🟡

Что такое `CupertinoPicker` и где он используется?

- A) Компонент выбора файлов
- B) Барабанный (wheel) пикер для выбора значения из списка — использован в `showCupertinoModalPopup`
- C) Выпадающий список в iOS стиле
- D) Диалог выбора цвета

<details>
<summary>Ответ</summary>

**B) Барабанный (wheel) пикер для выбора значения из списка — использован в `showCupertinoModalPopup`**

`CupertinoPicker` — прокручиваемый барабан как в iOS. Используется для дат (`CupertinoDatePicker`), времени, кастомных списков. Показывается через `showCupertinoModalPopup` в нижней части экрана.

</details>

---

### Вопрос 5 🟡

Как отобразить iOS-стиль диалог (ActionSheet)?

- A) `showCupertinoDialog(context, builder: ...)`
- B) `showCupertinoModalPopup(context: context, builder: (_) => CupertinoActionSheet(...))`
- C) `showIosSheet(context)`
- D) `CupertinoDialog.show(context)`

<details>
<summary>Ответ</summary>

**B) `showCupertinoModalPopup(context: context, builder: (_) => CupertinoActionSheet(...))`**

`CupertinoActionSheet` — iOS-стиль action sheet, появляющийся снизу. Содержит `title`, `message`, `actions` (список `CupertinoActionSheetAction`) и `cancelButton`. `showCupertinoDialog` — для alert-диалогов (по центру экрана).

</details>

---

### Вопрос 6 🟡

Что делает `CupertinoTabScaffold`?

- A) Создаёт вкладки с Material стилем
- B) iOS-стиль скаффолд с нижними вкладками (`CupertinoTabBar`) и отдельным навигационным стеком для каждой
- C) Горизонтальное прокручиваемое меню вкладок
- D) Scaffold с боковой навигацией

<details>
<summary>Ответ</summary>

**B) iOS-стиль скаффолд с нижними вкладками (`CupertinoTabBar`) и отдельным навигационным стеком для каждой**

`CupertinoTabScaffold` + `CupertinoTabBar` — поведение как в родных iOS приложениях: каждая вкладка имеет свой независимый `CupertinoNavigator` стек. Смена вкладки не теряет навигационную историю каждой.

</details>

---

### Вопрос 7 🟡

Как адаптивно выбрать Material или Cupertino виджет?

- A) `Platform.isIOS ? CupertinoWidget() : MaterialWidget()`
- B) Через `.adaptive()` конструктор или `PlatformWidget` из сторонних пакетов
- C) Только ручной `if/else`
- D) Flutter делает это автоматически

<details>
<summary>Ответ</summary>

**B) Через `.adaptive()` конструктор или `PlatformWidget` из сторонних пакетов**

Flutter предоставляет `Switch.adaptive()`, `Slider.adaptive()`, `CircularProgressIndicator.adaptive()`. Для полного покрытия — пакет `flutter_platform_widgets`. `Platform.isIOS` тоже работает, но менее элегантно.

</details>

---

### Вопрос 8 🔴

В чём разница `CupertinoPageRoute` и `MaterialPageRoute`?

- A) Они идентичны
- B) `CupertinoPageRoute` — горизонтальный слайд с жестом «назад» от края; `MaterialPageRoute` — вертикальный слайд снизу (Android)
- C) `CupertinoPageRoute` работает только на iOS
- D) `MaterialPageRoute` не поддерживает жест назад

<details>
<summary>Ответ</summary>

**B) `CupertinoPageRoute` — горизонтальный слайд с жестом «назад» от края; `MaterialPageRoute` — вертикальный слайд снизу (Android)**

iOS паттерн: следующий экран въезжает справа, жест от левого края экрана — «назад» с интерактивным переходом. Android: экран появляется снизу вверх. `PageRouteBuilder` — кастомные переходы.

</details>

---

### Вопрос 9 🔴

Что такое `CupertinoThemeData` и как переключить тему для Cupertino приложения?

- A) Тема не поддерживается в Cupertino
- B) Специфичная tema: `primaryColor`, шрифт SF, `brightness`. Передаётся в `CupertinoApp(theme: ...)`
- C) Идентична `ThemeData`
- D) `CupertinoThemeData` — только для тёмной темы iOS

<details>
<summary>Ответ</summary>

**B) Специфичная tema: `primaryColor`, шрифт SF, `brightness`. Передаётся в `CupertinoApp(theme: ...)`**

```dart
CupertinoApp(
  theme: const CupertinoThemeData(
    primaryColor: CupertinoColors.systemBlue,
    brightness: Brightness.dark,
  ),
)
```

`CupertinoTheme.of(context)` — чтение. При использовании `MaterialApp` + Cupertino виджетов — `CupertinoTheme` извлекает данные из `ThemeData`.

</details>

---

### Вопрос 10 🔴

Что такое iOS-стиль «транслюцентное» навигационное дерево в Cupertino?

- A) Прозрачный NavBar
- B) Каждый маршрут в стеке видно через полупрозрачный следующий экран при анимации перехода
- C) Фоновые виджеты рендерятся сквозь экран
- D) Эффект frosted glass на контенте

<details>
<summary>Ответ</summary>

**B) Каждый маршрут в стеке видно через полупрозрачный следующий экран при анимации перехода**

При `CupertinoPageRoute` предыдущий экран остаётся видимым (со смещением влево и затемнением) во время слайд-анимации. Это обеспечивает `CupertinoRouteTransitionMixin`. `BackdropFilter` + `ImageFilter.blur` — для frosted glass эффекта в кастомных виджетах.

</details>
