# Квиз: Riverpod

**Тема:** 07.3 — Riverpod  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как создать простой `Provider` в Riverpod?

- A) `final myProvider = Provider((ref) => MyService());`
- B) `final myProvider = RiverpodProvider(MyService())`
- C) `Provider<MyService>.create(() => MyService())`
- D) `@provider MyService myService = MyService()`

<details>
<summary>Ответ</summary>

**A) `final myProvider = Provider((ref) => MyService());`**

Глобальная переменная `Provider` — не синглтон в традиционном смысле, а дескриптор. Riverpod создаёт экземпляр лениво при первом использовании. `ref` — для доступа к другим провайдерам через `ref.watch`/`ref.read`.

</details>

---

### Вопрос 2 🟢

Как прочитать провайдер в `ConsumerWidget`?

- A) `context.read<MyProvider>()`
- B) `ref.watch(myProvider)` в `build(context, ref)`
- C) `myProvider.value`
- D) `Riverpod.of(context, myProvider)`

<details>
<summary>Ответ</summary>

**B) `ref.watch(myProvider)` в `build(context, ref)`**

`ConsumerWidget` получает `WidgetRef ref` вторым параметром `build`. `ref.watch(provider)` — подписка с rebuild при изменении. Отличие от Provider: не нужен `BuildContext` для чтения.

</details>

---

### Вопрос 3 🟢

Что такое `StateNotifierProvider`?

- A) Provider для `ChangeNotifier`
- B) Provider для `StateNotifier<State>` — неизменяемое состояние с методами для изменения
- C) Provider для `Stream`
- D) Устаревший тип, заменён `NotifierProvider`

<details>
<summary>Ответ</summary>

**B) Provider для `StateNotifier<State>`**

```dart
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);
  void increment() => state = state + 1;
}

final counterProvider = StateNotifierProvider<CounterNotifier, int>(
  (ref) => CounterNotifier(),
);
// Использование: ref.watch(counterProvider) → int
// Вызов метода: ref.read(counterProvider.notifier).increment()
```

</details>

---

### Вопрос 4 🟡

Чем Riverpod отличается от Provider (package:provider)?

- A) Они идентичны — только разный синтаксис
- B) Riverpod: compile-safe (нет ProviderNotFoundException), работает вне BuildContext, поддерживает переопределение провайдеров, auto-dispose, нет зависимости от дерева виджетов
- C) Riverpod только для Dart приложений
- D) Provider быстрее Riverpod

<details>
<summary>Ответ</summary>

**B) Riverpod устраняет ограничения Provider**

Ключевые отличия:

- **Compile-safe**: попытка читать несуществующий провайдер → ошибка компиляции
- **Без BuildContext**: провайдеры читаются через `ref`, не через `context`
- **Override**: лёгкое мокирование в тестах через `ProviderContainer.override`
- **Auto-dispose**: провайдеры освобождаются когда не используются
- **Семейства**: параметризованные провайдеры через `.family`

</details>

---

### Вопрос 5 🟡

Что такое `NotifierProvider` (Riverpod 2.0)?

- A) Новое название для `StateNotifierProvider`
- B) Современный тип провайдера с классом `Notifier` — заменяет `StateNotifier` с упрощённым синтаксисом
- C) Provider для уведомлений push
- D) `ChangeNotifierProvider` в Riverpod 2.0

<details>
<summary>Ответ</summary>

**B) Современный тип провайдера с классом `Notifier`**

```dart
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0; // начальное состояние

  void increment() => state++;
}
// Использование: ref.watch(counterProvider) → int
// Метод: ref.read(counterProvider.notifier).increment()
```

Аннотация `@riverpod` генерирует `counterProvider` через `build_runner`.

</details>

---

### Вопрос 6 🟡

Что делает `.family` модификатор в Riverpod?

