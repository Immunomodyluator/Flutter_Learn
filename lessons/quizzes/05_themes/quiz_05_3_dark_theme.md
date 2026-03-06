# Квиз: Тёмная тема

**Тема:** 05.3 — Dark Theme  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как добавить поддержку тёмной темы в Flutter?

- A) `MaterialApp(darkMode: true)`
- B) `MaterialApp(theme: lightTheme, darkTheme: darkTheme, themeMode: ThemeMode.system)`
- C) `ThemeData(brightness: Brightness.dark)`
- D) `Platform.isDark ? darkTheme : lightTheme`

<details>
<summary>Ответ</summary>

**B) `MaterialApp(theme: lightTheme, darkTheme: darkTheme, themeMode: ThemeMode.system)`**

`ThemeMode.system` — следует системным настройкам. `ThemeMode.light` — всегда светлая. `ThemeMode.dark` — всегда тёмная. При `themeMode: ThemeMode.system` Flutter автоматически переключается по `MediaQuery.platformBrightness`.

</details>

---

### Вопрос 2 🟢

Как определить текущую яркость (светлая/тёмная тема) в виджете?

- A) `Theme.of(context).isDark`
- B) `Theme.of(context).brightness == Brightness.dark`
- C) `MediaQuery.of(context).platformBrightness`
- D) `B — для темы виджета; C — для системной настройки`

<details>
<summary>Ответ</summary>

**D) B — для темы виджета; C — для системной настройки**

- `Theme.of(context).brightness` — яркость **применённой** темы (то что вы задали в `MaterialApp`)
- `MediaQuery.of(context).platformBrightness` — **системная** настройка устройства

Они могут различаться если вы принудительно задали `themeMode: ThemeMode.light` на тёмном устройстве.

</details>

---

### Вопрос 3 🟢

Как создать тёмный `ThemeData` в Material 3?

- A) `ThemeData.dark()`
- B) `ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: color, brightness: Brightness.dark), useMaterial3: true)`
- C) `ThemeData(brightness: Brightness.dark, primaryColor: Colors.black)`
- D) `ThemeData.copy(brightness: Brightness.dark)`

<details>
<summary>Ответ</summary>

**B) `ThemeData(colorScheme: ColorScheme.fromSeed(seedColor: color, brightness: Brightness.dark), useMaterial3: true)`**

`ColorScheme.fromSeed` с `brightness: Brightness.dark` генерирует тёмную цветовую схему из того же seed-цвета. Тёмная схема имеет другие значения surface, background, on-цветов для правильного контраста.

</details>

---

### Вопрос 4 🟡

Как сделать одно изображение для светлой и одно для тёмной темы?

- A) Проверить `Theme.of(context).brightness` и выбрать путь
- B) Использовать `AssetImage` с вариантами (`2.0x`, `3.0x` — для темы нет)
- C) `Image.asset(isDark ? 'dark_img.png' : 'light_img.png')`
- D) A и C оба правильны

<details>
<summary>Ответ</summary>

**D) A и C оба правильны**

```dart
final isDark = Theme.of(context).brightness == Brightness.dark;
Image.asset(isDark ? 'assets/logo_dark.png' : 'assets/logo_light.png')
```

Альтернатива: использовать SVG с `colorFilter` который адаптируется к теме, или `ThemeData.of(context).colorScheme` цвета вместо картинок.

</details>

---

### Вопрос 5 🟡

Что такое `surfaceVariant`, `onSurfaceVariant` в Material 3 ColorScheme?

- A) Устаревшие поля из Material 2
- B) Цвет поверхности для контейнеров и элементов со средним акцентом, `onSurfaceVariant` — цвет текста/иконок на нём
- C) Варианты цвета primary
- D) Цвета только для тёмной темы

<details>
<summary>Ответ</summary>

**B) Цвет поверхности для контейнеров и элементов со средним акцентом**

Material 3 расширил ColorScheme: `surface`, `surfaceVariant`, `surfaceContainerLowest/Low/High/Highest`. `onSurfaceVariant` обеспечивает нужный контраст на `surfaceVariant`. Используются в `Card`, `Chip`, `TextField` фонах.

</details>

---

### Вопрос 6 🟡

Как сохранить выбор темы пользователя между запусками приложения?

