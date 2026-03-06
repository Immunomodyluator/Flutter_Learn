# Квиз: Именованные маршруты

**Тема:** 06.2 — Named Routes  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как объявить именованные маршруты в `MaterialApp`?

- A) `MaterialApp(namedRoutes: {'/home': HomeScreen})`
- B) `MaterialApp(routes: {'/home': (ctx) => HomeScreen(), '/detail': (ctx) => DetailScreen()})`
- C) `MaterialApp(pages: [NamedPage('/home', HomeScreen())])`
- D) `Navigator.registerRoute('/home', HomeScreen)`

<details>
<summary>Ответ</summary>

**B) `MaterialApp(routes: {'/home': (ctx) => HomeScreen(), ...})`**

Ключи — строки маршрутов, значения — builder функции `(BuildContext) => Widget`. Начальный маршрут: `MaterialApp(initialRoute: '/home', routes: {...})`. Если задан `initialRoute`, не задавайте `home` — иначе конфликт.

</details>

---

### Вопрос 2 🟢

Как перейти на именованный маршрут?

- A) `Navigator.goto(context, '/detail')`
- B) `Navigator.of(context).pushNamed('/detail')`
- C) `context.push('/detail')`
- D) `Router.navigate('/detail')`

<details>
<summary>Ответ</summary>

**B) `Navigator.of(context).pushNamed('/detail')`**

Краткая форма: `Navigator.pushNamed(context, '/detail')`. С аргументами: `Navigator.pushNamed(context, '/detail', arguments: {'id': 42})`. Получение аргументов на экране: `final args = ModalRoute.of(context)!.settings.arguments as Map`.

</details>

---

### Вопрос 3 🟢

Что такое `initialRoute` в `MaterialApp`?

- A) Маршрут, который загружается первым при запуске приложения
- B) Маршрут для страницы 404
- C) Маршрут по умолчанию при ошибке навигации
- D) Алиас для `home`

<details>
<summary>Ответ</summary>

**A) Маршрут, который загружается первым при запуске приложения**

`initialRoute: '/splash'` — запускает приложение с маршрута `/splash`. По умолчанию — `'/'`. Если не указать `initialRoute` и не задать маршрут для `'/'`, а задан `home` — флаттер использует `home`.

</details>

---

### Вопрос 4 🟡

Как передать аргументы через именованный маршрут и получить их?

- A) Только через конструктор дочернего виджета
- B) `pushNamed(..., arguments: data)` → `ModalRoute.of(context)!.settings.arguments`
- C) `pushNamed(..., params: data)` → `RouteSettings.params`
- D) Нельзя — именованные маршруты не поддерживают аргументы

<details>
<summary>Ответ</summary>

**B) `pushNamed(..., arguments: data)` → `ModalRoute.of(context)!.settings.arguments`**

```dart
// Отправка:
Navigator.pushNamed(context, '/detail', arguments: Product(id: 42));

// Получение:
final product = ModalRoute.of(context)!.settings.arguments as Product;
```

Тип-небезопасно — рекомендуется GoRouter с типизированными маршрутами.

</details>

---

### Вопрос 5 🟡

Что такое `onGenerateRoute`?

- A) Автоматическая генерация маршрутов при компиляции
- B) Callback вызываемый когда запрошенный маршрут не найден в `routes` — позволяет динамически строить маршруты
- C) Хук для логирования навигации
- D) Генератор deep links

<details>
<summary>Ответ</summary>

**B) Callback для динамического построения маршрутов**

```dart
MaterialApp(
  onGenerateRoute: (settings) {
    if (settings.name == '/product') {
      final id = settings.arguments as int;
      return MaterialPageRoute(builder: (_) => ProductScreen(id: id));
    }
    return null; // → onUnknownRoute
  },
)
```

Позволяет динамические маршруты с параметрами вида `/product/42` при использовании простого Navigator.

</details>

---

### Вопрос 6 🟡

Как перехватить переход на неизвестный маршрут?