- A) Группирует провайдеры по семейству
- B) Создаёт параметризованный провайдер — отдельный экземпляр для каждого уникального параметра
- C) Ограничивает область видимости провайдера
- D) Объединяет несколько провайдеров

<details>
<summary>Ответ</summary>

**B) Создаёт параметризованный провайдер**

```dart
final userProvider = FutureProvider.family<User, int>((ref, userId) async {
  return api.getUser(userId);
});
// Использование:
ref.watch(userProvider(42)) // User с id=42
ref.watch(userProvider(99)) // другой User с id=99
```

Каждый параметр → отдельный кэшированный экземпляр.

</details>

---

### Вопрос 7 🟡

Что такое `autoDispose` и когда его использовать?

- A) Автоматически сохраняет состояние
- B) Провайдер автоматически уничтожается когда ни один виджет не слушает его — освобождает память
- C) Отключает `dispose()` метод
- D) Только для `FutureProvider`

<details>
<summary>Ответ</summary>

**B) Провайдер уничтожается когда никто не слушает — освобождает память**

```dart
final searchProvider = StateProvider.autoDispose<String>((ref) => '');
// При уходе с экрана поиска — провайдер уничтожается
// При возврате — создаётся заново (чистый)
```

Без `autoDispose` данные остаются в памяти на протяжении всей жизни приложения. С `ref.keepAlive()` можно предотвратить смерть провайдера при временном отсутствии слушателей.

</details>

---

### Вопрос 8 🔴

Как обработать `AsyncValue` из `FutureProvider`?

- A) `asyncValue.data` → данные или `null`
- B) `asyncValue.when(data: (v) => ..., loading: () => ..., error: (e, s) => ...)`
- C) `await asyncValue.future`
- D) `asyncValue.asData!.value`

<details>
<summary>Ответ</summary>

**B) `asyncValue.when(data: ..., loading: ..., error: ...)`**

```dart
final user = ref.watch(userProvider);
return user.when(
  data: (user) => Text(user.name),
  loading: () => CircularProgressIndicator(),
  error: (e, stack) => Text('Ошибка: $e'),
);
```

Также: `user.maybeWhen(data: (u) => ..., orElse: () => ...)`, `user.map(...)`. `user.value` — nullable значение без обработки состояний.

</details>

---

### Вопрос 9 🔴

Как переопределить провайдер в тестах без изменения production кода?

- A) `Riverpod.mock<MyProvider>(MockService())`
- B) `ProviderContainer(overrides: [myProvider.overrideWith((ref) => MockService())])` или `ProviderScope(overrides: [...])` для виджет-тестов
- C) Нельзя — провайдеры не переопределяются
- D) `@VisibleForTesting` аннотация

<details>
<summary>Ответ</summary>

**B) `ProviderScope(overrides: [...])` для виджет-тестов**

```dart
testWidgets('...', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        apiProvider.overrideWith((ref) => MockApiService()),
      ],
      child: MyApp(),
    ),
  );
});
```

`ProviderContainer` — для unit-тестов без Flutter.

</details>

---

### Вопрос 10 🔴

Что такое `ref.invalidate` и `ref.refresh` в Riverpod?

- A) Инвалидируют кэш БД
- B) `ref.invalidate(provider)` — помечает провайдер устаревшим (пересоздаст при следующем чтении); `ref.refresh(provider)` — инвалидирует и **немедленно** пересоздаёт
- C) Удаляют провайдер из памяти
- D) Сбрасывают `autoDispose` таймер

<details>
<summary>Ответ</summary>

**B) `invalidate` — ленивая; `refresh` — немедленная пересборка**

```dart
// Обновить список после добавления элемента:
ref.invalidate(todoListProvider); // пересоздастся при следующем watch

// Force-refresh FutureProvider:
ref.refresh(userProvider); // немедленный новый запрос
```

`invalidate` предпочтительнее для debounce-логики. `refresh` возвращает новое значение (для FutureProvider — `Future`).

</details>
