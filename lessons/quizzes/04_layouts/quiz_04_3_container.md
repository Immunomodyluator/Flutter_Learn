# Квиз: Container и Box Model

**Тема:** 04.3 — Container  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что делает `Container` без явно указанных параметров?

- A) Занимает весь экран
- B) Не отображается вообще
- C) Если нет ребёнка — занимает всё доступное пространство; если есть — подстраивается под размер ребёнка
- D) Всегда 0×0 пикселей

<details>
<summary>Ответ</summary>

**C) Если нет ребёнка — занимает всё доступное пространство; если есть — подстраивается под размер ребёнка**

Это распространённый gotcha: `Container()` без child заполняет всё. `Container(child: Text('hi'))` — wrap вокруг текста. Добавьте `color` и посмотрите сами.

</details>

---

### Вопрос 2 🟢

Что такое `BoxDecoration`?

- A) Обёртка для виджета с тенью
- B) Класс для описания визуального оформления: цвет, границы, тени, градиент, скругление углов
- C) Параметр `Container` для добавления padding
- D) Виджет для рисования геометрических фигур

<details>
<summary>Ответ</summary>

**B) Класс для описания визуального оформления: цвет, границы, тени, градиент, скругление углов**

`BoxDecoration(color: ..., borderRadius: ..., boxShadow: [...], gradient: LinearGradient(...), border: Border.all(...))` — передаётся в `Container(decoration: BoxDecoration(...))`. При использовании `decoration` нельзя одновременно указывать `color` в `Container`.

</details>

---

### Вопрос 3 🟢

В чём разница между `padding` и `margin` в `Container`?

- A) Это одно и то же
- B) `padding` — расстояние от границы контейнера **внутрь** (до содержимого); `margin` — расстояние **снаружи** контейнера
- C) `margin` отвечает за внутренние отступы
- D) `padding` работает только с `BoxDecoration`

<details>
<summary>Ответ</summary>

**B) `padding` — расстояние от границы контейнера внутрь (до содержимого); `margin` — расстояние снаружи контейнера**

CSS box model применим здесь напрямую. `Container(margin: EdgeInsets.all(8))` — то же, что обернуть `Container` в `Padding(padding: EdgeInsets.all(8))`. Внутренний `padding` добавляет пространство между границей и `child`.

</details>

---

### Вопрос 4 🟡

Что вызовет ошибку?

```dart
Container(
  color: Colors.blue,
  decoration: BoxDecoration(borderRadius: BorderRadius.circular(10)),
  child: Text('hello'),
)
```

- A) Нельзя использовать `decoration` с `child`
- B) `color` и `decoration` нельзя указывать одновременно — нужно перенести `color` в `BoxDecoration`
- C) `BorderRadius` работает только с `ClipRRect`
- D) `Text` не совместим с `BoxDecoration`

<details>
<summary>Ответ</summary>

**B) `color` и `decoration` нельзя указывать одновременно — нужно перенести `color` в `BoxDecoration`**

Flutter выбросит assertion error: `Cannot provide both a color and a decoration`. Правильно: `decoration: BoxDecoration(color: Colors.blue, borderRadius: BorderRadius.circular(10))`.

</details>

---

### Вопрос 5 🟡

Что делает `constraints: BoxConstraints.tight(Size(100, 100))`?

- A) Устанавливает минимальный размер 100×100
- B) Задаёт максимальный размер 100×100
- C) Фиксирует размер точно в 100×100 (min == max)
- D) Ограничивает только ширину

<details>
<summary>Ответ</summary>

**C) Фиксирует размер точно в 100×100 (min == max)**

`BoxConstraints.tight(size)` создаёт tight constraints, где minWidth == maxWidth и minHeight == maxHeight. `BoxConstraints.loose(size)` — min = 0, max = size (разрешает быть меньше). `BoxConstraints.expand()` — fills container.

</details>

---

### Вопрос 6 🟡

Как применить `transform` к `Container` для поворота на 45 градусов?

- A) `rotation: 45`
- B) `transform: Matrix4.rotationZ(pi / 4)`
- C) Обернуть в `RotatedBox(quarterTurns: 1)`
- D) `decoration: BoxDecoration(rotate: 45)`

