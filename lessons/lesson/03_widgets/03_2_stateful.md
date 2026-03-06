# 3.2 StatefulWidget и State

## 1. Суть концепции

`StatefulWidget` — виджет, у которого есть изменяемое **состояние** (`State`). Когда состояние меняется через `setState()`, Flutter пересобирает `build()` этого виджета.

**Разделение ответственности:**

- `StatefulWidget` — неизменяемая конфигурация (параметры конструктора)
- `State<T>` — изменяемые данные + жизненный цикл + `build()`

---

## 2. Базовый синтаксис

```dart
class CounterWidget extends StatefulWidget {
  final String label;

  const CounterWidget({super.key, required this.label});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int _count = 0;  // изменяемое состояние

  void _increment() {
    setState(() {         // уведомляет Flutter о необходимости rebuild
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('${widget.label}: $_count'),  // widget.xxx — доступ к параметрам
        ElevatedButton(
          onPressed: _increment,
          child: const Text('+'),
        ),
      ],
    );
  }
}
```

---

## 3. Жизненный цикл State

```
createState()
    ↓
initState()          // вызывается один раз при создании
    ↓
didChangeDependencies()  // после initState + при изменении InheritedWidget
    ↓
build()              // вызывается при каждом rebuild
    ↓
setState() ──────→ build() (повтор)
    ↓
didUpdateWidget()    // когда родитель передаёт новые параметры
    ↓
dispose()            // при удалении из дерева (важно: освобождай ресурсы)
```

---

## 4. Практический пример: форма с валидацией

```dart
class LoginForm extends StatefulWidget {
  const LoginForm({super.key});

  @override
  State<LoginForm> createState() => _LoginFormState();
}

class _LoginFormState extends State<LoginForm> {
  final _formKey = GlobalKey<FormState>();
  final _emailController = TextEditingController();
  final _passwordController = TextEditingController();
  bool _obscurePassword = true;
  bool _isLoading = false;

  @override
  void dispose() {
    // ОБЯЗАТЕЛЬНО освобождай контроллеры
    _emailController.dispose();
    _passwordController.dispose();
    super.dispose();
  }

  Future<void> _submit() async {
    if (!_formKey.currentState!.validate()) return;

    setState(() => _isLoading = true);

    try {
      await AuthService.login(
        _emailController.text.trim(),
        _passwordController.text,
      );
      if (!mounted) return; // проверка после await
      Navigator.of(context).pushReplacementNamed('/home');
    } catch (e) {
      if (!mounted) return;
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text('Ошибка: $e')),
      );
    } finally {
      if (mounted) setState(() => _isLoading = false);
    }
  }

  @override
  Widget build(BuildContext context) {
    return Form(
      key: _formKey,
      child: Column(
        children: [
          TextFormField(
            controller: _emailController,
            decoration: const InputDecoration(labelText: 'Email'),
            validator: (v) => v!.contains('@') ? null : 'Неверный email',
          ),
          TextFormField(
            controller: _passwordController,
            obscureText: _obscurePassword,
            decoration: InputDecoration(
              labelText: 'Пароль',
              suffixIcon: IconButton(
                icon: Icon(_obscurePassword ? Icons.visibility : Icons.visibility_off),
                onPressed: () => setState(() => _obscurePassword = !_obscurePassword),
              ),
            ),
            validator: (v) => v!.length >= 6 ? null : 'Минимум 6 символов',
          ),
          const SizedBox(height: 16),
          _isLoading
              ? const CircularProgressIndicator()
              : FilledButton(onPressed: _submit, child: const Text('Войти')),
        ],
      ),
    );
  }
}
```

---

## 5. initState — правильное использование

```dart
@override
void initState() {
  super.initState();                      // ВСЕГДА вызывать первым
  _controller = AnimationController(
    vsync: this,
    duration: const Duration(milliseconds: 300),
  );
  _loadData();                            // запуск async без async initState
}

Future<void> _loadData() async {
  final data = await repository.fetch();
  if (!mounted) return;                   // проверка mounted после await
  setState(() => _items = data);
}
```

**Нельзя делать `initState` async** — это синхронный метод жизненного цикла.

---

## 6. didUpdateWidget

```dart
@override
void didUpdateWidget(MyWidget oldWidget) {
  super.didUpdateWidget(oldWidget);
  // вызывается когда родитель перестраивается с новыми параметрами
  if (oldWidget.userId != widget.userId) {
    _loadUserData(widget.userId); // перезагрузить данные при смене ID
  }
}
```

---

## 7. Типичные ошибки

| Ошибка                            | Проблема                             | Решение                                 |
| --------------------------------- | ------------------------------------ | --------------------------------------- |
| `setState()` в `initState()`      | Бесполезно, build ещё не вызывался   | Просто присвоить значение               |
| Не вызван `super.dispose()`       | Утечка ресурсов фреймворка           | `super.dispose()` последним             |
| Не освобождены контроллеры        | Memory leak                          | `controller.dispose()` в `dispose()`    |
| `setState()` после `dispose()`    | Ошибка "called after dispose"        | Проверять `if (!mounted) return`        |
| Большое состояние в одном виджете | Трудно тестировать/поддерживать      | Разбить на части или поднять в Provider |
| `async initState()`               | Нарушение контракта жизненного цикла | Отдельный метод `_init()`               |

---

## 8. Практические рекомендации

1. **Минимизируй состояние** — держи только то, что реально изменяется.
2. **`if (!mounted) return`** — всегда после `await` перед использованием `context` или `setState`.
3. **`dispose()` — обязателен** при наличии `TextEditingController`, `AnimationController`, `ScrollController`, подписок.
4. **`widget.xxx`** — обращение к параметрам родительского виджета из `State`.
5. **Поднимай состояние вверх**, если два виджета нуждаются в одних данных.
6. **`_name` с underscore** — все поля State приватные по конвенции.
