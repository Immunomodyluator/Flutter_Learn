# 3.4 Виджеты ввода

## 1. Суть концепции

Виджеты ввода — TextField, Checkbox, Switch, Slider, DatePicker и др. В Flutter ввод управляется через **контроллеры** (для текстовых полей) или хранится в `State` (для переключателей). Для групп полей используется `Form` с `GlobalKey<FormState>`.

---

## 2. TextField и TextFormField

```dart
// Базовый TextField
final controller = TextEditingController();

TextField(
  controller: controller,
  decoration: const InputDecoration(
    labelText: 'Имя',
    hintText: 'Введите имя',
    prefixIcon: Icon(Icons.person),
    border: OutlineInputBorder(),
  ),
  keyboardType: TextInputType.emailAddress,
  textCapitalization: TextCapitalization.sentences,
  maxLines: 1,
  onChanged: (value) => print(value),
  onSubmitted: (value) => _submit(),
)

// TextFormField — для использования внутри Form (с валидацией)
TextFormField(
  validator: (value) {
    if (value == null || value.isEmpty) return 'Поле обязательно';
    if (value.length < 3) return 'Минимум 3 символа';
    return null; // null = валидно
  },
  decoration: const InputDecoration(labelText: 'Имя'),
)
```

---

## 3. Form — группа полей с валидацией

```dart
class RegisterForm extends StatefulWidget {
  const RegisterForm({super.key});

  @override
  State<RegisterForm> createState() => _RegisterFormState();
}

class _RegisterFormState extends State<RegisterForm> {
  final _formKey = GlobalKey<FormState>();
  final _nameCtrl = TextEditingController();
  final _emailCtrl = TextEditingController();

  @override
  void dispose() {
    _nameCtrl.dispose();
    _emailCtrl.dispose();
    super.dispose();
  }

  void _submit() {
    if (_formKey.currentState!.validate()) {
      // все поля валидны
      print('Name: ${_nameCtrl.text}, Email: ${_emailCtrl.text}');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      autovalidateMode: AutovalidateMode.onUserInteraction, // валидация в реалтайм
      child: Column(
        children: [
          TextFormField(
            controller: _nameCtrl,
            decoration: const InputDecoration(labelText: 'Имя'),
            validator: (v) => v!.isEmpty ? 'Введите имя' : null,
          ),
          TextFormField(
            controller: _emailCtrl,
            decoration: const InputDecoration(labelText: 'Email'),
            keyboardType: TextInputType.emailAddress,
            validator: (v) => v!.contains('@') ? null : 'Неверный email',
          ),
          const SizedBox(height: 16),
          FilledButton(onPressed: _submit, child: const Text('Зарегистрироваться')),
        ],
      ),
    );
  }
}
```

---

## 4. Переключатели и выбор

```dart
// Checkbox
bool _checked = false;
Checkbox(
  value: _checked,
  onChanged: (val) => setState(() => _checked = val!),
)

// CheckboxListTile — Checkbox + Label
CheckboxListTile(
  title: const Text('Принять условия'),
  value: _checked,
  onChanged: (val) => setState(() => _checked = val!),
)

// Switch
bool _enabled = true;
Switch(
  value: _enabled,
  onChanged: (val) => setState(() => _enabled = val),
)

SwitchListTile(
  title: const Text('Уведомления'),
  subtitle: const Text('Получать push-уведомления'),
  value: _enabled,
  onChanged: (val) => setState(() => _enabled = val),
)

// Radio
String? _selectedOption;
Column(
  children: ['Опция A', 'Опция B', 'Опция C'].map((option) =>
    RadioListTile<String>(
      title: Text(option),
      value: option,
      groupValue: _selectedOption,
      onChanged: (v) => setState(() => _selectedOption = v),
    ),
  ).toList(),
)

// Slider
double _value = 50;
Slider(
  value: _value,
  min: 0,
  max: 100,
  divisions: 10,
  label: _value.round().toString(),
  onChanged: (v) => setState(() => _value = v),
)
```

---

## 5. Выбор даты и времени

```dart
// DatePicker
Future<void> _pickDate() async {
  final date = await showDatePicker(
    context: context,
    initialDate: DateTime.now(),
    firstDate: DateTime(2020),
    lastDate: DateTime(2030),
  );
  if (date != null && mounted) {
    setState(() => _selectedDate = date);
  }
}

// TimePicker
Future<void> _pickTime() async {
  final time = await showTimePicker(
    context: context,
    initialTime: TimeOfDay.now(),
  );
  if (time != null && mounted) {
    setState(() => _selectedTime = time);
  }
}
```

---

## 6. DropdownButton и DropdownMenu

```dart
// Material 3: DropdownMenu (предпочтительно)
String? _selectedCity;
DropdownMenu<String>(
  initialSelection: _selectedCity,
  label: const Text('Город'),
  onSelected: (value) => setState(() => _selectedCity = value),
  dropdownMenuEntries: ['Москва', 'Спб', 'Казань'].map((city) =>
    DropdownMenuEntry(value: city, label: city),
  ).toList(),
)

// Классический DropdownButton
DropdownButton<String>(
  value: _selectedCity,
  hint: const Text('Выберите город'),
  items: ['Москва', 'Спб'].map((city) =>
    DropdownMenuItem(value: city, child: Text(city)),
  ).toList(),
  onChanged: (val) => setState(() => _selectedCity = val),
)
```

---

## 7. Типичные ошибки

| Ошибка                                             | Проблема                      | Решение                             |
| -------------------------------------------------- | ----------------------------- | ----------------------------------- |
| Не вызван `controller.dispose()`                   | Memory leak                   | Освобождать в `dispose()`           |
| `GlobalKey<FormState>` как локальная переменная    | Пересоздаётся на каждый build | Хранить в `State`                   |
| `form.validate()` без сохранения в `currentState!` | Выбрасывает null              | `_formKey.currentState!.validate()` |
| `autovalidateMode: always` с начала                | Показывает ошибки до ввода    | Использовать `onUserInteraction`    |
| Чтение `controller.text` без `trim()`              | Пробелы в значении            | `controller.text.trim()`            |

---

## 8. Практические рекомендации

1. **`Form` + `TextFormField`** для форм — `TextField` + ручная валидация только для простых случаев.
2. **`AutovalidateMode.onUserInteraction`** — пользователь видит ошибки только после взаимодействия.
3. **`TextInputAction.next`** + `FocusScope.of(context).nextFocus()` — переход между полями по кнопке на клавиатуре.
4. **`dispose()` для всех контроллеров** — каждый `TextEditingController` нужно освобождать.
5. **`InputDecoration` с `border: OutlineInputBorder()`** — для единообразного вида формы.
