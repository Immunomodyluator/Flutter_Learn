# Квиз: Hero анимации

**Тема:** 11.3 — Hero Animations  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что делает виджет `Hero` и когда его использовать?

- A) Создаёт полноэкранный анимированный переход
- B) Анимирует "полёт" виджета между двумя экранами при навигации — если `tag` совпадает на обоих экранах
- C) `Hero` = AnimatedContainer для навигации
- D) Анимирует Enter/Exit экрана целиком

<details>
<summary>Ответ</summary>

**B) "Полёт" виджета между экранами при совпадении `tag`**

```dart
// Экран 1:
Hero(
  tag: 'product-image-${product.id}', // уникальный tag
  child: Image.network(product.imageUrl, width: 100, height: 100),
)

// Экран 2 (детали):
Hero(
  tag: 'product-image-${product.id}', // тот же tag!
  child: Image.network(product.imageUrl, width: double.infinity, height: 300),
)
```

Flutter автоматически анимирует переход при `Navigator.push()`.

</details>

---

### Вопрос 2 🟢

Что должен содержать `tag` у виджета `Hero`?

- A) Строку `'hero'` — константное значение
- B) Уникальный идентификатор — должен быть одинаковым на обоих экранах и уникальным среди всех Hero на одном экране
- C) `GlobalKey<HeroState>`
- D) `int` значение — только числа

<details>
<summary>Ответ</summary>

**B) Уникальный идентификатор, одинаковый на source и destination экранах**

```dart
// ХОРОШО — уникальный tag с ID:
Hero(tag: 'avatar-${user.id}', child: CircleAvatar(backgroundImage: NetworkImage(user.avatar)))

// ПЛОХО — одинаковый tag для разных элементов в списке:
ListView.builder(
  itemBuilder: (ctx, i) => Hero(
    tag: 'item', // ошибка! все элементы имеют одинаковый tag
    child: ...
  )
)

// ХОРОШО — уникальный tag для каждого элемента:
Hero(tag: 'item-$i', child: ...)
```

</details>

---

### Вопрос 3 🟢

Работают ли Hero анимации с именованными маршрутами?

- A) Только с `Navigator.push()` прямым методом
- B) Да — Hero работает с любой навигацией Flutter: `push`, `pushNamed`, `go_router` — только совпадение `tag`
- C) Только с GoRouter
- D) Только с MaterialApp.router

<details>
<summary>Ответ</summary>

**B) Да — Hero работает с любой навигацией**

```dart
// С Navigator:
Navigator.push(ctx, MaterialPageRoute(builder: (_) => DetailScreen(id: id)));

// С именованными маршрутами:
Navigator.pushNamed(ctx, '/detail', arguments: {'id': id});

// С GoRouter — также поддерживается:
context.go('/detail/$id');
// Но для GoRouter может потребоваться HeroControllerScope

// Убедиться что HeroController присутствует:
MaterialApp(
  // HeroController добавляется автоматически MaterialApp
)
```

</details>

---

### Вопрос 4 🟡

Что такое `heroFlightShuttleBuilder` и зачем он нужен?

- A) Callback при завершении Hero анимации
- B) Позволяет кастомизировать виджет который "летит" между экранами — по умолчанию это виджет из destination
- C) Настраивает скорость Hero анимации
- D) Только для видео в Hero

<details>
<summary>Ответ</summary>

**B) Кастомизация "летящего" виджета во время анимации**

```dart
Hero(
  tag: 'avatar-${user.id}',
  flightShuttleBuilder: (
    flightContext,
    animation,
    flightDirection,
    fromHeroContext,
    toHeroContext,
  ) {
    // Показать другой виджет во время полёта:
    return DefaultTextStyle(
      style: DefaultTextStyle.of(toHeroContext).style,
      child: CircleAvatar(
        backgroundImage: NetworkImage(user.avatar),
        radius: animation.value * 50, // кастомная анимация размера
      ),
    );
  },
  child: CircleAvatar(backgroundImage: NetworkImage(user.avatar)),
)
```

</details>

---

### Вопрос 5 🟡

Как использовать Hero с кастомной `PageRoute` для управления кривой анимации?

- A) Нельзя — Hero использует стандартную кривую
- B) `MaterialPageRoute(builder: ...).then` + кастомный `PageTransitionsTheme`
- C) Использовать `PageRouteBuilder` с `transitionDuration` и `transitionsBuilder` — Hero подстраивается под длительность маршрута
- D) `Hero(curve: Curves.elasticOut, ...)`

<details>
<summary>Ответ</summary>

**C) `PageRouteBuilder` — Hero использует `transitionDuration` маршрута**

```dart
Navigator.push(ctx, PageRouteBuilder(
  transitionDuration: const Duration(milliseconds: 600),
  reverseTransitionDuration: const Duration(milliseconds: 400),
  pageBuilder: (ctx, animation, secondary) => const DetailScreen(),
  transitionsBuilder: (ctx, animation, secondary, child) {
    // Фоновый переход (Hero анимируется отдельно и поверх):
    return FadeTransition(opacity: animation, child: child);
  },
));
```

Hero duration = маршрута duration; Hero curve настраивается через `createRectTween`.

</details>

---

### Вопрос 6 🟡

Как настроить форму траектории Hero через `createRectTween`?

