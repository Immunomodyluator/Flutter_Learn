# Квиз: Rebuild Optimization

**Тема:** 15.2 — Rebuild Optimization  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Почему `const` конструктор помогает производительности Flutter?

- A) `const` ускоряет Dart VM
- B) `const` виджеты создаются один раз и не перестраиваются при `setState()` родителя, если их параметры не изменились
- C) `const` отключает анимации
- D) `const` только для статического текста

<details>
<summary>Ответ</summary>

**B) `const` виджеты не перестраиваются**

```dart
// БЕЗ const — перестраивается при каждом setState родителя:
Widget build(BuildContext context) {
  return Column(children: [
    Text('Заголовок'),          // перестраивается каждый раз
    Icon(Icons.star),           // перестраивается каждый раз
    DynamicWidget(value: _val), // нужна перестройка
  ]);
}

// С const — Flutter пропускает константные виджеты:
Widget build(BuildContext context) {
  return Column(children: [
    const Text('Заголовок'),    // ✅ пропускается при rebuild
    const Icon(Icons.star),     // ✅ пропускается при rebuild
    DynamicWidget(value: _val), // только этот перестраивается
  ]);
}
```

Flutter сравнивает ссылки: два `const Text('same')` — один объект в памяти.

</details>

---

### Вопрос 2 🟢

Что такое `RepaintBoundary` и когда его использовать?

- A) Граница для перерисовки всего экрана
- B) Виджет, создающий отдельный слой рендеринга — дочерний виджет перерисовывается независимо от родителя
- C) Замена `ClipRect`
- D) `RepaintBoundary` увеличивает производительность всегда

<details>
<summary>Ответ</summary>

**B) Отдельный слой рендеринга для изоляции перерисовок**

```dart
// Анимированный виджет в списке без RepaintBoundary:
// → каждый кадр анимации перерисовывает весь список

// С RepaintBoundary:
ListView.builder(
  itemBuilder: (context, index) => RepaintBoundary(
    child: AnimatedProductCard(product: products[index]),
  ),
)

// Когда ПОЛЕЗЕН:
// ✅ Часто перерисовываемый виджет в стабильном окружении
// ✅ Сложные анимации (анимированный логотип в шапке)
// ✅ Видео/GIF в списке

// Когда ВРЕДЕН (лишние расходы):
// ❌ Простые виджеты без анимаций
// ❌ Вся страница обёрнута в RepaintBoundary
// ❌ Когда родитель и дочерний всегда перерисовываются вместе
```

</details>

---

### Вопрос 3 🟢

Как `Keys` влияют на производительность и перестройку виджетов?

- A) `Keys` только для нахождения виджетов в тестах
- B) `Keys` помогают Flutter сопоставлять элементы при изменении списка — сохраняют состояние и избегают ненужных перестроек
- C) `Keys` всегда ускоряют ListView
- D) `GlobalKey` лучший выбор для производительности

<details>
<summary>Ответ</summary>

**B) Keys сохраняют состояние и помогают при переупорядочивании**

```dart
// БЕЗ ключей — при удалении первого элемента все перестраиваются:
ListView(children: [
  ItemWidget(item: items[0]),  // Flutter не знает что это было items[0]
  ItemWidget(item: items[1]),
]);

// С ключами — Flutter отслеживает конкретный элемент:
ListView(children: items.map((item) =>
  ItemWidget(key: ValueKey(item.id), item: item),
).toList());

// ObjectKey — по ссылке на объект:
ItemWidget(key: ObjectKey(item), item: item)

// UniqueKey — новый ключ при каждом rebuild (принудительно пересоздаёт):
Container(key: UniqueKey())  // принудительный rebuild

// PageStorageKey — сохранение scroll position:
ListView(key: const PageStorageKey('myList'))
```

</details>

---

### Вопрос 4 🟡

Как минимизировать перестройки виджетов при использовании `Provider`?

- A) Всегда использовать `Consumer<T>` на весь экран
- B) `Consumer` / `Selector` оборачивает только тот виджет, которому нужны данные; `context.select()` подписывается на отдельное поле
- C) `Provider.of(context, listen: true)` в build() корня
- D) Использовать `ChangeNotifier` только в корне приложения

<details>
<summary>Ответ</summary>

**B) `Selector` и `context.select()` для точечных подписок**

