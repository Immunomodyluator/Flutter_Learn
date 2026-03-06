# Квиз: Material Widgets

**Тема:** 03.3 — Material Widgets  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой виджет служит корневым для Material Design приложения?

- A) `MaterialWidget`
- B) `MaterialApp`
- C) `Scaffold`
- D) `Material`

<details>
<summary>Ответ</summary>

**B) `MaterialApp`**

`MaterialApp` — корневой виджет: задаёт тему, локализацию, навигацию, title. `Scaffold` — структура отдельного экрана (AppBar, Body, FAB, Drawer). `Material` — примитивный виджет с elevation и ink ripple.

</details>

---

### Вопрос 2 🟢

Что такое `Scaffold`?

- A) Инструмент для автогенерации экранов
- B) Базовая структура экрана: AppBar, Body, BottomBar, Drawer, FAB
- C) Виджет только для форм
- D) Контейнер с тенью

<details>
<summary>Ответ</summary>

**B) Базовая структура экрана: AppBar, Body, BottomBar, Drawer, FAB**

`Scaffold` реализует базовый Material layout: `appBar`, `body`, `floatingActionButton`, `bottomNavigationBar`, `drawer`, `endDrawer`, `snackBar`. Обязателен для использования `ScaffoldMessenger.of(context).showSnackBar()`.

</details>

---

### Вопрос 3 🟢

Как показать `SnackBar`?

- A) `SnackBar.show(context, message)`
- B) `ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('msg')))`
- C) `Scaffold.of(context).showSnackBar(...)`
- D) `showSnackBar(context, 'msg')`

<details>
<summary>Ответ</summary>

**B) `ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('msg')))`**

С Flutter 2.0 `SnackBar` показывается через `ScaffoldMessenger` (не через `Scaffold.of`). `ScaffoldMessenger` живёт выше в дереве и не зависит от конкретного `Scaffold`, позволяя показывать SnackBar при навигации между экранами.

</details>

---

### Вопрос 4 🟡

Чем `ElevatedButton` отличается от `TextButton`?

- A) `TextButton` устарел
- B) `ElevatedButton` имеет elevation (тень), `TextButton` — плоский без фона
- C) `TextButton` принимает только `Text` виджет
- D) `ElevatedButton` нельзя использовать в `AppBar`

<details>
<summary>Ответ</summary>

**B) `ElevatedButton` имеет elevation (тень), `TextButton` — плоский без фона**

Material 3 предлагает: `ElevatedButton` (raised, для главных действий), `FilledButton` (заливка цветом), `OutlinedButton` (граница), `TextButton` (плоский, для вторичных действий). Каждый имеет свою визуальную иерархию.

</details>

---

### Вопрос 5 🟡

Что такое `InkWell` и для чего используется?

- A) Виджет для работы с водяными знаками
- B) Добавляет Material ink ripple эффект на нажатие к любому виджету
- C) Отображает подчёркнутый текст
- D) Создаёт анимированный контейнер

<details>
<summary>Ответ</summary>

**B) Добавляет Material ink ripple эффект на нажатие к любому виджету**

`InkWell` — `GestureDetector` с Material ripple. Нужен `Material` предок для корректного ripple рендеринга. `Ink` + `InkWell` — для кастомных виджетов с ripple. `InkResponse` — более низкоуровневый вариант с настройкой геометрии волны.

</details>

---

### Вопрос 6 🟡

Как показать `Dialog` в Flutter?

- A) `Dialog.show(context)`
- B) `showDialog(context: context, builder: (ctx) => AlertDialog(...))`
- C) `Navigator.pushDialog(context, ...)`
- D) `Overlay.of(context).insert(Dialog())`

<details>
<summary>Ответ</summary>

**B) `showDialog(context: context, builder: (ctx) => AlertDialog(...))`**

`showDialog` возвращает `Future<T?>` — результат, который можно `await`. `AlertDialog` — стандартный диалог с title, content, actions. `SimpleDialog` — список вариантов. `Dialog` — кастомный контент.

</details>

---

### Вопрос 7 🟡

Для чего нужен `AppBar.actions`?

- A) Список роутов приложения
- B) Иконки/кнопки справа в AppBar
- C) Анимации перехода
- D) Вкладки для TabBar

<details>
<summary>Ответ</summary>

**B) Иконки/кнопки справа в AppBar**

`actions: [IconButton(...), PopupMenuButton(...)]` — виджеты справа. `leading` — слева (обычно «назад» или «меню»). `title` — центр. `bottom` — для `TabBar`. `flexibleSpace` — для `SliverAppBar` с коллапсом.

</details>

---

### Вопрос 8 🔴

В чём разница между `BottomNavigationBar` и `NavigationBar` (Material 3)?

- A) Они идентичны
- B) `NavigationBar` — Material 3 компонент с pill-индикатором и анимацией; `BottomNavigationBar` — Material 2 стиль
- C) `NavigationBar` поддерживает только 3 вкладки
- D) `BottomNavigationBar` поддерживает Material 3 темы

<details>
<summary>Ответ</summary>

**B) `NavigationBar` — Material 3 компонент с pill-индикатором и анимацией; `BottomNavigationBar` — Material 2 стиль**

`NavigationBar` из Material 3: индикатор активной вкладки в виде pill-контейнера с `NavigationDestination`. `BottomNavigationBar` — старый стиль. При включённом `useMaterial3: true` предпочтительнее использовать `NavigationBar`.

</details>

---

### Вопрос 9 🔴

Что делает `Divider(indent: 16, endIndent: 16)`?

- A) Добавляет разделитель с отступами слева (16px) и справа (16px)
- B) Устанавливает толщину линии
- C) Создаёт вертикальный разделитель
- D) Добавляет padding к следующему элементу

<details>
<summary>Ответ</summary>

**A) Добавляет разделитель с отступами слева (16px) и справа (16px)**

`indent` — отступ от левого края, `endIndent` — от правого. `thickness` — толщина линии. `color` — цвет. `VerticalDivider` — вертикальный вариант. Используется в `ListView`, `Column` для визуального разделения элементов.

</details>

---

### Вопрос 10 🔴

Что такое `Theme.of(context).copyWith()` и когда применяется?

- A) Создаёт тему только для текущего виджета
- B) Создаёт локальную тему — переопределяет параметры только для поддерева через `Theme(data: ..., child: ...)`
- C) Обновляет глобальную тему приложения
- D) Копирует тему в файл конфигурации

<details>
<summary>Ответ</summary>

**B) Создаёт локальную тему — переопределяет параметры только для поддерева через `Theme(data: ..., child: ...)`**

```dart
Theme(
  data: Theme.of(context).copyWith(
    colorScheme: ColorScheme.fromSeed(seedColor: Colors.red),
  ),
  child: ElevatedButton(...), // кнопка с красной темой, остальное приложение не меняется
)
```

`copyWith` — иммутабельное копирование с заменой конкретных полей.

</details>