- A) Через `try/catch` вокруг `pushNamed`
- B) `MaterialApp(onUnknownRoute: (settings) => MaterialPageRoute(builder: (_) => NotFoundScreen()))`
- C) `Navigator.on404((route) => ...)`
- D) `MaterialApp(errorRoute: '/404')`

<details>
<summary>Ответ</summary>

**B) `MaterialApp(onUnknownRoute: ...)`**

`onUnknownRoute` вызывается когда маршрут не найден ни в `routes`, ни в `onGenerateRoute`. Аналог 404 для навигации. Обязателен если `routes` + `onGenerateRoute` не покрывают все случаи.

</details>

---

### Вопрос 7 🟡

Почему `pushNamedAndRemoveUntil` полезен при выходе из системы?

- A) Он ускоряет навигацию
- B) Очищает весь стек навигации и переходит на стартовый экран, предотвращая возврат на защищённые экраны
- C) Сохраняет историю навигации
- D) Вызывает `dispose` у всех экранов в стеке

<details>
<summary>Ответ</summary>

**B) Очищает весь стек навигации при переходе на стартовый экран**

```dart
Navigator.of(context).pushNamedAndRemoveUntil(
  '/login',
  (route) => false, // false = удалить все маршруты
);
```

После logout пользователь не сможет нажать "назад" и попасть на защищённые экраны. `(route) => route.isFirst` — удаляет всё кроме первого маршрута.

</details>

---

### Вопрос 8 🔴

Как реализовать guards (защиту маршрутов) для аутентификации через `onGenerateRoute`?

- A) Flutter не поддерживает route guards
- B) В `onGenerateRoute` проверять статус аутентификации и перенаправлять на `/login` если не авторизован
- C) Через `NavigatorObserver`
- D) Декоратором `@Protected` над классом экрана

<details>
<summary>Ответ</summary>

**B) В `onGenerateRoute` проверять статус аутентификации**

```dart
onGenerateRoute: (settings) {
  final isLoggedIn = AuthService.isLoggedIn;
  final protectedRoutes = ['/home', '/profile', '/settings'];

  if (!isLoggedIn && protectedRoutes.contains(settings.name)) {
    return MaterialPageRoute(builder: (_) => LoginScreen());
  }
  // обычная маршрутизация...
},
```

GoRouter делает это элегантнее через `redirect` callback.

</details>

---

### Вопрос 9 🔴

Что такое `NavigatorObserver` и как его использовать?

- A) Observer для отладки — только в debug режиме
- B) Интерфейс для прослушивания событий навигации (push/pop/replace) — используется для аналитики, логирования
- C) Обёртка для `Navigator` добавляющая анимации
- D) Observer для системных событий (rotate, resize)

<details>
<summary>Ответ</summary>

**B) Интерфейс для прослушивания событий навигации**

```dart
class AnalyticsObserver extends NavigatorObserver {
  @override
  void didPush(Route route, Route? previousRoute) {
    analytics.logScreenView(route.settings.name);
  }
}

// Регистрация:
MaterialApp(navigatorObservers: [AnalyticsObserver()])
```

Firebase Analytics предоставляет `FirebaseAnalyticsObserver` готовый к использованию.

</details>

---

### Вопрос 10 🔴

Какие недостатки именованных маршрутов по сравнению с GoRouter?

- A) Именованные маршруты лучше GoRouter во всём
- B) Нет типизации аргументов, сложная вложенная навигация, нет поддержки web URL, отсутствие redirect и guards, сложность в тестировании
- C) Именованные маршруты не работают на iOS
- D) Отличаются только синтаксисом

<details>
<summary>Ответ</summary>

**B) Нет типизации аргументов, сложная вложенная навигация, нет web URL, нет redirect/guards**

GoRouter преимущества:

- Типизированные параметры: `state.pathParameters['id']`
- URL-based: `/product/42` → `ProductScreen(id: 42)`
- Redirect: guard на уровне маршрута
- Declarative: описываешь маршруты декларативно
- Nested navigation из коробки (ShellRoute)
- Поддерживается Flutter team

</details>