```dart
// ПЛОХО: весь экран перестраивается при любом изменении:
class ProductScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final vm = context.watch<ProductViewModel>();
    return Scaffold(
      body: ProductList(products: vm.products),
      floatingActionButton: vm.isLoading ? ... : ..., // из-за этого
    );
  }
}

// ХОРОШО: только кнопка реагирует на isLoading:
class ProductScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Consumer<ProductViewModel>(
        builder: (_, vm, __) => ProductList(products: vm.products),
      ),
      floatingActionButton: Selector<ProductViewModel, bool>(
        selector: (_, vm) => vm.isLoading,
        builder: (_, isLoading, __) =>
            isLoading ? CircularProgressIndicator() : AddButton(),
      ),
    );
  }
}
```

</details>

---

### Вопрос 5 🟡

Как использовать `select` vs `watch` в Riverpod для оптимизации?

- A) `select` — для написания кода; `watch` — для производительности
- B) `ref.watch(provider)` — полная подписка; `ref.watch(provider.select((s) => s.field))` — подписка только на конкретное поле, rebuild только при его изменении
- C) `ref.select()` доступен только в Notifier
- D) `select` работает только с `StateProvider`

<details>
<summary>Ответ</summary>

**B) `select` — точечная подписка на часть состояния**

```dart
// watch полного состояния — rebuild при любом изменении:
final user = ref.watch(userProvider);
Text(user.name); // rebuild даже при изменении user.email

// select — только при изменении name:
final userName = ref.watch(userProvider.select((u) => u.name));
Text(userName); // rebuild ТОЛЬКО когда name изменился

// Множественные поля через record:
final (name, email) = ref.watch(
  userProvider.select((u) => (u.name, u.email)),
);

// В Notifier — те же принципы:
class UserNameWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Только когда username изменится:
    final username = ref.watch(
      userProvider.select((state) => state.username),
    );
    return Text(username);
  }
}
```

</details>

---

### Вопрос 6 🟡

Как работает `shouldRebuild` в `Consumer` и когда его переопределять?

- A) `shouldRebuild` — метод `StatefulWidget`
- B) В `Consumer` нет `shouldRebuild`; он есть в кастомном `InheritedWidget` — контролирует зависимых виджетов перестройку
- C) `shouldRebuild` — часть `StreamBuilder`
- D) `shouldRebuild` в `AnimatedBuilder`

<details>
<summary>Ответ</summary>

**B) `shouldRebuild` в кастомном `InheritedWidget`**

```dart
// InheritedWidget с кастомным shouldRebuild:
class AppThemeData extends InheritedWidget {
  final ThemeConfig config;

  const AppThemeData({
    required this.config,
    required super.child,
  });

  // Перестроить зависимых ТОЛЬКО если config изменился:
  @override
  bool updateShouldNotify(AppThemeData oldWidget) =>
      config != oldWidget.config;

  static AppThemeData of(BuildContext context) =>
      context.dependOnInheritedWidgetOfExactType<AppThemeData>()!;
}

// В BLoC — BlocBuilder.buildWhen:
BlocBuilder<CounterBloc, CounterState>(
  buildWhen: (previous, current) =>
      previous.count != current.count, // rebuild только при смене count
  builder: (context, state) => Text('${state.count}'),
)
```

</details>

---

### Вопрос 7 🟡

Как разбить крупный `build()` метод на части без потери производительности?

- A) Извлечь в `_buildHeader()` — приватный метод
- B) Вынести в отдельный `StatelessWidget` / `Widget Function()` — метод не даёт оптимизации, только виджет-класс изолирует rebuild
- C) Использовать `Builder` виджет
- D) Нет разницы между методом и классом

<details>
<summary>Ответ</summary>

**B) Отдельный `StatelessWidget` — правильный выбор**

```dart
// ПЛОХО: приватный метод — часть родительского build():
Widget build(BuildContext context) {
  return Column(children: [
    _buildHeader(),   // при rebuild родителя — перестраивается
    _buildBody(),
    _buildFooter(),
  ]);
}

// ХОРОШО: отдельный виджет с const — пропускается если параметры не менялись:
Widget build(BuildContext context) {
  return Column(children: [
    const PageHeader(title: 'Продукты'),  // const = не перестраивается
    ProductBody(products: _products),
    const PageFooter(),                   // const = не перестраивается
  ]);
}

class PageHeader extends StatelessWidget {
  final String title;
  const PageHeader({required this.title});
  // ...
}
```

