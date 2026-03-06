# Квиз: Вложенная навигация

**Тема:** 06.4 — Nested Navigation  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое вложенная навигация во Flutter?

- A) Навигация внутри диалоговых окон
- B) Несколько независимых стеков навигации в рамках одного экрана (например, вкладки с собственной историей)
- C) Навигация через несколько route table
- D) Переходы между вложенными Scaffold

<details>
<summary>Ответ</summary>

**B) Несколько независимых стеков навигации в рамках одного экрана**

Пример: BottomNavigationBar с вкладками Home, Search, Profile. Каждая вкладка имеет свою историю переходов. При переключении вкладки стек сохраняется — пользователь может вернуться к месту где остановился.

</details>

---

### Вопрос 2 🟢

Как создать вложенный Navigator?

- A) `SubNavigator(key: _key, routes: {...})`
- B) Вложить виджет `Navigator(key: GlobalKey<NavigatorState>(), ...)` в дерево виджетов
- C) Через `OverlayEntry`
- D) `context.createSubNavigator()`

<details>
<summary>Ответ</summary>

**B) Вложить виджет `Navigator` с `GlobalKey<NavigatorState>` в дерево виджетов**

```dart
final _key = GlobalKey<NavigatorState>();

Navigator(
  key: _key,
  initialRoute: '/tab_home',
  onGenerateRoute: (settings) => ...,
)
```

Каждая вкладка получает свой Navigator с уникальным ключом.

</details>

---

### Вопрос 3 🟢

Как GoRouter реализует вложенную навигацию для вкладок?

- A) Через `NestedNavigator` класс
- B) Через `ShellRoute` или `StatefulShellRoute`
- C) Через `IndexedNavigator`
- D) Не поддерживает вложенную навигацию

<details>
<summary>Ответ</summary>

**B) Через `ShellRoute` или `StatefulShellRoute`**

`ShellRoute` — вкладки без сохранения состояния. `StatefulShellRoute` — сохраняет state каждой вкладки (рекомендуется). GoRouter 10+ `StatefulShellRoute.indexedStack` создаёт оптимизированную структуру.

</details>

---

### Вопрос 4 🟡

Как сохранить состояние (state) вкладок при переключении BottomNavigationBar?

- A) Состояние сохраняется автоматически
- B) `IndexedStack` + каждая вкладка как отдельный Navigator
- C) `AutomaticKeepAliveClientMixin` в корневом виджете вкладки
- D) B или C — оба подхода работают; `StatefulShellRoute` использует `IndexedStack` под капотом

<details>
<summary>Ответ</summary>

**D) B или C — оба подхода работают**

`IndexedStack` — отображает один дочерний виджет, остальные в памяти (состояние сохранено, но UI не обновляется). `AutomaticKeepAliveClientMixin` — сохраняет state при выходе из viewport. `StatefulShellRoute` объединяет оба подхода елегантно.

</details>

---

### Вопрос 5 🟡

Как из дочернего экрана вложенного Navigator вернуться к родительскому Navigator?

- A) `Navigator.of(context, rootNavigator: true).pop()`
- B) `context.go('/')` всегда идёт к корню
- C) Специальный `ParentNavigatorKey.pop()`
- D) `Navigator.of(context).popUntil((r) => r.isFirst)`

<details>
<summary>Ответ</summary>

**A) `Navigator.of(context, rootNavigator: true).pop()`**

`rootNavigator: true` — получает корневой Navigator, минуя вложенные. Полезно при вызове `showDialog`, `showModalBottomSheet` из вложенного Navigator — без `rootNavigator: true` диалог может быть ограничен вложенным стеком.

</details>

---

### Вопрос 6 🟡

Как реализовать BottomNavigationBar с вложенной навигацией используя GoRouter `StatefulShellRoute`?

- A) Через `ShellRoute` с `IndexedStack` вручную
- B) `StatefulShellRoute.indexedStack(builder: ..., branches: [StatefulShellBranch(routes: [...])])`
- C) `GoRouter.nested(branches: [...])`
- D) `TabBar` + `TabBarView` + Navigator в каждом tab

