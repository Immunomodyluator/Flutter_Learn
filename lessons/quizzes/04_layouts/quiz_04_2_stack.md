# Квиз: Stack и Positioned

**Тема:** 04.2 — Stack  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как `Stack` располагает своих детей?

- A) По вертикали, как Column
- B) В виде сетки
- C) Слоями — каждый следующий дочерний виджет рисуется поверх предыдущего
- D) По горизонтали, как Row

<details>
<summary>Ответ</summary>

**C) Слоями — каждый следующий дочерний виджет рисуется поверх предыдущего**

`Stack` накладывает детей друг на друга. Последний в списке `children` находится на переднем плане. Позиционирование: без `Positioned` — по `alignment`, с `Positioned` — по абсолютным координатам.

</details>

---

### Вопрос 2 🟢

Для чего используется виджет `Positioned` внутри `Stack`?

- A) Устанавливает z-index элемента
- B) Задаёт положение дочернего виджета относительно границ `Stack` через top/bottom/left/right
- C) Анимирует перемещение виджета
- D) Фиксирует виджет при скроллинге `Stack`

<details>
<summary>Ответ</summary>

**B) Задаёт положение дочернего виджета относительно границ Stack через top/bottom/left/right**

`Positioned(top: 10, right: 20, child: ...)` размещает виджет в 10px от верха и 20px от правого края `Stack`. Можно указывать `width`/`height` напрямую в `Positioned`.

</details>

---

### Вопрос 3 🟢

Каков `alignment` по умолчанию у не-`Positioned` детей `Stack`?

- A) `Alignment.center`
- B) `Alignment.bottomRight`
- C) `Alignment.topLeft`
- D) `Alignment.topCenter`

<details>
<summary>Ответ</summary>

**C) `Alignment.topLeft`**

По умолчанию `Stack(alignment: AlignmentDirectional.topStart)` — т.е. верхний-левый угол. Чтобы центрировать не-`Positioned` детей: `Stack(alignment: Alignment.center, ...)`.

</details>

---

### Вопрос 4 🟡

Что означает `Stack(fit: StackFit.expand)`?

- A) `Stack` расширяется в зависимости от размеров детей
- B) Все не-`Positioned` дети принудительно занимают весь размер `Stack`
- C) Добавляет скроллинг при переполнении
- D) Растягивает только первого ребёнка

<details>
<summary>Ответ</summary>

**B) Все не-`Positioned` дети принудительно занимают весь размер Stack**

`StackFit.loose` (по умолчанию) — дети могут быть меньше `Stack`. `StackFit.expand` — не-`Positioned` дети получают tight-constraints и занимают весь `Stack`. Полезно для фоновых изображений: `Stack(fit: StackFit.expand, children: [Image.asset(...), overlay])`.

</details>

---

### Вопрос 5 🟡

Чем `Positioned.fill` отличается от `Positioned(top: 0, left: 0, right: 0, bottom: 0)`?

- A) Они идентичны
- B) `Positioned.fill` дополнительно центрирует содержимое
- C) `Positioned.fill` — именованный конструктор-сокращение для top/left/right/bottom = 0
- D) `Positioned.fill` работает только с `Container`

<details>
<summary>Ответ</summary>

**C) `Positioned.fill` — именованный конструктор-сокращение для top/left/right/bottom = 0**

Разница лишь в синтаксисе. Оба заставляют дочерний виджет занять всё пространство `Stack`. Также есть `Positioned.fromRect` и `Positioned.fromRelativeRect` для продвинутого позиционирования.

</details>

---

### Вопрос 6 🟡

Что такое `IndexedStack`?

- A) Stack с нумерованными слоями
- B) Stack, который показывает только одного ребёнка по индексу, но сохраняет состояние всех
- C) Stack с поддержкой скроллинга
- D) Stack с анимацией переходов

<details>
<summary>Ответ</summary>

**B) Stack, который показывает только одного ребёнка по индексу, но сохраняет состояние всех**

`IndexedStack(index: _currentIndex, children: [...])` — находит применение как альтернатива `PageView` когда нужно сохранять состояния всех страниц BottomNavigationBar. Все виджеты построены, но видим только `children[index]`.

</details>

---

### Вопрос 7 🟡

Как изменить порядок наложения (z-index) элементов в `Stack`?

- A) Через параметр `zIndex` в `Positioned`
- B) Порядок слоёв определяется порядком в списке `children` — последний сверху
- C) Через `Overlay` виджет
- D) Через `elevation` параметр

<details>
<summary>Ответ</summary>

**B) Порядок слоёв определяется порядком в списке children — последний сверху**

Чтобы поднять элемент на передний план — переместить его в конец списка. В Flutter нет свойства `zIndex` как в CSS. Для динамического управления слоями можно использовать `Overlay` из `Navigator`.

</details>

---

### Вопрос 8 🔴

Какой будет размер `Stack`, если все его дети — `Positioned`?

- A) `Stack` принимает размер экрана
- B) Размер smallest child
- C) Размер largest child
- D) `Stack` займёт всё доступное пространство от родителя (принимает loose constraints)

<details>
<summary>Ответ</summary>

**D) Stack займёт всё доступное пространство от родителя (принимает loose constraints)**

Если все дети `Positioned` — `Stack` не может вычислить размер из них и занимает максимально доступный. Если есть хотя бы один не-`Positioned` ребёнок — `Stack` подстраивается под его размер (при `StackFit.loose`).

</details>

---

### Вопрос 9 🔴

Что произойдёт, если `Positioned` виджет выйдет за пределы `Stack`?

- A) Автоматически обрезается
- B) Выбрасывается исключение
- C) По умолчанию контент рисуется за пределами — управляется `clipBehavior: Clip.hardEdge`
- D) Виджет сжимается до границ Stack

<details>
<summary>Ответ</summary>

**C) По умолчанию контент рисуется за пределами — управляется `clipBehavior: Clip.hardEdge`**

`Stack(clipBehavior: Clip.none)` — явно разрешает выход за границы, полезно для декоративных элементов. `Clip.hardEdge` — обрезает пиксельно, `Clip.antiAlias` — с антиалиасингом, `Clip.antiAliasWithSaveLayer` — наиболее качественно, но медленнее.

</details>

---

### Вопрос 10 🔴

Как реализовать `AnimatedPositioned` для анимации перемещения виджета в `Stack`?

- A) Использовать `Tween` + `AnimationController`
- B) `AnimatedPositioned` — implicit animation виджет, принимает те же параметры что `Positioned` + `duration`
- C) Обернуть `Positioned` в `AnimatedContainer`
- D) Использовать `Hero` анимацию

<details>
<summary>Ответ</summary>

**B) `AnimatedPositioned` — implicit animation виджет, принимает те же параметры что `Positioned` + `duration`**

```dart
AnimatedPositioned(
  duration: Duration(milliseconds: 300),
  top: _isOpen ? 100.0 : 0.0,
  left: 20,
  child: MyCard(),
)
```

При изменении `top`/`left`/`right`/`bottom` Flutter автоматически анимирует переход. Это implicit animation — не требует `AnimationController`.

</details>
