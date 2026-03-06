# Квиз: Material 3 и ThemeData

**Тема:** 05.1 — Material 3  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как включить Material 3 в Flutter приложении?

- A) `MaterialApp(material3: true)`
- B) `ThemeData(useMaterial3: true)`
- C) `MaterialApp(theme: ThemeData(useMaterial3: true))`
- D) Material 3 включён по умолчанию с Flutter 3.0

<details>
<summary>Ответ</summary>

**C) `MaterialApp(theme: ThemeData(useMaterial3: true))`**

Начиная с Flutter 3.16, `useMaterial3: true` является **значением по умолчанию**. Раньше нужно было явно включить. `ThemeData(useMaterial3: true)` также автоматически меняет форму компонентов, типографику и систему цветов (ColorScheme).

</details>

---

### Вопрос 2 🟢

Что такое `ColorScheme.fromSeed`?

- A) Генерирует случайную цветовую схему
- B) Создаёт Material 3 цветовую схему из одного seed-цвета с помощью Material Color Utilities
- C) Импортирует цветовую схему из JSON
- D) Копирует цветовую схему другого Material компонента

<details>
<summary>Ответ</summary>

**B) Создаёт Material 3 цветовую схему из одного seed-цвета с помощью Material Color Utilities**

`ColorScheme.fromSeed(seedColor: Colors.deepPurple)` генерирует все 30+ цветовых ролей (primary, secondary, tertiary, error, surface, onPrimary...) с правильным контрастом и гармонией. Гарантирует WCAG-совместимый контраст.

</details>

---

### Вопрос 3 🟢

Как применить тему во всём приложении?

- A) Передать `ThemeData` в `MaterialApp(theme: ...)`
- B) Использовать `Theme` виджет в каждом компоненте
- C) `ThemeProvider.set(ThemeData(...))`
- D) `GlobalTheme.instance.apply(...)`

<details>
<summary>Ответ</summary>

**A) Передать `ThemeData` в `MaterialApp(theme: ...)`**

`MaterialApp(theme: ThemeData(...), darkTheme: ThemeData(...), themeMode: ThemeMode.system)` — задаёт глобальную тему. Локальная тема для поддерева: `Theme(data: Theme.of(context).copyWith(...), child: ...)`.

</details>

---

### Вопрос 4 🟡

Как получить цвет `primary` текущей темы?

- A) `Colors.primary`
- B) `Theme.of(context).primaryColor`
- C) `Theme.of(context).colorScheme.primary`
- D) `context.theme.primary`

<details>
<summary>Ответ</summary>

**C) `Theme.of(context).colorScheme.primary`**

В Material 3 рекомендуется использовать `colorScheme` вместо прямых свойств `ThemeData`. `colorScheme.primary`, `colorScheme.onPrimary`, `colorScheme.surface`, `colorScheme.error` и т.д. `Theme.of(context).primaryColor` — устаревший M2-подход.

</details>

---

### Вопрос 5 🟡

Как кастомизировать стиль всех `ElevatedButton` в теме?

- A) Создать свой класс `ElevatedButton`
- B) `ThemeData(elevatedButtonTheme: ElevatedButtonThemeData(style: ButtonStyle(...)))`
- C) `ButtonTheme(buttonColor: Colors.blue)`
- D) `Theme.of(context).setButtonStyle(...)`

<details>
<summary>Ответ</summary>

**B) `ThemeData(elevatedButtonTheme: ElevatedButtonThemeData(style: ButtonStyle(...)))`**

Каждый Material компонент имеет свой `ThemeData` параметр: `textButtonTheme`, `outlinedButtonTheme`, `cardTheme`, `chipTheme`, `dialogTheme`, `snackBarTheme`, `appBarTheme` и т.д. `ButtonStyle` использует `MaterialStateProperty` для разных состояний (pressed, hovered, disabled...).

</details>

---

### Вопрос 6 🟡

Что такое `TextTheme` и как им пользоваться?

- A) Класс для кастомных шрифтов
- B) Набор предопределённых стилей текста (displayLarge, titleMedium, bodySmall...) из Material Design
- C) Тема только для `TextField`
- D) Способ задать глобальный шрифт

