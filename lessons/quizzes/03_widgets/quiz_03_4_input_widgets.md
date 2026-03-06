# Квиз: Input Widgets

**Тема:** 03.4 — Input Widgets  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как получить текст, введённый в `TextField`?

- A) `TextField.value`
- B) Через `TextEditingController` привязанный к `controller` параметру
- C) Через `onChanged` с `Text` объектом
- D) `TextField.getText()`

<details>
<summary>Ответ</summary>

**B) Через `TextEditingController` привязанный к `controller` параметру**

```dart
final controller = TextEditingController();
TextField(controller: controller);
// Чтение: controller.text
// Обработка: controller.addListener(() { ... })
```

`TextEditingController` также позволяет программно изменять текст, управлять выделением и позицией курсора.

</details>

---

### Вопрос 2 🟢

Что делает `obscureText: true` в `TextField`?

- A) Скрывает поле от скринридеров
- B) Скрывает введённые символы (режим пароля)
- C) Делает поле только для чтения
- D) Отключает автокоррекцию

<details>
<summary>Ответ</summary>

**B) Скрывает введённые символы (режим пароля)**

`obscureText: true` заменяет символы точками или звёздочками — стандартный режим поля пароля. Сочетается с `keyboardType: TextInputType.visiblePassword` и кнопкой «показать пароль» через переключение `obscureText`.

</details>

---

### Вопрос 3 🟢

Что такое `Checkbox` в Flutter?

- A) Виджет выбора из нескольких вариантов
- B) Бинарный виджет выбора (true/false), не хранит состояния — требует внешнего управления
- C) Переключатель для групп настроек
- D) Компонент выпадающего списка

<details>
<summary>Ответ</summary>

**B) Бинарный виджет выбора (true/false), не хранит состояния — требует внешнего управления**

`Checkbox(value: _checked, onChanged: (v) => setState(() => _checked = v!))` — управляемый (controlled) виджет. `CheckboxListTile` — с label. `TriStateCheckbox` — поддерживает null (неопределённое состояние).

</details>

---

### Вопрос 4 🟡

Чем `Form` виджет отличается от набора отдельных `TextField`?

- A) `Form` быстрее
- B) `Form` + `TextFormField` предоставляют валидацию, сброс и сохранение всех полей через `FormState`
- C) `Form` не требует `key`
- D) `Form` автоматически отправляет данные

<details>
<summary>Ответ</summary>

**B) `Form` + `TextFormField` предоставляют валидацию, сброс и сохранение всех полей через `FormState`**

```dart
final _formKey = GlobalKey<FormState>();
Form(
  key: _formKey,
  child: TextFormField(validator: (v) => v!.isEmpty ? 'Required' : null),
);
// Валидация: _formKey.currentState!.validate()
// Сброс: _formKey.currentState!.reset()
```

</details>

---

### Вопрос 5 🟡

Что такое `FocusNode` и для чего нужен?

- A) Узел в дереве сфокусированных элементов
- B) Объект для управления фокусом клавиатуры у конкретного `TextField`
- C) Обработчик событий нажатия клавиш
- D) Менеджер очереди ввода

<details>
<summary>Ответ</summary>

**B) Объект для управления фокусом клавиатуры у конкретного `TextField`**

`FocusNode` — ручка к фокусному состоянию поля. Используется для: программного фокуса (`node.requestFocus()`), перехода между полями (`FocusScope.of(context).nextFocus()`), прослушивания изменений фокуса (`node.addListener(() {})`). Нужно вызывать `node.dispose()` в `dispose()`.

</details>

---

### Вопрос 6 🟡

Что делает `TextInputAction.next` в `TextField`?

- A) Переходит к следующему экрану
- B) Меняет кнопку «Enter» на клавиатуре на «Далее» и позволяет переключиться на следующее поле
- C) Добавляет новую строку
- D) Отправляет форму

<details>
<summary>Ответ</summary>

**B) Меняет кнопку «Enter» на клавиатуре на «Далее» и позволяет переключиться на следующее поле**

`textInputAction: TextInputAction.next` + `onSubmitted: (_) => FocusScope.of(context).nextFocus()` — стандартный UX паттерн для форм. `TextInputAction.done` — «Готово» (закрывает клавиатуру). `TextInputAction.search` — иконка лупы.

</details>

---

### Вопрос 7 🟡

Что делает `DropdownButton`?

- A) Открывает диалог выбора
- B) Показывает выпадающий список вариантов
- C) Открывает `BottomSheet`
- D) Переключает между Radio вариантами

<details>
<summary>Ответ</summary>

**B) Показывает выпадающий список вариантов**

`DropdownButton<T>(value: _selected, items: [...DropdownMenuItem...], onChanged: (v) => setState(() => _selected = v!))`. Material 3 Рекомендует `DropdownMenu` — новый компонент с поиском и кастомным вводом.

</details>

---

### Вопрос 8 🔴

Как реализовать real-time валидацию поля (проверку при каждом вводе символа)?

- A) `validator` в `TextFormField` автоматически вызывается при вводе
- B) `autovalidateMode: AutovalidateMode.onUserInteraction` или `onChange` + `_formKey.currentState!.validate()`
- C) Только через кастомный `TextInputFormatter`
- D) `TextField(realTimeValidation: true)`

<details>
<summary>Ответ</summary>

**B) `autovalidateMode: AutovalidateMode.onUserInteraction` или `onChange` + `_formKey.currentState!.validate()`**

`AutovalidateMode.onUserInteraction` — запускает `validator` автоматически после первого взаимодействия. `AutovalidateMode.always` — валидирует с первого рендера. По умолчанию — `disabled` (только при явном `validate()`).

</details>

---

### Вопрос 9 🔴

Что такое `TextInputFormatter`?

- A) Форматирует текст для отображения (не изменяя `controller.text`)
- B) Фильтрует и трансформирует ввод до установки в поле — например, маски, форматы телефонов
- C) Применяет стили к введённому тексту
- D) Проверяет ввод после потери фокуса

<details>
<summary>Ответ</summary>

**B) Фильтрует и трансформирует ввод до установки в поле — например, маски, форматы телефонов**

```dart
TextField(
  inputFormatters: [
    FilteringTextInputFormatter.digitsOnly,        // только цифры
    LengthLimitingTextInputFormatter(10),           // макс 10 символов
    // кастомный: класс extends TextInputFormatter
  ],
)
```

Formatters применяются в порядке списка при каждом изменении.

</details>

---

### Вопрос 10 🔴

Как отобразить `DatePicker` и получить выбранную дату?

- A) `DatePicker(onSelect: (date) => ...)`
- B) `final date = await showDatePicker(context: context, initialDate: ..., firstDate: ..., lastDate: ...)`
- C) `showCalendar(context, onDateSelected: ...)`
- D) `Navigator.push(context, DatePickerRoute())`

<details>
<summary>Ответ</summary>

**B) `final date = await showDatePicker(context: context, initialDate: ..., firstDate: ..., lastDate: ...)`**

`showDatePicker` возвращает `Future<DateTime?>` — null если пользователь отменил. Параметры `firstDate` и `lastDate` — обязательные ограничения диапазона. Аналогично `showTimePicker` для времени. Оба стилизуются через `datePickerTheme` в `ThemeData`.

</details>
