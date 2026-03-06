# Квиз: GoRouter

**Тема:** 06.3 — GoRouter  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как подключить GoRouter к приложению?

- A) `MaterialApp(router: goRouter)`
- B) `MaterialApp.router(routerConfig: goRouter)`
- C) `GoRouter.attach(context, router)`
- D) `MaterialApp(goRouter: goRouter)`

<details>
<summary>Ответ</summary>

**B) `MaterialApp.router(routerConfig: goRouter)`**

```dart
final _router = GoRouter(routes: [...]);

MaterialApp.router(
  routerConfig: _router,
  title: 'My App',
  theme: ThemeData(...),
)
```

`MaterialApp.router` использует Navigator 2.0 под капотом и принимает `RouterConfig`.

</details>

---

### Вопрос 2 🟢

Как определить базовый маршрут в GoRouter?

- A) `Route(path: '/home', widget: HomeScreen())`
- B) `GoRoute(path: '/home', builder: (context, state) => HomeScreen())`
- C) `GoRouter.addRoute('/home', () => HomeScreen())`
- D) `GoRouteDef('/home', (ctx, _) => HomeScreen())`

<details>
<summary>Ответ</summary>

**B) `GoRoute(path: '/home', builder: (context, state) => HomeScreen())`**

`GoRoute` принимает `path`, `builder` (или `pageBuilder` для кастомных переходов), `routes` (вложенные маршруты), `redirect`. `GoRouterState state` содержит `pathParameters`, `queryParameters`, `extra`.

</details>

---

### Вопрос 3 🟢

Как перейти на маршрут с GoRouter?

- A) `Navigator.of(context).pushNamed('/home')`
- B) `context.go('/home')` или `context.push('/home')`
- C) `GoRouter.of(context).navigate('/home')`
- D) `router.go('/home')` — только через ссылку на роутер

<details>
<summary>Ответ</summary>

**B) `context.go('/home')` или `context.push('/home')`**

Разница: `context.go('/home')` — декларативная навигация (заменяет URL, может изменить стек), `context.push('/home')` — добавляет поверх стека. `context.pop()` — возврат. `context.goNamed('home')` — по имени маршрута.

</details>

---

### Вопрос 4 🟡

Как передать параметры пути — `/product/42`?

- A) `GoRoute(path: '/product', builder: ...)` + `context.go('/product?id=42')`
- B) `GoRoute(path: '/product/:id', builder: (ctx, state) => ProductScreen(id: state.pathParameters['id']!))`
- C) `GoRoute(path: '/product/{id}', ...)`
- D) Именованные параметры не поддерживаются в GoRouter

<details>
<summary>Ответ</summary>

**B) `GoRoute(path: '/product/:id', ...)` + `state.pathParameters['id']`**

```dart
GoRoute(
  path: '/product/:id',
  builder: (context, state) {
    final id = int.parse(state.pathParameters['id']!);
    return ProductScreen(id: id);
  },
)
// Переход:
context.go('/product/42');
```

</details>

---

### Вопрос 5 🟡

Чем `context.go` отличается от `context.push`?

- A) Они идентичны
- B) `go` — заменяет текущую локацию (похоже на `pushReplacement`), `push` — добавляет в стек (можно вернуться)
- C) `go` только для именованных маршрутов
- D) `push` работает только для вложенных маршрутов

<details>
<summary>Ответ</summary>

**B) `go` заменяет, `push` добавляет в стек**

`context.go('/home')` — пользователь не может вернуться назад кнопкой. Используется для перехода между разделами. `context.push('/detail')` — добавляет экран, с кнопкой "назад". `context.replace('/new')` — заменяет текущий экран без изменения остального стека.

</details>

---

### Вопрос 6 🟡

Как реализовать redirect (перенаправление) для защиты маршрутов?

- A) Через `NavigatorObserver`
- B) В `GoRouter(redirect: ...)` или на уровне `GoRoute(redirect: ...)` — проверяется перед каждым переходом
- C) В `onGenerateRoute`
- D) `GoRoute(guard: AuthGuard())`

<details>
<summary>Ответ</summary>

