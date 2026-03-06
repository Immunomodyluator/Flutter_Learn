# Квиз: BLoC / Cubit

**Тема:** 07.4 — BLoC  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое BLoC паттерн?

- A) База данных для Flutter приложений
- B) Business Logic Component — слой бизнес-логики, отделяющий UI от данных, использующий Streams
- C) Аналог MVC для Flutter
- D) Пакет для работы с REST API

<details>
<summary>Ответ</summary>

**B) Business Logic Component — слой бизнес-логики с Streams**

BLoC разделяет: UI (виджеты) → Events → BLoC → States → UI. Входные данные — события (`Event`), выходные — состояния (`State`). Пакет `flutter_bloc` предоставляет `Bloc` и упрощённый `Cubit`.

</details>

---

### Вопрос 2 🟢

Чем `Cubit` отличается от `Bloc`?

- A) `Cubit` использует Streams, `Bloc` использует ChangeNotifier
- B) `Cubit` — упрощённый `Bloc` без Events: методы напрямую эмитируют новые состояния
- C) `Bloc` только для глобального состояния
- D) `Cubit` поддерживает только примитивные типы

<details>
<summary>Ответ</summary>

**B) `Cubit` — без Events, методы эмитируют состояния напрямую**

```dart
// Cubit:
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1);
}

// Bloc:
class CounterBloc extends Bloc<CounterEvent, int> {
  CounterBloc() : super(0) {
    on<IncrementEvent>((event, emit) => emit(state + 1));
  }
}
```

</details>

---

### Вопрос 3 🟢

Как предоставить BLoC виджетам?

- A) `MultiBlocProvider` + `BlocProvider`
- B) `BlocProvider(create: (_) => CounterCubit(), child: MyWidget())`
- C) `Provider(create: (_) => CounterCubit())`
- D) A и B оба корректны

<details>
<summary>Ответ</summary>

**D) A и B — оба корректны**

```dart
// Один:
BlocProvider(create: (_) => MyCubit(), child: ...)

// Несколько:
MultiBlocProvider(providers: [
  BlocProvider(create: (_) => AuthBloc()),
  BlocProvider(create: (_) => CartCubit()),
], child: ...)
```

`BlocProvider.value(value: existingBloc)` — передаёт уже созданный bloc (не управляет lifecycle).

</details>

---

### Вопрос 4 🟡

Как подписаться на состояние BLoC в виджете?

- A) `context.watch<MyCubit>()`
- B) `BlocBuilder<MyCubit, MyState>(builder: (ctx, state) => ...)`
- C) `StreamBuilder(stream: cubit.stream, builder: ...)`
- D) B и C оба работают; B — предпочтительнее

<details>
<summary>Ответ</summary>

**D) B и C оба работают; `BlocBuilder` — предпочтительнее**

`BlocBuilder` — удобная обёртка над `StreamBuilder`:

```dart
BlocBuilder<CounterCubit, int>(
  buildWhen: (prev, current) => prev != current, // опционально
  builder: (context, count) => Text('$count'),
)
```

`buildWhen` — аналог `shouldRebuild`, оптимизирует перестройки.

</details>

---

### Вопрос 5 🟡

Что такое `BlocListener` и когда его использовать?

- A) Подписка которая не вызывает rebuild — для side effects
- B) Listener для отладки BLoC
- C) Альтернатива `BlocBuilder` с подпиской
- D) Observer для множественных BLoC

<details>
<summary>Ответ</summary>

**A) Подписка без rebuild — для side effects**

```dart
BlocListener<AuthBloc, AuthState>(
  listenWhen: (prev, curr) => curr is AuthError,
  listener: (context, state) {
    if (state is AuthError) {
      ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text(state.message)));
    }
  },
  child: LoginForm(),
)
```

Используй для: показа SnackBar, диалогов, навигации, аналитики.

</details>

---

### Вопрос 6 🟡

Что такое `BlocConsumer`?

- A) Потребитель множественных BLoC
- B) Комбинирует `BlocBuilder` + `BlocListener` в одном виджете
- C) Расширенный `BlocBuilder` с кэшированием
- D) BLoC + Provider интеграция

