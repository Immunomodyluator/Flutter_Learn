# Квиз: Column и Row

**Тема:** 04.1 — Column & Row  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как центрировать дочерние элементы `Column` по горизонтали?

- A) `alignment: Alignment.center`
- B) `crossAxisAlignment: CrossAxisAlignment.center`
- C) `mainAxisAlignment: MainAxisAlignment.center`
- D) `horizontalAlignment: HorizontalAlignment.center`

<details>
<summary>Ответ</summary>

**B) `crossAxisAlignment: CrossAxisAlignment.center`**

В `Column` главная ось — вертикальная, поперечная — горизонтальная. `crossAxisAlignment` управляет горизонтальным выравниванием. `mainAxisAlignment` — вертикальным распределением (spaceBetween, center, spaceAround...).

</details>

---

### Вопрос 2 🟢

Что произойдёт если `Column` расположена внутри другой `Column` с детьми без ограничений высоты?

- A) Автоматически добавятся скроллбары
- B) `RenderFlex overflowed` — ошибка overflow
- C) Дочерние элементы сожмутся до нуля
- D) Ничего — Flutter справляется автоматически

<details>
<summary>Ответ</summary>

**B) `RenderFlex overflowed` — ошибка overflow**

`Column` занимает всё доступное пространство и распределяет его между детьми. Если детей больше чем высоты — overflow. Решение: обернуть в `SingleChildScrollView`, использовать `ListView` или `Expanded` / `Flexible`.

</details>

---

### Вопрос 3 🟢

Какое отличие `MainAxisAlignment.spaceEvenly` от `spaceBetween`?

- A) Они одинаковы
- B) `spaceEvenly` — равные промежутки **включая** края; `spaceBetween` — только **между** элементами, без краёв
- C) `spaceBetween` работает только в `Row`
- D) `spaceEvenly` добавляет отступ только справа

<details>
<summary>Ответ</summary>

**B) `spaceEvenly` — равные промежутки включая края; `spaceBetween` — только между элементами, без краёв**

- `spaceBetween`: [item] — [item] — [item] (крайние элементы у границ)
- `spaceAround`: space/2 [item] space [item] space/2 (половинные отступы у краёв)
- `spaceEvenly`: space [item] space [item] space (равные отступы везде)

</details>

---

### Вопрос 4 🟡

Что делает `Expanded` внутри `Row`?

- A) Растягивает дочерний виджет по поперечной оси
- B) Заставляет дочерний виджет занять всё оставшееся пространство по главной оси
- C) Добавляет padding вокруг виджета
- D) Центрирует виджет

<details>
<summary>Ответ</summary>

**B) Заставляет дочерний виджет занять всё оставшееся пространство по главной оси**

`Expanded(child: Widget)` — эквивалентно `Flexible(fit: FlexFit.tight)`. При нескольких `Expanded` пространство делится по `flex` коэффициентам: `Expanded(flex: 2, ...)` занимает вдвое больше чем `Expanded(flex: 1, ...)`.

</details>

---

### Вопрос 5 🟡

Чем `Flexible` отличается от `Expanded`?

- A) Они идентичны
- B) `Flexible` растягивается максимум до доступного пространства, но **может** быть меньше; `Expanded` — всегда занимает всё оставшееся место
- C) `Flexible` работает только с `Column`
- D) `Expanded` можно использовать вне Flex-контейнеров

<details>
<summary>Ответ</summary>

**B) `Flexible` растягивается максимум до доступного пространства, но может быть меньше; `Expanded` — всегда занимает всё оставшееся место**

`Flexible(fit: FlexFit.loose)` — дочерний виджет может быть меньше выделенного пространства. `Expanded` = `Flexible(fit: FlexFit.tight)` — дочерний виджет **обязан** заполнить всё выделенное пространство.

</details>

---

### Вопрос 6 🟡

Что такое `mainAxisSize: MainAxisSize.min` в `Column`?

- A) Устанавливает минимальную высоту 0
- B) Column занимает только минимально необходимое пространство по главной оси (wrap children)
- C) Запрещает скроллинг
- D) Ограничивает количество children

<details>
<summary>Ответ</summary>

**B) Column занимает только минимально необходимое пространство по главной оси (wrap children)**

По умолчанию `MainAxisSize.max` — Column занимает всю доступную высоту. `min` — сжимается до высоты всех детей. Полезно в `AlertDialog`, `BottomSheet`, всплывающих карточках.

</details>

---

### Вопрос 7 🟡

Как добавить равные промежутки между виджетами в `Column` без `mainAxisAlignment`?

- A) Использовать `Padding` вокруг каждого виджета
- B) `ListView.separated`
- C) `Column` с генерацией через `intersperse` паттерн или `gap` пакет
- D) `SizedBox` между каждым виджетом

<details>
<summary>Ответ</summary>

**D) `SizedBox` между каждым виджетом**

`SizedBox(height: 8)` или `SizedBox(width: 8)` — стандартный Flutter паттерн для отступов. Пакет `gap` предоставляет удобный `Gap(8)`. Для динамических списков — `ListView.separated(separatorBuilder: ...)`.

</details>

---

### Вопрос 8 🔴

Что выведет следующий код?

```dart
Row(
  children: [
    Container(width: 100, height: 50, color: Colors.red),
    Expanded(child: Container(height: 50, color: Colors.blue)),
    Container(width: 80, height: 50, color: Colors.green),
  ],
)
```

Предположим Row шириной 300.

- A) Красный: 100, Синий: 120, Зелёный: 80
- B) Красный: 100, Синий: 300, Зелёный: 80
- C) Все одинаковые (100px)
- D) `RenderFlex overflow`

<details>
<summary>Ответ</summary>

**A) Красный: 100, Синий: 120, Зелёный: 80**

`Expanded` занимает оставшееся пространство: 300 - 100 - 80 = 120px. Фиксированные контейнеры занимают свои размеры, `Expanded` получает всё остальное.

</details>

---

### Вопрос 9 🔴

Что произойдёт при использовании `Column` внутри `ListView`?

- A) Нормально работает
- B) `Column` получает unbounded height — нужно обернуть `Column` в `Shrinkwrap` или заменить на `ListView` с `shrinkWrap: true`
- C) `ListView` отключает собственный скроллинг
- D) Автоматически применяется `Expanded`

<details>
<summary>Ответ</summary>

**B) `Column` получает unbounded height — нужно завернуть в `shrinkWrap` или заменить на вложенный `ListView(shrinkWrap: true)`**

`ListView` даёт детям неограниченную высоту. `Column` без ограничений пытается занять бесконечное пространство → исключение. Решение: `Column` с `mainAxisSize: MainAxisSize.min` или `ListView(shrinkWrap: true, physics: NeverScrollableScrollPhysics())`.

</details>

---

### Вопрос 10 🔴

Как работает `crossAxisAlignment: CrossAxisAlignment.baseline` в `Row`?

- A) Все дочерние виджеты выравниваются по нижнему краю
- B) Текстовые базовые линии всех `Text` виджетов выравниваются — требует указания `textBaseline`
- C) Работает только с `Column`
- D) Выравнивание по центру высоты

<details>
<summary>Ответ</summary>

**B) Текстовые базовые линии всех Text виджетов выравниваются — требует указания `textBaseline`**

`CrossAxisAlignment.baseline` нужно использовать с `textBaseline: TextBaseline.alphabetic`. Это выравнивает текст разных размеров так, что их базовые линии совпадают (нижний край строчных букв). Полезно для строк с текстами разных размеров.

</details>