<details>
<summary>Ответ</summary>

**B) `StatefulShellRoute.indexedStack(builder: ..., branches: [StatefulShellBranch(...)])`**

```dart
StatefulShellRoute.indexedStack(
  builder: (ctx, state, shell) => ScaffoldWithNavBar(shell: shell),
  branches: [
    StatefulShellBranch(routes: [GoRoute(path: '/home', ...)]),
    StatefulShellBranch(routes: [GoRoute(path: '/search', ...)]),
  ],
)
```

`shell.goBranch(index)` — переключение вкладок.

</details>

---

### Вопрос 7 🟡

Что происходит когда пользователь нажимает "назад" находясь в корне вложенного Navigator?

- A) Закрывает приложение
- B) По умолчанию переходит в предыдущую вкладку
- C) Пузырится вверх к родительскому Navigator — может закрыть приложение или перейти к предыдущему экрану корневого Navigator
- D) Ничего — кнопка "назад" игнорируется

<details>
<summary>Ответ</summary>

**C) Пузырится вверх к родительскому Navigator**

Если вложенный Navigator не может `pop()` (стек пуст) — событие кнопки "назад" передаётся родителю. Для `BottomNavigationBar` это может выглядеть как: если на вкладке корень → переключиться на первую вкладку → ещё раз назад → выйти. Управляется через `PopScope`.

</details>

---

### Вопрос 8 🔴

Как реализовать поведение "нажать назад на главной вкладке → закрыть приложение"?

- A) `SystemNavigator.pop()` в `onTapBottomBar`
- B) `PopScope(canPop: false, onPopInvoked: (didPop) { if (currentTab == 0) SystemNavigator.pop(); else switchToTab(0); })`
- C) `WillPopScope` + `exit(0)`
- D) Автоматическое поведение Flutter

<details>
<summary>Ответ</summary>

**B) `PopScope` с логикой переключения на главную вкладку или выхода**

```dart
PopScope(
  canPop: false,
  onPopInvoked: (didPop) {
    if (_currentIndex != 0) {
      setState(() => _currentIndex = 0);
    } else {
      SystemNavigator.pop(); // закрыть приложение
    }
  },
  child: Scaffold(bottomNavigationBar: ...),
)
```

</details>

---

### Вопрос 9 🔴

Как передать `GlobalKey<NavigatorState>` вложенного Navigator для программного управления?

- A) Через `InheritedWidget`
- B) Создать ключ как поле/синглтон, передать в `Navigator(key: key)`, использовать `key.currentState!.push(...)`
- C) Через `context.findNavigator()`
- D) Navigator ключи не поддерживают программный доступ

<details>
<summary>Ответ</summary>

**B) Создать ключ как поле/синглтон, передать в `Navigator(key: key)`, использовать `key.currentState!.push(...)`**

```dart
final homeTabKey = GlobalKey<NavigatorState>();

// В Navigator:
Navigator(key: homeTabKey, ...)

// Программная навигация из любого места:
homeTabKey.currentState!.push(MaterialPageRoute(builder: (_) => DetailsScreen()));
```

Полезно для навигации из вне UI (например из state management).

</details>

---

### Вопрос 10 🔴

Какие проблемы создаёт вложенная навигация при работе с `Hero` анимациями?

- A) Hero анимации не работают с вложенной навигацией
- B) По умолчанию Hero привязан к корневому Navigator — для работы внутри вложенного нужно добавить `HeroFlightShuttleBuilder` или задать `Navigator` с `heroFlightShuttleBuilder`
- C) Hero анимации всегда проходят через корневой Navigator игнорируя вложенные
- D) Нужно явно указать `heroRoot: false`

<details>
<summary>Ответ</summary>

**C) Hero анимации всегда проходят через корневой Navigator**

Hero ищет соответствующий виджет в пределах одного `Navigator`. При вложенных Navigator Hero может не найти пару или анимировать неверно. Решение: задать `transitionOnUserGestures: true` и убедиться что оба Hero в одном Navigator scope. Для перехода между вкладками Hero работает только если маршруты в одном Navigator.

</details>
