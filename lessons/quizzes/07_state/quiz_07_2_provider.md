# Квиз: Provider

**Тема:** 07.2 — Provider  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как предоставить объект через Provider?

- A) `StatefulWidget(provider: MyModel())`
- B) `ChangeNotifierProvider(create: (_) => MyModel(), child: MyApp())`
- C) `Provider.attach(context, MyModel())`
- D) `InheritedProvider(value: MyModel())`

<details>
<summary>Ответ</summary>

**B) `ChangeNotifierProvider(create: (_) => MyModel(), child: MyApp())`**

`ChangeNotifierProvider` создаёт и предоставляет `ChangeNotifier` поддереву. `create` вызывается лениво при первом использовании. `dispose()` модели вызывается автоматически когда Provider удаляется из дерева.

</details>

---

### Вопрос 2 🟢

Как прочитать значение из Provider?

- A) `Provider.get<MyModel>(context)`
- B) `context.watch<MyModel>()` или `context.read<MyModel>()`
- C) `Provider.listen<MyModel>(context)`
- D) `MyModel.of(context)`

<details>
<summary>Ответ</summary>

**B) `context.watch<MyModel>()` или `context.read<MyModel>()`**

- `context.watch<T>()` — подписывается на изменения, вызывает rebuild при уведомлении
- `context.read<T>()` — только читает, не подписывается (для `onPressed` и т.п.)
- `context.select<T, R>((m) => m.field)` — подписывается только на `field`

</details>

---

### Вопрос 3 🟢

Что такое `ChangeNotifier`?

- A) Синглтон для хранения глобального состояния
- B) Flutter класс с `notifyListeners()` и `addListener()`/`removeListener()` — основа для моделей в Provider
- C) Обёртка над `Stream`
- D) Аналог Redux store

<details>
<summary>Ответ</summary>

**B) Flutter класс с `notifyListeners()` и `addListener()`/`removeListener()`**

```dart
class CounterModel extends ChangeNotifier {
  int _count = 0;
  int get count => _count;

  void increment() {
    _count++;
    notifyListeners(); // уведомить слушателей → rebuild
  }
}
```

`ChangeNotifierProvider` автоматически подписывается и отписывается.

</details>

---

### Вопрос 4 🟡

Чем `context.watch` отличается от `context.read`?

- A) Они идентичны
- B) `watch` — подписывается на изменения (используй в `build()`); `read` — только читает текущее значение (используй в коллбэках)
- C) `read` работает асинхронно
- D) `watch` работает только с `ChangeNotifier`

<details>
<summary>Ответ</summary>

**B) `watch` — подписка в `build()`; `read` — чтение в обработчиках событий**

```dart
// В build():
final counter = context.watch<CounterModel>().count; // rebuild при изменении

// В onPressed:
onPressed: () => context.read<CounterModel>().increment(), // нет подписки
```

`context.watch` в коллбэке — ошибка (виджет может быть уже unmounted).

</details>

---

### Вопрос 5 🟡

Как предоставить несколько провайдеров?

- A) `MultiProvider(providers: [ChangeNotifierProvider(...), Provider(...)])`
- B) Вложить провайдеры друг в друга
- C) `ProviderGroup(providers: [...])`
- D) A и B оба работают; A — более читаем

<details>
<summary>Ответ</summary>

**D) A и B оба работают; `MultiProvider` — предпочтительнее**

```dart
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => AuthModel()),
    ChangeNotifierProvider(create: (_) => CartModel()),
    Provider(create: (_) => ApiService()),
  ],
  child: MyApp(),
)
```

Вложенные провайдеры читаемы при 1-2 штуках, `MultiProvider` — при 3+.

</details>

---

### Вопрос 6 🟡

Что такое `ProxyProvider`?

- A) Provider для HTTP запросов
- B) Provider зависящий от другого Provider — создаёт/обновляет объект при изменении зависимости
- C) Синхронная версия `FutureProvider`
- D) Alias для `ChangeNotifierProvider`

<details>
<summary>Ответ</summary>

**B) Provider зависящий от другого Provider**

```dart
ProxyProvider<AuthModel, ApiService>(
  update: (ctx, auth, previous) => ApiService(token: auth.token),
)
```

При изменении `AuthModel.token` → пересоздаётся `ApiService` с новым токеном. `ChangeNotifierProxyProvider` — аналог для `ChangeNotifier`.

</details>

---

### Вопрос 7 🟡

Как подписаться на отдельное поле модели, а не на всю модель?

- A) `context.watch<Model>().field` — подписка только на field
- B) `context.select<Model, Type>((model) => model.field)` — rebuild только при изменении `field`
- C) `FieldProvider<Model, field>`
- D) Невозможно — Provider не поддерживает гранулярную подписку

<details>
<summary>Ответ</summary>

**B) `context.select<Model, Type>((m) => m.field)`**

```dart
// Перестраивается только при изменении username:
final username = context.select<UserModel, String>((m) => m.username);
```

`context.watch<UserModel>()` — перестраивается при **любом** изменении `UserModel`. `select` — только при изменении выбранного значения.

</details>

---

### Вопрос 8 🔴

Почему `context.read` не должен вызываться в `build()` метода?

- A) Это вызовет ошибку компиляции
- B) `read` не подписывается → виджет не будет перестраиваться при изменениях → устаревшие данные в UI
- C) `read` устарел в новых версиях Provider
- D) Это влияет только на производительность

<details>
<summary>Ответ</summary>

**B) `read` не подписывается → виджет не обновится при изменениях**

Если в `build()` нужны данные — используй `watch` или `select`. `read` — только для не-UI логики: `onPressed`, `initState` (когда нужен объект, но не его данные во время build), callback-ов. Дополнительно: вызов `read` внутри `build()` может вызвать `ProviderNotFoundError` если build происходит после rebuild дерева.

</details>

---

### Вопрос 9 🔴

Что такое `Consumer` виджет и когда его использовать вместо `context.watch`?

- A) Они полностью идентичны — только синтаксис разный
- B) `Consumer` позволяет минимизировать зону ребилда — только часть виджета перестраивается, когда `watch` заставил бы перестроить весь `build()`
- C) `Consumer` используется только в `StatelessWidget`
- D) `Consumer` — deprecated API

<details>
<summary>Ответ</summary>

**B) `Consumer` минимизирует зону rebuild**

```dart
Column(children: [
  HeavyWidget(), // не перестраивается
  Consumer<CounterModel>(
    builder: (ctx, counter, child) => Text('${counter.count}'),
  ),
])
```

`child` параметр Consumer — виджет внутри builder который **не** перестраивается при rebuild. `context.watch` перестроит весь `build()` текущего виджета.

</details>

---

### Вопрос 10 🔴

Как избежать `ProviderNotFoundException` при тестировании?

- A) `ProviderNotFoundException` невозможен в тестах
- B) Обернуть тестируемый виджет в провайдеры: `pumpWidget(ChangeNotifierProvider(create: (_) => MockModel(), child: MyWidget()))`
- C) Использовать `TestProvider.mock<T>()`
- D) Отключить Provider в тестах

<details>
<summary>Ответ</summary>

**B) Обернуть тестируемый виджет в нужные Provider-ы с mock-моделями**

```dart
testWidgets('counter increments', (tester) async {
  await tester.pumpWidget(
    ChangeNotifierProvider<CounterModel>(
      create: (_) => MockCounterModel(),
      child: MaterialApp(home: CounterScreen()),
    ),
  );
  await tester.tap(find.byIcon(Icons.add));
  await tester.pump();
  expect(find.text('1'), findsOneWidget);
});
```

</details>
