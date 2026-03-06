# Квиз: StatefulWidget

**Тема:** 03.2 — StatefulWidget  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какой метод вызывается один раз при создании `State` объекта?

- A) `build()`
- B) `initState()`
- C) `create()`
- D) `onInit()`

<details>
<summary>Ответ</summary>

**B) `initState()`**

`initState()` вызывается ровно один раз — сразу после создания `State`. Используется для: подписки на Stream, инициализации AnimationController, загрузки начальных данных. Обязательно вызывать `super.initState()` первым.

</details>

---

### Вопрос 2 🟢

Что произойдёт, если вызвать `setState()` после удаления виджета из дерева?

- A) Исключение сразу
- B) Тихое игнорирование
- C) `FlutterError` или exception в debug-режиме
- D) Состояние обновится при следующем появлении

<details>
<summary>Ответ</summary>

**C) `FlutterError` или exception в debug-режиме**

Вызов `setState()` на unmounted State (`!mounted`) — ошибка. В debug режиме Flutter выбросит ошибку. Правило: всегда проверять `if (mounted) setState(...)` при вызове после async операций.

</details>

---

### Вопрос 3 🟢

В каком методе нужно освобождать ресурсы (`AnimationController.dispose()`, `cancel()` подписок)?

- A) `initState()`
- B) `build()`
- C) `dispose()`
- D) `deactivate()`

<details>
<summary>Ответ</summary>

**C) `dispose()`**

`dispose()` вызывается перед удалением `State` из дерева. Здесь нужно освобождать: `AnimationController`, `TextEditingController`, `FocusNode`, `StreamSubscription`. `super.dispose()` — обязательно в конце.

</details>

---

### Вопрос 4 🟡

Что обязательно должен содержать коллбэк `setState()`?

- A) Только `return` значение
- B) Синхронное изменение состояния (поля State)
- C) Вызов `build()`
- D) `async` операции для обновления данных

<details>
<summary>Ответ</summary>

**B) Синхронное изменение состояния (поля State)**

`setState(() { counter++; })` — изменяет поле и помечает виджет dirty для перестройки. Внутри `setState` нельзя `await`. Async операции должны быть завершены до `setState`: `final data = await fetch(); setState(() { _data = data; });`.

</details>

---

### Вопрос 5 🟡

Какой жизненный цикл State при hot reload?

- A) `dispose()` → `initState()` → `build()`
- B) `reassemble()` → `build()`
- C) `build()` сразу
- D) `setState()` автоматически

<details>
<summary>Ответ</summary>

**B) `reassemble()` → `build()`**

При Hot Reload Flutter вызывает `reassemble()` на всех `State` объектах (сверху вниз по дереву), затем `build()`. `initState()` не вызывается повторно — состояние сохраняется. `reassemble()` можно переопределить для специальной логики.

</details>

---

### Вопрос 6 🟡

Зачем разделять `StatefulWidget` и `State` на два класса?

- A) Ограничение языка Dart
- B) Widget (иммутабельная конфигурация) отдельно от State (изменяемого состояния); Flutter может переиспользовать State
- C) Для поддержки Hot Reload
- D) State должен быть приватным классом

<details>
<summary>Ответ</summary>

**B) Widget (иммутабельная конфигурация) отдельно от State (изменяемого состояния); Flutter может переиспользовать State**

При перемещении виджета в дереве (с сохранением типа и ключа) Flutter переиспользует `State` объект, присваивая ему новый `widget`. Разделение позволяет виджету быть иммутабельным (re-created freely) при сохранении изменяемого `State`.

</details>

---

### Вопрос 7 🟡

Когда вызывается `didUpdateWidget(oldWidget)`?

- A) Каждый раз при вызове `setState()`
- B) Когда родитель передаёт новый Widget того же типа с новыми параметрами
- C) При Hot Reload
- D) При переходе между экранами

<details>
<summary>Ответ</summary>

**B) Когда родитель передаёт новый Widget того же типа с новыми параметрами**

`didUpdateWidget` позволяет реагировать на изменение пропов: перезапустить анимацию, обновить подписки. `oldWidget` — предыдущая конфигурация. Если `widget.stream != oldWidget.stream` — нужно переподписаться.

</details>

---

### Вопрос 8 🔴

В чём разница между `deactivate()` и `dispose()`?

- A) Они одинаковы
- B) `deactivate()` — временное удаление из дерева (GlobalKey перемещение); `dispose()` — постоянное удаление
- C) `deactivate()` вызывается только при навигации назад
- D) `dispose()` вызывается до `deactivate()`

<details>
<summary>Ответ</summary>

**B) `deactivate()` — временное удаление из дерева (GlobalKey перемещение); `dispose()` — постоянное удаление**

`deactivate()` вызывается при удалении State из дерева, но он может быть переинсертирован (например при использовании `GlobalKey` для перемещения). Если переинсертирование не произошло — вызывается `dispose()`. Большинству виджетов нужен только `dispose()`.

</details>

---

### Вопрос 9 🔴

Как корректно обновить состояние после async операции?

```dart
Future<void> loadData() async {
  final data = await api.fetchData();
  // как обновить _data?
}
```

- A) `_data = data;` напрямую
- B) `setState(() { _data = data; });`
- C) `if (mounted) setState(() { _data = data; });`
- D) `Future.microtask(() { _data = data; });`

<details>
<summary>Ответ</summary>

**C) `if (mounted) setState(() { _data = data; });`**

После `await` виджет мог быть удалён из дерева. Вариант A без `setState` не вызовет rebuild. Вариант B без `mounted` вызовет ошибку если виджет уже удалён. Правильно: проверить `mounted` перед `setState`.

</details>

---

### Вопрос 10 🔴

Что такое `widget` в контексте `State`?

- A) Родительский виджет
- B) Ссылка на текущий `StatefulWidget` экземпляр, связанный с этим State
- C) Ссылка на корневой виджет приложения
- D) Контекст текущего виджета

<details>
<summary>Ответ</summary>

**B) Ссылка на текущий `StatefulWidget` экземпляр, связанный с этим State**

`widget` — геттер в `State<T>` возвращающий текущий `T` объект. При `didUpdateWidget` — уже содержит **новый** виджет (старый передаётся параметром `oldWidget`). Используется для доступа к пропам: `widget.title`, `widget.onPressed`.

</details>