<details>
<summary>Ответ</summary>

**B) Комбинирует `BlocBuilder` + `BlocListener` в одном виджете**

```dart
BlocConsumer<LoginCubit, LoginState>(
  listener: (ctx, state) {
    if (state.isSuccess) context.go('/home');
  },
  builder: (ctx, state) {
    if (state.isLoading) return CircularProgressIndicator();
    return LoginForm();
  },
)
```

Используй когда нужно и обновить UI, и выполнить side effect.

</details>

---

### Вопрос 7 🟡

Как добавить error handling в BLoC?

- A) `try/catch` внутри event handler с эмиссией Error-состояния
- B) `BlocObserver.onError`
- C) `addError(exception, stackTrace)` метод BLoC
- D) Все три подхода являются правильными

<details>
<summary>Ответ</summary>

**D) Все три подхода правильны в разных ситуациях**

```dart
// В event handler:
on<FetchDataEvent>((event, emit) async {
  try {
    final data = await api.getData();
    emit(DataLoaded(data));
  } catch (e) {
    emit(DataError(e.toString()));
  }
});
```

`addError` — для нечастых/неожиданных ошибок. `BlocObserver` — для глобального логирования ошибок.

</details>

---

### Вопрос 8 🔴

Как реализовать `BlocObserver` для логирования всех событий и состояний?

- A) `BlocObserver` не существует в flutter_bloc
- B) Расширить `BlocObserver`, переопределить методы, зарегистрировать через `Bloc.observer = MyObserver()`
- C) Через `Flutter.log()`
- D) `BlocProvider.observer = ...`

<details>
<summary>Ответ</summary>

**B) Расширить `BlocObserver` и зарегистрировать через `Bloc.observer`**

```dart
class AppBlocObserver extends BlocObserver {
  @override
  void onChange(BlocBase bloc, Change change) {
    super.onChange(bloc, change);
    log('${bloc.runtimeType}: ${change.currentState} → ${change.nextState}');
  }
  @override
  void onError(BlocBase bloc, Object error, StackTrace stackTrace) {
    super.onError(bloc, error, stackTrace);
    log('ERROR in ${bloc.runtimeType}: $error');
  }
}
// main.dart:
Bloc.observer = AppBlocObserver();
```

</details>

---

### Вопрос 9 🔴

Как тестировать BLoC с помощью `bloc_test` пакета?

- A) Обычный `test()` с ручным вызовом методов
- B) `blocTest<MyCubit, MyState>('description', build: () => MyCubit(), act: (c) => c.method(), expect: () => [State1, State2])`
- C) `BlocTester(bloc).expect([...])`
- D) `WidgetTest` — единственный способ тестировать BLoC

<details>
<summary>Ответ</summary>

**B) `blocTest` из пакета `bloc_test`**

```dart
blocTest<CounterCubit, int>(
  'increment emits [1]',
  build: () => CounterCubit(),
  act: (cubit) => cubit.increment(),
  expect: () => [1],
);

blocTest<CounterCubit, int>(
  'increment twice emits [1, 2]',
  build: () => CounterCubit(),
  act: (cubit) {
    cubit.increment();
    cubit.increment();
  },
  expect: () => [1, 2],
);
```

</details>

---

### Вопрос 10 🔴

Когда использовать `Bloc` (с Events) вместо `Cubit`?

- A) Всегда — `Cubit` только для учебных целей
- B) `Bloc` предпочтителен когда: нужна история событий, debounce/throttle событий, события из нескольких источников, сложная event-трансформация через `EventTransformer`
- C) `Bloc` только для серверных данных
- D) Они полностью взаимозаменяемы — выбор не имеет значения

<details>
<summary>Ответ</summary>

**B) `Bloc` для сложной event-обработки**

`EventTransformer` — ключевое преимущество `Bloc`:

```dart
// Debounce поиска:
on<SearchEvent>(
  (event, emit) async => emit(await search(event.query)),
  transformer: debounce(Duration(milliseconds: 300)),
);
```

Пакет `bloc_concurrency` предоставляет: `sequential`, `concurrent`, `droppable`, `restartable`.

</details>