**B) `GoRouter(redirect: ...)` — глобальный или `GoRoute(redirect: ...)` — локальный**

```dart
GoRouter(
  redirect: (context, state) {
    final isLoggedIn = authService.isLoggedIn;
    if (!isLoggedIn && state.matchedLocation != '/login') {
      return '/login';
    }
    return null; // null = no redirect
  },
)
```

Redirect может быть асинхронным с `refreshListenable` для реактивного перенаправления.

</details>

---

### Вопрос 7 🟡

Что такое `ShellRoute` в GoRouter?

- A) Маршрут для полноэкранных диалогов
- B) Контейнер-маршрут с постоянным UI оболочкой (напр. BottomNavigationBar), внутри которого рендерятся дочерние маршруты
- C) Маршрут для platform-specific UI
- D) Обёртка для асинхронных маршрутов

<details>
<summary>Ответ</summary>

**B) Контейнер с постоянным UI оболочкой внутри которого рендерятся дочерние маршруты**

```dart
ShellRoute(
  builder: (ctx, state, child) => MainShell(child: child), // child = текущий дочерний маршрут
  routes: [
    GoRoute(path: '/home', builder: ...),
    GoRoute(path: '/search', builder: ...),
    GoRoute(path: '/profile', builder: ...),
  ],
)
```

`StatefulShellRoute` сохраняет состояние каждой вкладки.

</details>

---

### Вопрос 8 🔴

Как реализовать типизированную навигацию (typed routes) в GoRouter?

- A) GoRouter не поддерживает типизацию
- B) `GoRouterBuilder` code generation: классы маршрутов с аннотацией `@TypedGoRoute`, параметры — поля класса
- C) `TypedRoute<ProductScreen>(id: int)` встроен в GoRouter
- D) Через `get_it` dependency injection

<details>
<summary>Ответ</summary>

**B) `GoRouterBuilder` code generation с аннотацией `@TypedGoRoute`**

```dart
@TypedGoRoute<ProductRoute>(path: '/product/:id')
class ProductRoute extends GoRouteData {
  final int id;
  const ProductRoute({required this.id});

  @override
  Widget build(BuildContext context, GoRouterState state) {
    return ProductScreen(id: id);
  }
}
// Навигация:
ProductRoute(id: 42).go(context);
```

</details>

---

### Вопрос 9 🔴

Как реализовать `refreshListenable` для автоматического перенаправления при изменении состояния авторизации?

- A) `GoRouter(onAuthChange: authStream)`
- B) `GoRouter(refreshListenable: authStateNotifier)` — при изменении `Listenable` GoRouter перезапускает `redirect`
- C) Подписаться на изменения вручную через `addListener`
- D) `GoRouter.refresh()` вызывается вручную в UI

<details>
<summary>Ответ</summary>

**B) `GoRouter(refreshListenable: authStateNotifier)`**

```dart
GoRouter(
  refreshListenable: authProvider, // ChangeNotifier или ValueNotifier
  redirect: (context, state) {
    final isLoggedIn = ref.read(authProvider).isLoggedIn;
    if (!isLoggedIn) return '/login';
    return null;
  },
)
```

При изменении `authProvider` (logout/login) GoRouter автоматически перезапустит `redirect`. С Riverpod: обернуть `ref` в `ChangeNotifier`.

</details>

---

### Вопрос 10 🔴

Как управлять ошибкой парсинга маршрута (пользователь ввёл несуществующий URL в браузере)?

- A) `GoRouter(onError: (ctx, state, error) => ErrorScreen())`
- B) `GoRouter(errorBuilder: (context, state) => NotFoundScreen(error: state.error))`
- C) `onGenerateRoute` fallback
- D) Автоматически показывает debugger

<details>
<summary>Ответ</summary>

**B) `GoRouter(errorBuilder: (context, state) => NotFoundScreen(error: state.error))`**

```dart
GoRouter(
  errorBuilder: (context, state) => Scaffold(
    body: Center(child: Text('Страница не найдена: ${state.error}')),
  ),
  routes: [...],
)
```

`state.error` — `Exception` с описанием. `errorPageBuilder` — для кастомных `Page` объектов с кастомными анимациями.

</details>