- A) `Hero(curve: Curves.easeIn, ...)` — только через curve
- B) Переопределить `createRectTween` в `HeroController` или использовать `Hero(createRectTween: ...)` — контролирует путь движения
- C) `Hero(path: PathType.curved, ...)`
- D) Траектория всегда прямолинейная

<details>
<summary>Ответ</summary>

**B) `Hero(createRectTween: ...)` — кастомная траектория**

```dart
// Дуговая траектория:
Hero(
  tag: 'item',
  createRectTween: (begin, end) {
    return MaterialRectArcTween(begin: begin, end: end); // дуга
    // или:
    // return RectTween(begin: begin, end: end); // прямая (по умолчанию)
  },
  child: ...,
)

// Глобально через MaterialApp:
MaterialApp(
  home: HeroControllerScope(
    controller: MaterialApp.createMaterialHeroController(),
    // MaterialRectArcTween используется по умолчанию в Material!
    child: MyApp(),
  ),
)
```

</details>

---

### Вопрос 7 🟡

Как правильно использовать Hero с вложенными Navigator?

- A) Hero не работает с вложенными Navigator
- B) Hero летит только в пределах одного Navigator; для корневого Navigator — `rootNavigator: true` в навигации или `HeroControllerScope` с общим контроллером
- C) Нужен отдельный `HeroController` для каждого Navigator
- D) Автоматически проходит через все уровни Navigator

<details>
<summary>Ответ</summary>

**B) Hero ограничен уровнем Navigator; для cross-navigator — настройка нужна**

```dart
// Проблема: Hero в nested Navigator не летит к root экрану

// Решение 1: rootNavigator: true для перехода на root уровень:
Navigator.of(ctx, rootNavigator: true).push(routeWithHero);

// Решение 2: общий HeroController для всех уровней:
final heroController = HeroController();

MaterialApp(
  navigatorKey: rootNavigatorKey,
  home: HeroControllerScope(
    controller: heroController,
    child: Navigator(
      observers: [heroController], // тот же контроллер
      ...
    ),
  ),
)
```

</details>

---

### Вопрос 8 🔴

Как решить проблему Hero анимации когда виджет обрезается (clipBehavior) во время полёта?

- A) Hero автоматически убирает clip
- B) `Hero` поднимает "летящий" виджет в overlay — clip родительских виджетов не применяется; решение — избегать clip или использовать `placeholderBuilder`
- C) `Hero(clipBehavior: Clip.none, ...)`
- D) Проблемы с clip не существует в Hero

<details>
<summary>Ответ</summary>

**B) Hero в overlay — clip не применяется; `placeholderBuilder` для управления источником**

```dart
Hero(
  tag: 'card',
  // Пока виджет "летит", его место занимает placeholder:
  placeholderBuilder: (ctx, heroSize, child) {
    return SizedBox(
      width: heroSize.width,
      height: heroSize.height,
      // Пусто или заглушка (по умолчанию — прозрачный SizedBox)
    );
  },
  child: Card(
    clipBehavior: Clip.antiAlias, // clipBehavior только на destination
    child: Image.network(url),
  ),
)
```

</details>

---

### Вопрос 9 🔴

Как реализовать Hero анимацию для `Text` с изменением стиля?

- A) Hero на Text не работает корректно
- B) Hero на Text работает — но часто нужен `DefaultTextStyle.merge()` или `Material` виджет-обёртка для корректного рендеринга во время полёта
- C) Использовать `AnimatedDefaultTextStyle` вместо Hero
- D) Только через кастомный `flyingShuttleBuilder` с анимацией TextStyle

<details>
<summary>Ответ</summary>

**B) Hero на Text нужна обёртка в `Material` для корректного рендеринга**

```dart
// Hero без Material может показывать желтые/черные borders во время анимации:
Hero(
  tag: 'title-${item.id}',
  child: Material( // ВАЖНО: обернуть в Material!
    type: MaterialType.transparency,
    child: Text(
      item.title,
      style: const TextStyle(fontSize: 24, color: Colors.black),
    ),
  ),
)

// Destination с другим стилем:
Hero(
  tag: 'title-${item.id}',
  child: Material(
    type: MaterialType.transparency,
    child: Text(
      item.title,
      style: const TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
    ),
  ),
)
```

</details>

---

### Вопрос 10 🔴

Как протестировать Hero анимацию в `testWidgets`?

- A) Hero не тестируется в Widget tests
- B) `tester.pump()` / `pumpAndSettle()` по тегу; для Hero нужен `MaterialApp` с `navigatorObservers` и `pumpAndSettle` для полного завершения
- C) `tester.heroTest(tag)`
- D) Только в integration tests на реальном устройстве

<details>
<summary>Ответ</summary>

**B) `testWidgets` с MaterialApp + `pumpAndSettle()` для полного завершения**

```dart
testWidgets('Hero animates between screens', (tester) async {
  await tester.pumpWidget(MaterialApp(
    home: ScreenOne(),
    routes: {'/detail': (_) => DetailScreen()},
  ));

  // Найти Hero виджет:
  expect(find.byKey(const ValueKey('hero-img')), findsOneWidget);

  // Нажать для навигации:
  await tester.tap(find.byType(InkWell));
  await tester.pump(); // начать переход
  await tester.pump(const Duration(milliseconds: 300)); // середина анимации

  // Hero "летит":
  expect(find.byType(Hero), findsWidgets);

  await tester.pumpAndSettle(); // завершить анимацию

  // Проверить destination:
  expect(find.byType(DetailScreen), findsOneWidget);
});
```

</details>
