# Квиз: GetX

**Тема:** 07.5 — GetX  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое GetX в Flutter?

- A) Пакет только для навигации
- B) Всё-в-одном пакет: state management, навигация, dependency injection, утилиты
- C) Аналог Provider от Google
- D) Инструмент для кодогенерации

<details>
<summary>Ответ</summary>

**B) Всё-в-одном пакет: state management, навигация, DI, утилиты**

GetX предоставляет: `GetxController` (state), `Get.to`/`Get.back` (навигация), `Get.put`/`Get.find` (DI), `Obx` (reactive widget), `GetBuilder` (simple state), снэкбар/диалоги/боттомшиты без context.

</details>

---

### Вопрос 2 🟢

Как создать реактивную переменную в GetX?

- A) `ReactiveVar<int> count = ReactiveVar(0)`
- B) `var count = 0.obs` — `.obs` делает переменную наблюдаемой (Observable)
- C) `Observable<int> count = Observable(0)`
- D) `GetX.reactive(0, name: 'count')`

<details>
<summary>Ответ</summary>

**B) `var count = 0.obs` — `.obs` делает переменную наблюдаемой**

```dart
class CounterController extends GetxController {
  var count = 0.obs;
  void increment() => count++;
}
// Использование в UI:
final c = Get.find<CounterController>();
Obx(() => Text('${c.count}'))
```

</details>

---

### Вопрос 3 🟢

Как перейти на новый экран с помощью GetX?

- A) `Navigator.of(context).push(...)`
- B) `Get.to(() => NewScreen())`
- C) `GetNavigation.push(NewScreen)`
- D) `context.go('/new')`

<details>
<summary>Ответ</summary>

**B) `Get.to(() => NewScreen())`**

GetX навигация не требует `context`. Методы: `Get.to()`, `Get.back()`, `Get.off()` (replace), `Get.offAll()` (clear stack), `Get.toNamed('/home')` (именованный). Настройка в `GetMaterialApp(getPages: [...])`.

</details>

---

### Вопрос 4 🟡

Чем `Obx` отличается от `GetBuilder`?

- A) Они идентичны
- B) `Obx` — реактивный, перестраивается при изменении `.obs` переменных; `GetBuilder` — простой, перестраивается по `update()` вызову
- C) `GetBuilder` быстрее `Obx`
- D) `Obx` устарел в пользу `GetBuilder`

<details>
<summary>Ответ</summary>

**B) `Obx` — реактивный (observable); `GetBuilder` — простой (update-based)**

```dart
// Obx — автоматически отслеживает obs переменные:
Obx(() => Text('${controller.count}'))

// GetBuilder — нужно явно вызвать update():
GetBuilder<CounterController>(
  builder: (c) => Text('${c.count}'),
)
// В контроллере:
void increment() { count++; update(); } // явный вызов
```

</details>

---

### Вопрос 5 🟡

Как зарегистрировать зависимость в GetX DI?

- A) `GetX.register(MyService())`
- B) `Get.put(MyService())` или `Get.lazyPut(() => MyService())`
- C) `Injector.provide(MyService())`
- D) `ServiceLocator.register(MyService)`

<details>
<summary>Ответ</summary>

**B) `Get.put(MyService())` или `Get.lazyPut(() => MyService())`**

- `Get.put(instance)` — создаёт сразу
- `Get.lazyPut(() => MyService())` — создаёт при первом `Get.find`
- `Get.putAsync(() async => ...)` — асинхронная инициализация
- `Get.find<MyService>()` — получение из DI

</details>

---

### Вопрос 6 🟡

Что такое `GetxController` lifecycle?

- A) Аналог `StatefulWidget` lifecycle
- B) `onInit()` — при первом использовании; `onReady()` — после первого render; `onClose()` — при удалении из DI
- C) `created()`, `mounted()`, `destroyed()`
- D) GetxController не имеет lifecycle

<details>
<summary>Ответ</summary>

**B) `onInit()`, `onReady()`, `onClose()`**

```dart
class MyController extends GetxController {
  @override
  void onInit() {
    super.onInit();
    fetchData(); // вызывается сразу при создании
  }

  @override
  void onReady() {
    super.onReady();
    // вызывается после первого frame — безопасно показывать диалоги
  }

  @override
  void onClose() {
    super.onClose();
    // очистка ресурсов
  }
}
```

</details>

---

### Вопрос 7 🟡

Как показать SnackBar через GetX без BuildContext?

- A) `ScaffoldMessenger.of(context).showSnackBar(...)`
- B) `Get.snackbar('Заголовок', 'Сообщение')`
- C) `GetSnackBar(title: ..., message: ...)`
- D) Нельзя — всегда нужен context

<details>
<summary>Ответ</summary>

**B) `Get.snackbar('Заголовок', 'Сообщение')`**

GetX предоставляет UI без context: `Get.snackbar()`, `Get.dialog()`, `Get.bottomSheet()`. Работает через overlay корневого Navigator. Настройка: `Get.snackbar('Title', 'Msg', snackPosition: SnackPosition.BOTTOM, backgroundColor: Colors.green)`.

</details>

---

### Вопрос 8 🔴

Какие недостатки GetX вызывают споры в Flutter сообществе?

- A) GetX идеален — только преимущества
- B) Скрытые зависимости, глобальный state, tight coupling, сложность тестирования, магические строки в named routes, нарушение принципа единственной ответственности
- C) Медленнее Provider в 10 раз
- D) Не работает на iOS

<details>
<summary>Ответ</summary>

**B) Скрытые зависимости, глобальный state, сложность тестирования**

Критика GetX:

- `Get.find<T>()` — скрытая зависимость, не видна в конструкторе
- Глобальные переменные сложно тестировать и мокировать
- `Get.to()` без контекста нарушает Flutter navigation parad
- Many concerns в одном пакете
- Для небольших приложений удобен, для крупных — сложен в поддержке

</details>

---

### Вопрос 9 🔴

Как реализовать `Workers` в GetX для реакции на изменения state?

- A) `.listen()` на obs переменную
- B) `ever(obs, (val) => ...)`, `once(obs, callback)`, `debounce(obs, callback, time: Duration)`, `interval(obs, callback, time: Duration)`
- C) `GetX.watch(() => obs)` + callback
- D) `StreamWorker(obs.stream)`

<details>
<summary>Ответ</summary>

**B) `ever`, `once`, `debounce`, `interval`**

```dart
class SearchController extends GetxController {
  var query = ''.obs;

  @override
  void onInit() {
    super.onInit();
    debounce(query, (_) => search(query.value), time: 500.ms);
    ever(query, (val) => print('query changed: $val'));
    once(query, (_) => analytics.log('first_search'));
  }
}
```

</details>

---

### Вопрос 10 🔴

Как тестировать GetX контроллеры?

- A) GetX контроллеры нетестируемы
- B) `Get.put(MockService())` до теста → `Get.find<MyController>()` методы тестируются напрямую → `Get.reset()` после теста
- C) Только через `WidgetTest`
- D) Через `GetTest` пакет

<details>
<summary>Ответ</summary>

**B) `Get.put(MockService())` + прямое тестирование контроллера + `Get.reset()`**

```dart
void main() {
  setUp(() {
    Get.put<ApiService>(MockApiService());
  });

  tearDown(() => Get.reset());

  test('fetchUser updates state', () async {
    final controller = Get.put(UserController());
    await controller.fetchUser(id: 1);
    expect(controller.user.value?.name, 'Test User');
  });
}
```

</details>
