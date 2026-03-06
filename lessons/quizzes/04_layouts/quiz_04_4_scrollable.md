# Квиз: Scrollable виджеты

**Тема:** 04.4 — ListView, GridView, Slivers  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой конструктор `ListView` нужно использовать для длинных динамических списков?

- A) `ListView(children: [...])`
- B) `ListView.builder(itemCount: n, itemBuilder: (ctx, i) => ...)`
- C) `ListView.from(items)`
- D) `ListView.dynamic`

<details>
<summary>Ответ</summary>

**B) `ListView.builder(itemCount: n, itemBuilder: (ctx, i) => ...)`**

`ListView(children: [...])` создаёт **все** виджеты сразу — не подходит для длинных списков. `ListView.builder` создаёт виджеты лениво, только для видимых элементов. Это критично для производительности при большом числе элементов.

</details>

---

### Вопрос 2 🟢

Что делает `GridView.count`?

- A) Создаёт сетку с фиксированным числом строк
- B) Создаёт сетку с фиксированным числом колонок (`crossAxisCount`)
- C) Создаёт сетку с элементами фиксированного размера
- D) Считает количество виджетов

<details>
<summary>Ответ</summary>

**B) Создаёт сетку с фиксированным числом колонок (`crossAxisCount`)**

`GridView.count(crossAxisCount: 3)` — всегда 3 колонки. Альтернатива: `GridView.extent(maxCrossAxisExtent: 150)` — максимальная ширина ячейки 150px, Flutter сам вычислит количество колонок. `GridView.builder` — ленивая версия.

</details>

---

### Вопрос 3 🟢

Для чего используется `physics: NeverScrollableScrollPhysics()`?

- A) Отключает bounce-эффект на iOS
- B) Запрещает скроллинг виджета — используется при вложении в другой скроллируемый виджет
- C) Устанавливает физику iOS для Android
- D) Ускоряет скроллинг

<details>
<summary>Ответ</summary>

**B) Запрещает скроллинг виджета — используется при вложении в другой скроллируемый виджет**

При `ListView` внутри `SingleChildScrollView` или другого `ListView` нужно отключить скроллинг вложенного: `ListView(shrinkWrap: true, physics: NeverScrollableScrollPhysics(), ...)`. Иначе два scroll-контроллера конфликтуют.

</details>

---

### Вопрос 4 🟡

Что такое `Sliver` в Flutter?

- A) Виджет для анимированных переходов
- B) Часть скроллируемой области, которая знает свою позицию и может менять размер/поведение при скроллинге
- C) Способ разбить `ListView` на страницы
- D) Альтернативное название для `Expanded`

<details>
<summary>Ответ</summary>

**B) Часть скроллируемой области, которая знает свою позицию и может менять размер/поведение при скроллинге**

Slivers — строительные блоки `CustomScrollView`. `SliverAppBar`, `SliverList`, `SliverGrid`, `SliverPadding`. Позволяют создавать сложные scroll-эффекты: AppBar который сворачивается, sticky headers, разные типы контента в одном прокрутке.

</details>

---

### Вопрос 5 🟡

Чем `SliverAppBar` отличается от обычного `AppBar`?

- A) Они идентичны
- B) `SliverAppBar` может растягиваться, сворачиваться и закрепляться при скроллинге (`expandedHeight`, `floating`, `pinned`, `snap`)
- C) `SliverAppBar` поддерживает только текст
- D) `SliverAppBar` не поддерживает `leading`

<details>
<summary>Ответ</summary>

**B) `SliverAppBar` может растягиваться, сворачиваться и закрепляться при скроллинге**

```dart
SliverAppBar(
  expandedHeight: 200,
  pinned: true,     // остаётся видимым при скролле вверх
  floating: true,   // появляется при малейшем скролле вниз
  flexibleSpace: FlexibleSpaceBar(background: Image.asset(...)),
)
```

</details>

---

### Вопрос 6 🟡

Как добавить разделители между элементами `ListView`?

