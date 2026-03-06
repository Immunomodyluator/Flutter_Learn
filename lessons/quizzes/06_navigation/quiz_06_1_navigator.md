# Квиз: Navigator и маршруты

**Тема:** 06.1 — Navigator  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как перейти на новый экран с помощью `Navigator`?

- A) `Navigator.go(context, NewScreen())`
- B) `Navigator.of(context).push(MaterialPageRoute(builder: (_) => NewScreen()))`
- C) `context.navigate(NewScreen())`
- D) `Router.push(NewScreen())`

<details>
<summary>Ответ</summary>

**B) `Navigator.of(context).push(MaterialPageRoute(builder: (_) => NewScreen()))`**

`push` добавляет маршрут в стек Navigator. `MaterialPageRoute` — стандартный переход с платформо-зависимой анимацией (slide на iOS, fade на Android). Остальные варианты несуществующие API.

</details>

---

### Вопрос 2 🟢

Как вернуться на предыдущий экран?

- A) `Navigator.of(context).back()`
- B) `Navigator.of(context).pop()`
- C) `context.goBack()`
- D) `Router.pop(context)`

<details>
<summary>Ответ</summary>

**B) `Navigator.of(context).pop()`**

`pop()` убирает верхний маршрут из стека. `pop(result)` — возвращает значение, которое можно получить через `await Navigator.push(...)`. `canPop()` проверяет, есть ли что возвращать. `maybePop()` — вызывает pop только если `canPop()`.

</details>

---

### Вопрос 3 🟢

Как передать данные на следующий экран?

- A) Через `Navigator.push` — нет такой возможности, только через state management
- B) Передать через конструктор виджета в `MaterialPageRoute(builder: (_) => DetailScreen(item: item))`
- C) Через глобальную переменную
- D) `Navigator.send(context, data: item)`

<details>
<summary>Ответ</summary>

**B) Передать через конструктор виджета в `MaterialPageRoute`**

```dart
Navigator.of(context).push(
  MaterialPageRoute(builder: (_) => DetailScreen(item: selectedItem)),
);
```

Детальный экран: `class DetailScreen extends StatelessWidget { final Item item; const DetailScreen({required this.item}); }`. Это самый простой и прямой способ.

</details>

---

### Вопрос 4 🟡

Что делает `Navigator.pushReplacement`?

- A) Добавляет новый экран поверх и удаляет текущий
- B) Заменяет весь стек навигации
- C) Открывает экран без анимации
- D) Добавляет экран в стек, не удаляя текущий

<details>
<summary>Ответ</summary>

**A) Добавляет новый экран поверх и удаляет текущий**

`pushReplacement` = push + удалить предыдущий. Используется для смены экранов без возможности вернуться назад: после логина → главный экран, после splash → onboarding. `pushAndRemoveUntil` — очищает стек до определённого маршрута.

</details>

---

### Вопрос 5 🟡

Как получить результат от закрытого экрана?

- A) Через shared state
- B) `final result = await Navigator.of(context).push(...)`, а на возвращаемом экране `Navigator.pop(context, result)`
- C) Через `Stream` и `StreamController`
- D) Callback через конструктор не работает с Navigator

<details>
<summary>Ответ</summary>

**B) `await Navigator.push(...)` + `Navigator.pop(context, result)`**

```dart
// Вызывающий экран:
final result = await Navigator.of(context).push<String>(
  MaterialPageRoute(builder: (_) => PickerScreen()),
);
if (result != null) doSomethingWith(result);

// Picker экран:
Navigator.of(context).pop('selected_value');
```

</details>

---

### Вопрос 6 🟡

Что такое именованные маршруты и как их настроить?

- A) Маршруты с именами для отладки
- B) `MaterialApp(routes: {'/home': (_) => HomeScreen(), '/detail': (_) => DetailScreen()})` + `Navigator.pushNamed(context, '/home')`
- C) Маршруты определённые в отдельном файле
- D) Flutter 3.0 удалил поддержку именованных маршрутов