<details>
<summary>Ответ</summary>

**B) Набор предопределённых стилей текста из Material Design типографической системы**

Material 3 типографика: `displayLarge/Medium/Small`, `headlineLarge/Medium/Small`, `titleLarge/Medium/Small`, `bodyLarge/Medium/Small`, `labelLarge/Medium/Small`. Использование: `Theme.of(context).textTheme.titleMedium`. Кастомизация: `ThemeData(textTheme: TextTheme(titleMedium: TextStyle(...)))`.

</details>

---

### Вопрос 7 🟡

Как создать локальную тему только для части экрана?

- A) Нельзя — тема только глобальная
- B) `Theme(data: Theme.of(context).copyWith(colorScheme: ...), child: MyWidget())`
- C) `LocalTheme(child: MyWidget())`
- D) `ChildTheme(overwrites: {...})`

<details>
<summary>Ответ</summary>

**B) `Theme(data: Theme.of(context).copyWith(...), child: MyWidget())`**

`Theme.of(context).copyWith(...)` создаёт копию темы с изменёнными полями. Виджет `Theme` делает эту тему доступной всему поддереву через `InheritedWidget` механизм. Полезно для диалогов, bottom sheets, отдельных секций экрана.

</details>

---

### Вопрос 8 🔴

Что такое `MaterialStateProperty` в `ButtonStyle`?

- A) Свойство хранящее значение состояния компонента
- B) Объект, возвращающий разные значения для разных интерактивных состояний кнопки (pressed, hovered, focused, disabled, selected, error)
- C) Аналог CSS pseudo-selector
- D) Все три ответа описывают одно и то же

<details>
<summary>Ответ</summary>

**B) Объект, возвращающий разные значения для разных интерактивных состояний**

```dart
backgroundColor: MaterialStateProperty.resolveWith<Color>((states) {
  if (states.contains(MaterialState.disabled)) return Colors.grey;
  if (states.contains(MaterialState.pressed)) return Colors.blue.shade900;
  return Colors.blue;
}),
```

В Flutter 3.19+ переименована в `WidgetStateProperty`. `MaterialStateProperty.all(value)` — одно значение для всех состояний.

</details>

---

### Вопрос 9 🔴

Как создать динамическую тему которая меняется в runtime?

- A) Изменить `ThemeData` в `setState`
- B) Управлять `themeMode` в `MaterialApp` через state management (Provider, Riverpod и т.д.)
- C) `Theme.of(context).update(...)`
- D) `MaterialApp.of(context).switchTheme(...)`

<details>
<summary>Ответ</summary>

**B) Управлять `themeMode` в MaterialApp через state management**

```dart
// С Provider:
MaterialApp(
  theme: lightTheme,
  darkTheme: darkTheme,
  themeMode: ref.watch(themeModeProvider), // ThemeMode.dark/light/system
)
```

`MaterialApp` является корнем дерева — при изменении `themeMode` перестраивается всё приложение. Это нормально и ожидаемо для смены темы.

</details>

---

### Вопрос 10 🔴

Как реализовать `Dynamic Color` — адаптацию к цвету обоев Android 12+?

- A) Flutter автоматически поддерживает Dynamic Color
- B) Через пакет `dynamic_color`: `DynamicColorBuilder` возвращает `ColorScheme` полученную от системы
- C) `ThemeData(useDynamicColor: true)`
- D) `ColorScheme.fromSystem(context)`

<details>
<summary>Ответ</summary>

**B) Через пакет `dynamic_color`: `DynamicColorBuilder`**

```dart
DynamicColorBuilder(
  builder: (lightDynamic, darkDynamic) {
    return MaterialApp(
      theme: ThemeData(
        colorScheme: lightDynamic ?? fallbackLight,
        useMaterial3: true,
      ),
      darkTheme: ThemeData(
        colorScheme: darkDynamic ?? fallbackDark,
        useMaterial3: true,
      ),
    );
  },
)
```

`lightDynamic`/`darkDynamic` null на Android < 12 и iOS — нужен fallback.

</details>
