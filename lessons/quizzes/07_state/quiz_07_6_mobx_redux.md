# Квиз: MobX / Redux (другие подходы)

**Тема:** 07.6 — MobX / Redux  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что отличает Redux от других подходов управления состоянием?

- A) Redux использует Streams
- B) Единое неизменяемое дерево состояния (Store) + чистые функции (Reducers) + Action-объекты
- C) Redux — только для веб-приложений
- D) Redux автоматически синхронизирует состояние с сервером

<details>
<summary>Ответ</summary>

**B) Единый Store + чистые функции Reducers + Actions**

Redux принципы:

1. **Single source of truth**: одно дерево состояния
2. **State is read-only**: меняется только через Actions
3. **Pure reducers**: `reducer(state, action) → newState` — без side effects

`flutter_redux` пакет интегрирует Redux с Flutter через `StoreProvider`/`StoreConnector`.

</details>

---

### Вопрос 2 🟢

Как MobX обеспечивает реактивность?

- A) Через `Stream` подписки
- B) `@observable` аннотации + реактивный граф зависимостей + автоматический tracking при чтении в `@computed`/`@action`
- C) Через `ChangeNotifier`
- D) MobX использует Proxy из JavaScript

<details>
<summary>Ответ</summary>

**B) `@observable` + реактивный граф зависимостей**

MobX для Dart через кодогенерацию:

```dart
part 'counter.g.dart';

class Counter = _Counter with _$Counter;

abstract class _Counter with Store {
  @observable
  int count = 0;

  @computed
  String get countString => count.toString();

  @action
  void increment() => count++;
}
```

</details>

---

### Вопрос 3 🟢

Что такое `Reducer` в Redux?

- A) Функция уменьшающая количество виджетов
- B) Чистая функция `(State, Action) => State` — обрабатывает действие и возвращает новое состояние
- C) Класс для асинхронных операций
- D) Observer для Store

<details>
<summary>Ответ</summary>

**B) Чистая функция `(State, Action) => State`**

```dart
AppState appReducer(AppState state, dynamic action) {
  if (action is IncrementAction) {
    return state.copyWith(counter: state.counter + 1);
  }
  return state;
}
```

"Чистая" — нет side effects, нет мутации, одинаковые inputs → одинаковый output. Хорошо тестируется.

</details>

---

### Вопрос 4 🟡

Как использовать Store из Redux в виджете?

- A) `StoreProvider.of<AppState>(context).state`
- B) `StoreConnector<AppState, ViewModel>(converter: ..., builder: ...)` или `StoreBuilder`
- C) `context.watch<Store<AppState>>()`
- D) `Redux.of(context).state`

<details>
<summary>Ответ</summary>

**B) `StoreConnector` или `StoreBuilder`**

```dart
StoreConnector<AppState, int>(
  converter: (store) => store.state.counter,
  builder: (context, count) => Text('$count'),
)
// Dispatch:
StoreProvider.of<AppState>(context).dispatch(IncrementAction());
```

`converter` строит ViewModel из Store — виджет перестраивается только при изменении ViewModel.

</details>

---

### Вопрос 5 🟡

Что такое `Middleware` в Redux?

- A) Промежуточный слой между Store и UI
- B) Функция вызываемая при каждом `dispatch(action)` — используется для логирования, async операций, сайд-эффектов
- C) Middleware — только для аутентификации
- D) Аналог `BlocObserver`

<details>
<summary>Ответ</summary>

**B) Функция при каждом dispatch — для async операций и side effects**

```dart
void loggingMiddleware(Store<AppState> store, action, NextDispatcher next) {
  print('Dispatching: $action | Current: ${store.state}');
  next(action); // передать дальше
  print('Next state: ${store.state}');
}

Store(appReducer, middleware: [loggingMiddleware, thunkMiddleware]);
```

Redux Thunk — для async actions: `store.dispatch((store) async { ... })`.

</details>

---

### Вопрос 6 🟡

Что такое `@computed` в MobX?

- A) Синоним `@observable`
- B) Аннотация для производных значений — кэшированные вычисления, пересчитываются только при изменении зависимых observables
- C) Метка для методов которые изменяют state
- D) Декоратор для async методов

<details>
<summary>Ответ</summary>

**B) Производные кэшированные значения — пересчитываются при изменении dependables**

```dart
@computed
String get fullName => '${firstName} ${lastName}';
// Пересчитывается только при изменении firstName или lastName
```

Аналог `useMemo` в React Hooks. Мемоизация встроена — не пересчитывается лишний раз.

</details>

---

### Вопрос 7 🟡

Как обработать асинхронные операции в MobX?

- A) Через обычный `async/await` в `@action`
- B) `ObservableFuture` и `runInAction` для обновления state из async контекста
- C) Нельзя — MobX только синхронный
- D) Через `@asyncAction` аннотацию

<details>
<summary>Ответ</summary>

**B) `ObservableFuture` и `runInAction`**

```dart
@observable
ObservableFuture<User?> userFuture = ObservableFuture.value(null);

@action
Future<void> fetchUser(int id) async {
  userFuture = ObservableFuture(api.getUser(id));
  final user = await userFuture;
  runInAction(() {
    currentUser = user; // изменение observable вне action — через runInAction
  });
}
```

</details>

---

### Вопрос 8 🔴

Когда Redux уместнее Riverpod в корпоративном Flutter проекте?

- A) Redux всегда лучше Riverpod
- B) Redux уместен когда: команда знакома с Redux (React background), нужна полная история действий (time-travel debugging), строгая unidirectional flow обязательна по архитектурным требованиям
- C) Redux только для проектов без backend
- D) Riverpod и Redux одинаково подходят — нет разницы

<details>
<summary>Ответ</summary>

**B) Redux уместен при команде с Redux-opытом и потребности в time-travel debugging**

В Flutter экосистеме Redux используется реже чем в React/JS. Предпочтения: большинство Flutter команд выбирают Riverpod/BLoC. Redux — когда проект мигрирует из React Native, или нужен Redux DevTools для инспекции состояния.

</details>

---

### Вопрос 9 🔴

Как генерировать MobX код через `build_runner`?

- A) `flutter pub run mobx:generate`
- B) `flutter pub run build_runner build` — генерирует `.g.dart` файлы
- C) `dart run mobx_codegen`
- D) Вручную — кодогенерация не используется

<details>
<summary>Ответ</summary>

**B) `flutter pub run build_runner build`**

Зависимости в `pubspec.yaml`:

```yaml
dependencies:
  mobx: ^2.x
  flutter_mobx: ^2.x
dev_dependencies:
  build_runner: ^2.x
  mobx_codegen: ^2.x
```

`build_runner watch` — непрерывная генерация. `build_runner build --delete-conflicting-outputs` — при конфликтах.

</details>

---

### Вопрос 10 🔴

Как реализовать `@reaction` в MobX для реакции на изменения без UI rebuild?

- A) `reaction()` это `@computed` с side effect
- B) `reaction((p0) => observableValue, (value) => sideEffect())` — выполняет коллбэк при изменении отслеживаемого значения
- C) `@reaction void onCountChange() => ...`
- D) Только через `Observer` виджет

<details>
<summary>Ответ</summary>

**B) `reaction((p0) => observableValue, (value) => sideEffect())`**

```dart
@override
void onInit() {
  super.onInit();
  // Сохранять в SharedPreferences при изменении:
  _disposer = reaction(
    (_) => store.isDarkMode,
    (isDark) => prefs.setBool('dark', isDark),
  );
}

void dispose() {
  _disposer(); // отменить reaction
}
```

Также: `autorun` (запускается сразу), `when` (однократно при условии).

</details>