<details>
<summary>Ответ</summary>

**B) `transform: Matrix4.rotationZ(pi / 4)`**

`Container(transform: Matrix4.rotationZ(0.785))` — pi/4 радиан = 45°. `transformAlignment` контролирует точку трансформации. Для простых поворотов на четверть оборота `RotatedBox(quarterTurns: 1)` удобнее, но меняет layout, тогда как `transform` — только visual.

</details>

---

### Вопрос 7 🟡

Что такое `DecoratedBox` и когда его использовать вместо `Container`?

- A) Это то же самое, что `Container`
- B) `DecoratedBox` — легковесный виджет только для декорирования, без layout свойств (margin/padding/constraints)
- C) `DecoratedBox` поддерживает анимацию
- D) `DecoratedBox` работает только с цветом фона

<details>
<summary>Ответ</summary>

**B) `DecoratedBox` — легковесный виджет только для декорирования, без layout свойств**

`Container` — удобная обёртка над несколькими виджетами (`Padding`, `DecoratedBox`, `ConstrainedBox`, `Transform`). Если нужна только декорация — `DecoratedBox` чище и эффективнее. Flutter lint рекомендует избегать `Container` без необходимости.

</details>

---

### Вопрос 8 🔴

Как создать градиентный фон с плавным переходом от синего к прозрачному?

- A) `Container(color: Colors.blue.withOpacity(0.5))`
- B) `Container(decoration: BoxDecoration(gradient: LinearGradient(colors: [Colors.blue, Colors.transparent])))`
- C) `Container(decoration: BoxDecoration(gradient: Gradient.linear(...)))`
- D) `GradientContainer(begin: Colors.blue, end: Colors.transparent)`

<details>
<summary>Ответ</summary>

**B) `Container(decoration: BoxDecoration(gradient: LinearGradient(colors: [Colors.blue, Colors.transparent])))`**

`LinearGradient(begin: Alignment.topLeft, end: Alignment.bottomRight, colors: [...], stops: [0.0, 1.0])`. Также доступны `RadialGradient` и `SweepGradient`. `stops` — позиции цветов (0.0–1.0), по умолчанию равномерно.

</details>

---

### Вопрос 9 🔴

Что происходит при вложенных `BoxConstraints` — родитель задаёт max 200, дочерний запрашивает 300?

- A) Дочерний виджет обрезается до 200
- B) Дочерний виджет рисует 300, но overflow предупреждение
- C) Flutter выбрасывает ошибку
- D) Дочерний `Container` ограничивается constraints родителя — займёт 200, игнорируя собственные 300

<details>
<summary>Ответ</summary>

**D) Дочерний Container ограничивается constraints родителя — займёт 200, игнорируя собственные 300**

Флаттеровская constraint система: родитель передаёт constraints вниз, ребёнок не может их превысить. `Container(width: 300)` в `ConstrainedBox(maxWidth: 200)` → 200px. Для обхода: `UnconstrainedBox` (но ребёнок может overflow).

</details>

---

### Вопрос 10 🔴

Какой виджет реализует `Container` под капотом? Назовите основные слои.

- A) `Container` — нативный виджет без разбиения
- B) `Container` компонуется из `LimitedBox` + `ConstrainedBox` + `Align` + `Padding` + `DecoratedBox` + `Transform` в зависимости от переданных параметров
- C) `Container` = `Material` + `InkWell`
- D) `Container` = `SizedBox` + `ColoredBox`

<details>
<summary>Ответ</summary>

**B) Container компонуется из нескольких виджетов в зависимости от параметров**

Порядок вложения (снаружи → внутрь):

1. `Align` (если задан alignment)
2. `ConstrainedBox` (если constraints/width/height)
3. `Padding` (для margin — через Align+LimitedBox)
4. `DecoratedBox` (foreground decoration)
5. `Padding` (padding)
6. `DecoratedBox` (background decoration)
7. `Transform` (если задан transform)
8. child

Поэтому лучше использовать специализированные виджеты напрямую, если нужен только один из эффектов.

</details>