<details>
<summary>Ответ</summary>

**B) `MaterialApp(routes: {...})` + `Navigator.pushNamed(context, '/home')`**

Именованные маршруты удобны, но ограничены: передача аргументов через `Navigator.pushNamed(context, '/detail', arguments: item)`, получение: `ModalRoute.of(context)!.settings.arguments`. Для сложной навигации используйте GoRouter.

</details>

---

### Вопрос 7 🟡

Что такое `WillPopScope` (или `PopScope` в новых версиях)?

- A) Виджет для анимированного возврата
- B) Виджет для перехвата нажатия кнопки "назад" с возможностью отмены или кастомной логики
- C) Scope для множественных Navigator
- D) Обёртка для диалогов

<details>
<summary>Ответ</summary>

**B) Виджет для перехвата нажатия кнопки "назад"**

`WillPopScope` (Flutter < 3.12) / `PopScope` (Flutter ≥ 3.12):

```dart
PopScope(
  canPop: false,
  onPopInvoked: (didPop) {
    if (!didPop) showUnsavedChangesDialog(context);
  },
  child: Scaffold(...),
)
```

Используется для подтверждения выхода с несохранёнными изменениями.

</details>

---

### Вопрос 8 🔴

Чем `Navigator 1.0` отличается от `Navigator 2.0`?

- A) Navigator 2.0 быстрее
- B) Navigator 1.0 — императивный (push/pop методы); Navigator 2.0 — декларативный (Router + RouteInformationParser + RouterDelegate) с поддержкой web URL и deep links
- C) Navigator 2.0 только для Desktop
- D) Они идентичны, разные только версии API

<details>
<summary>Ответ</summary>

**B) Navigator 1.0 — императивный; Navigator 2.0 — декларативный с web/deep link поддержкой**

Navigator 2.0 (Router API) позволяет: отражать состояние приложения в URL браузера, обрабатывать deep links, управлять историей. Сложен в реализации напрямую — используйте GoRouter, Auto Route, Beamer.

</details>

---

### Вопрос 9 🔴

Как реализовать вложенную навигацию (nested navigation) для `BottomNavigationBar`?

- A) Один Navigator на всё приложение
- B) Каждый таб получает свой `Navigator` ключ через `GlobalKey<NavigatorState>` — сохраняет стек вкладки
- C) `TabController` + `Navigator`
- D) `IndexedStack` без Navigator

<details>
<summary>Ответ</summary>

**B) Каждый таб получает свой `Navigator` через `GlobalKey<NavigatorState>`**

```dart
// У каждого таба:
Navigator(
  key: _tabKeys[_currentIndex],
  onGenerateRoute: (settings) => ...,
)
```

Или с GoRouter `ShellRoute` / `StatefulShellRoute` который автоматически управляет стеками вкладок. `IndexedStack` используется чтобы сохранять виджеты вкладок.

</details>

---

### Вопрос 10 🔴

Как обработать deep link `myapp://product/42` и открыть `ProductScreen(id: 42)`?

- A) Через `UniversalLinks` пакет
- B) Настроить `onGenerateRoute` в `MaterialApp` или использовать GoRouter с deep link конфигурацией + настроить `AndroidManifest.xml` / `Info.plist`
- C) Flutter не поддерживает deep links
- D) `Navigator.handleDeepLink(uri)`

<details>
<summary>Ответ</summary>

**B) `onGenerateRoute` или GoRouter + нативные конфиги**

С GoRouter:

```dart
GoRouter(routes: [
  GoRoute(path: '/product/:id', builder: (ctx, state) {
    return ProductScreen(id: int.parse(state.pathParameters['id']!));
  }),
])
```

Нативно: `AndroidManifest.xml` с `intent-filter` для `myapp://` scheme, `Info.plist` `CFBundleURLSchemes`. Универсальные ссылки (`https://`) требуют `assetlinks.json` / `apple-app-site-association`.

</details>