- A) `ListView.builder` + вручную `if (i < last) Divider()`
- B) `ListView.separated(separatorBuilder: (ctx, i) => Divider())`
- C) `ListTile(divider: true)`
- D) Параметр `showDividers: true`

<details>
<summary>Ответ</summary>

**B) `ListView.separated(separatorBuilder: (ctx, i) => Divider())`**

`ListView.separated` принимает два builder-а: `itemBuilder` и `separatorBuilder`. Разделитель рендерится между каждой парой элементов. `separatorBuilder` получает индекс элемента _перед_ разделителем.

</details>

---

### Вопрос 7 🟡

Что такое `CustomScrollView` и когда его использовать?

- A) `ListView` с кастомной физикой
- B) Scroll-контейнер, принимающий `slivers: []` — позволяет смешивать SliverList/SliverGrid/SliverAppBar в одной прокрутке
- C) Виджет для горизонтального скроллинга
- D) `ListView` с кастомной скоростью

<details>
<summary>Ответ</summary>

**B) Scroll-контейнер, принимающий `slivers: []`**

```dart
CustomScrollView(slivers: [
  SliverAppBar(...),
  SliverToBoxAdapter(child: Header()),
  SliverList(delegate: SliverChildBuilderDelegate(...)),
  SliverGrid(...),
])
```

Весь контент прокручивается со связной физикой. `SliverToBoxAdapter` — оборачивает обычный виджет в sliver.

</details>

---

### Вопрос 8 🔴

Как реализовать `sticky headers` (заголовки, остающиеся видимыми при скроллинге секции) во Flutter?

- A) `ListView` + `Positioned`
- B) `SliverPersistentHeader` с `pinned: true` в `CustomScrollView`
- C) Встроенного решения нет — только через `Stack`
- D) `SliverAppBar` с `floating: false`

<details>
<summary>Ответ</summary>

**B) `SliverPersistentHeader` с `pinned: true` в `CustomScrollView`**

`SliverPersistentHeader(pinned: true, delegate: MyHeaderDelegate())` — заголовок остаётся на экране пока скроллится секция. `SliverPersistentHeaderDelegate` требует реализации `build`, `maxExtent`, `minExtent`, `shouldRebuild`. Пакеты `sliver_tools`, `sticky_headers` упрощают это.

</details>

---

### Вопрос 9 🔴

Что такое `ScrollController` и как его использовать для программной прокрутки?

- A) Класс для кастомной физики скроллинга
- B) `ChangeNotifier`-подобный объект, прикрепляемый к scroll виджету для чтения позиции и программного управления прокруткой
- C) Хук для `ListView.builder`
- D) Обёртка вокруг `PageController`

<details>
<summary>Ответ</summary>

**B) `ChangeNotifier`-подобный объект для чтения позиции и программного управления прокруткой**

```dart
final _controller = ScrollController();

// Прокрутить в начало:
_controller.animateTo(0, duration: Duration(ms: 300), curve: Curves.easeOut);

// Слушать позицию:
_controller.addListener(() => print(_controller.offset));

// Присвоить:
ListView.builder(controller: _controller, ...)
```

Нужно вызывать `_controller.dispose()` в `dispose()`.

</details>

---

### Вопрос 10 🔴

Что такое `keepAlive` в scroll виджетах и как его реализовать?

- A) Сохраняет виджет в памяти после dispose
- B) `AutomaticKeepAliveClientMixin` — сохраняет state элементов списка при выходе из viewport
- C) Параметр `ListView(keepAlive: true)`
- D) `PreservedState` аннотация

<details>
<summary>Ответ</summary>

**B) `AutomaticKeepAliveClientMixin` — сохраняет state элементов списка при выходе из viewport**

```dart
class _MyItemState extends State<MyItem>
    with AutomaticKeepAliveClientMixin {
  @override
  bool get wantKeepAlive => true;

  @override
  Widget build(BuildContext context) {
    super.build(context); // обязательно!
    return ...;
  }
}
```

Без этого `ListView.builder` уничтожает и пересоздаёт элементы при прокрутке. Полезно для элементов с формами, видео, анимациями.

</details>