Исключение: `Builder` виджет полезен для доступа к `BuildContext` потомка.

</details>

---

### Вопрос 8 🔴

Как оптимизировать `ListView` с большим количеством элементов?

- A) Использовать `SingleChildScrollView` + `Column`
- B) `ListView.builder` с `itemExtent`/`prototypeItem`; `const` элементы; `AutomaticKeepAlive` только когда нужно; кэшировать тяжёлые элементы
- C) Загрузить все данные сразу
- D) `ListView(shrinkWrap: true)` внутри `SingleChildScrollView`

<details>
<summary>Ответ</summary>

**B) `ListView.builder` + `itemExtent` + оптимизации**

```dart
// Хорошая практика:
ListView.builder(
  itemCount: items.length,
  // itemExtent: фиксированная высота — Flutter не вычисляет layout:
  itemExtent: 72.0,
  // или prototypeItem: один пример для вычисления высоты:
  // prototypeItem: ProductTile(product: items.first),

  // cacheExtent: сколько пикселей за пределами экрана кэшировать:
  cacheExtent: 500,

  itemBuilder: (context, index) => ProductTile(
    key: ValueKey(items[index].id), // ключи для правильного сопоставления
    product: items[index],
  ),
)

// ИЗБЕГАТЬ:
// ❌ Column + ListView (двойной scroll)
// ❌ shrinkWrap: true в больших списках (рассчитывает всю высоту)
// ❌ AutomaticKeepAlive везде (держит все элементы в памяти)
// ❌ Строить виджеты в itemBuilder вместо кэширования
```

</details>

---

### Вопрос 9 🔴

Что такое Jank и как его диагностировать?

- A) Jank — это ошибка сборки Flutter
- B) Jank — видимые задержки/рывки анимации из-за пропущенных фреймов (> 16ms/frame); диагностика: Performance overlay, Timeline, `flutter run --profile`
- C) Jank только на Android устройствах
- D) Jank — медленная загрузка данных из сети

<details>
<summary>Ответ</summary>

**B) Пропущенные фреймы → видимые рывки**

```dart
// Причины Jank и решения:

// 1. Тяжёлый build():
// БАД: синхронное чтение файла в build()
// ХОРОШО: FutureBuilder / загружать данные вне build()

// 2. Лишние rebuild:
// БАД: context.watch() в корне
// ХОРОШО: Selector / select() / Consumer в листьях

// 3. Большие изображения без кэша:
// БАД: Image.network каждый раз
// ХОРОШО: cached_network_image

// 4. Shader compilation jank (первый рендер):
// Решение: --bundle-sksl-path + warmup
flutter run --cache-sksl --purge-persistent-cache
flutter run --bundle-sksl-path flutter_01.sksl.json

// 5. GC jank:
// Решение: object pooling, избегать аллокаций в build()

// Измерение:
flutter run --profile --trace-skia
// в DevTools: количество фреймов > 16ms в Performance
```

</details>

---

### Вопрос 10 🔴

Как использовать `AutomaticKeepAliveClientMixin` правильно?

- A) Всегда добавлять ко всем элементам `PageView`
- B) Добавлять только когда нужно сохранить состояние (scroll position, form data) при переключении вкладок; иначе — утечка памяти
- C) `AutomaticKeepAlive` = кэш изображений
- D) `AutomaticKeepAliveClientMixin` только для `ListView`

<details>
<summary>Ответ</summary>

**B) Только когда нужно сохранить состояние — иначе утечка памяти**

```dart
// Правильное использование — например вкладка с формой:
class FormTab extends StatefulWidget {
  const FormTab();
  @override State<FormTab> createState() => _FormTabState();
}

class _FormTabState extends State<FormTab>
    with AutomaticKeepAliveClientMixin {

  // ОБЯЗАТЕЛЬНО переопределить:
  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context); // ОБЯЗАТЕЛЬНО вызвать!
    return const FormWidget();
  }
}

// Когда НЕ нужен KeepAlive:
// - Простые списки без состояния
// - Данные берутся из провайдера (не из local state)
// - Пользователь редко возвращается на вкладку
// → Flutter уничтожит и пересоздаст виджет (нормально!)

// PageView с keepPage: true (дефолт) сохраняет _scroll_
// но не сохраняет widget state без KeepAlive
```

</details>