- A) Flutter сохраняет это автоматически
- B) Сохранить `ThemeMode` в `SharedPreferences`, загружать при старте
- C) Использовать `Platform.savedTheme`
- D) `ThemeData` автоматически кэшируется

<details>
<summary>Ответ</summary>

**B) Сохранить `ThemeMode` в `SharedPreferences`, загружать при старте**

```dart
// Сохранение:
prefs.setString('themeMode', 'dark');

// Загрузка:
final savedMode = prefs.getString('themeMode');
ThemeMode mode = savedMode == 'dark' ? ThemeMode.dark : ThemeMode.light;
```

С Riverpod: `themeMode` в `StateProvider` + `SharedPreferences` для персистентности. Пакет `hydrated_bloc` автоматизирует персистентность для BLoC.

</details>

---

### Вопрос 7 🟡

Как анимировать переход между светлой и тёмной темой?

- A) Flutter анимирует автоматически
- B) `AnimatedTheme` виджет вместо `Theme` для плавной анимации
- C) Нельзя анимировать смену темы во Flutter
- D) `ThemeTransition` виджет

<details>
<summary>Ответ</summary>

**B) `AnimatedTheme` виджет вместо `Theme` для плавной анимации**

`AnimatedTheme` интерполирует между двумя `ThemeData` при изменении. `MaterialApp` использует `AnimatedTheme` внутри, поэтому смена `themeMode` уже анимирована. Для кастомных анимаций (circular reveal и т.д.) нужен `CustomPainter` + `AnimationController`.

</details>

---

### Вопрос 8 🔴

Что такое `elevation overlay` в тёмной теме Material 2 и как это изменилось в Material 3?

- A) Тени исчезли в Material 3
- B) В M2 тёмная тема добавляла белый оверлей на поверхности с elevation для имитации глубины; в M3 используются `tonalElevation` — оттенки от primary цвета
- C) `elevation overlay` — только для светлой темы
- D) Это одинаково в M2 и M3

<details>
<summary>Ответ</summary>

**B) В M2 — белый оверлей; в M3 — tonal elevation через primary цвет**

Material 2 тёмная тема: при elevation 8dp на surface добавляется `Colors.white` с opacity ~0.14. Material 3: `surfaceContainerLow`, `surfaceContainer`, `surfaceContainerHigh`, `surfaceContainerHighest` — разные поверхности с tonal tint от primary. Тени (`boxShadow`) используются меньше, глубина передаётся через цвет.

</details>

---

### Вопрос 9 🔴

Как создать адаптивный цвет, который выглядит хорошо в обеих темах?

- A) Использовать константу `Colors.blue` — она адаптируется автоматически
- B) Всегда обращаться к `Theme.of(context).colorScheme.*` вместо хардкода цветов
- C) Использовать `CupertinoColors.systemBlue` — он адаптивный
- D) Нет простого способа — нужны два разных дерева виджетов

<details>
<summary>Ответ</summary>

**B) Всегда обращаться к `Theme.of(context).colorScheme.*`**

`colorScheme.primary` в светлой теме — насыщенный цвет, в тёмной — более светлый оттенок того же seed. `colorScheme.surface` автоматически тёмный в dark mode. Принцип: **никогда не хардкодить цвета** — только из `colorScheme` или `textTheme`. Тогда смена темы работает бесплатно.

</details>

---

### Вопрос 10 🔴

Как реализовать `schedule-based` смену темы (светлая днём, тёмная ночью)?

- A) `ThemeMode.system` делает это автоматически
- B) Подписаться на время через `Timer.periodic`, менять `ThemeMode` на основе `TimeOfDay`
- C) Использовать `flutter_local_notifications` для смены темы
- D) `Platform.scheduleTheme`

<details>
<summary>Ответ</summary>

**B) Подписаться на время через `Timer.periodic`, менять `ThemeMode` по `TimeOfDay`**

```dart
Timer.periodic(Duration(minutes: 1), (_) {
  final hour = DateTime.now().hour;
  final mode = (hour >= 20 || hour < 7) ? ThemeMode.dark : ThemeMode.light;
  ref.read(themeModeProvider.notifier).state = mode;
});
```

Также слушать `WidgetsBinding.addObserver` + `didChangePlatformBrightness` для системных событий смены темы.

</details>
