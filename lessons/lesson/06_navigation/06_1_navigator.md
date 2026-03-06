# 6.1 Navigator 1.0

## 1. Суть концепции

Navigator 1.0 — императивный API для управления стеком экранов. Использует `push`/`pop` паттерн — экраны укладываются в стек, как страницы истории браузера.

`Navigator` — виджет, управляющий стеком `Route`. `Route` — обёртка над экраном с анимацией перехода.

---

## 2. MaterialPageRoute — стандартный переход

```dart
// Переход на новый экран
Future<void> _goToDetails(BuildContext context) async {
  await Navigator.push<void>(
    context,
    MaterialPageRoute(
      builder: (context) => const DetailsScreen(),
      fullscreenDialog: false,  // true = iOS-стиль снизу вверх
    ),
  );
}

// Кастомная анимация перехода
Navigator.push(
  context,
  PageRouteBuilder(
    pageBuilder: (_, animation, __) => FadeTransition(
      opacity: animation,
      child: const DetailsScreen(),
    ),
    transitionDuration: const Duration(milliseconds: 300),
  ),
);
```

---

## 3. Возврат данных между экранами

```dart
// Экран A — ждёт результат
Future<void> _openPicker() async {
  final selected = await Navigator.push<String>(
    context,
    MaterialPageRoute(builder: (_) => const ColorPickerScreen()),
  );
  if (selected != null && mounted) {
    setState(() => _selectedColor = selected);
  }
}

// Экран B — возвращает результат
ElevatedButton(
  onPressed: () => Navigator.pop(context, 'red'),
  child: const Text('Выбрать красный'),
)
```

---

## 4. Управление стеком навигации

```dart
// Стандартные операции
Navigator.push(context, route);           // добавить экран
Navigator.pop(context);                   // убрать текущий
Navigator.pop(context, result);           // убрать + вернуть данные

// Заменить текущий экран
Navigator.pushReplacement(context, MaterialPageRoute(builder: (_) => NewScreen()));

// Убрать все экраны до указанного
Navigator.popUntil(context, ModalRoute.withName('/home'));

// Добавить и очистить стек
Navigator.pushAndRemoveUntil(
  context,
  MaterialPageRoute(builder: (_) => HomeScreen()),
  (route) => false,                       // убрать все предыдущие
);

// Проверить, можно ли вернуться
if (Navigator.canPop(context)) {
  Navigator.pop(context);
}

// Получить инстанс Navigator
final navigator = Navigator.of(context);
```

---

## 5. WillPopScope — перехват кнопки "Назад"

```dart
// Flutter 3.12+: PopScope (замена WillPopScope)
PopScope(
  canPop: false,  // false = запретить pop
  onPopInvoked: (didPop) async {
    if (didPop) return;
    // Показать диалог подтверждения
    final shouldPop = await showDialog<bool>(
      context: context,
      builder: (_) => AlertDialog(
        title: const Text('Выйти?'),
        content: const Text('Несохранённые изменения будут потеряны'),
        actions: [
          TextButton(onPressed: () => Navigator.pop(context, false), child: const Text('Отмена')),
          FilledButton(onPressed: () => Navigator.pop(context, true), child: const Text('Выйти')),
        ],
      ),
    );
    if (shouldPop == true && context.mounted) {
      Navigator.pop(context);
    }
  },
  child: Scaffold(...),
)
```

---

## 6. GlobalKey<NavigatorState> — навигация вне контекста

```dart
// Объявить глобальный ключ (в main.dart или router)
final navigatorKey = GlobalKey<NavigatorState>();

// Передать в MaterialApp
MaterialApp(
  navigatorKey: navigatorKey,
  home: const HomeScreen(),
)

// Использовать из любого места (например, из сервиса уведомлений)
navigatorKey.currentState?.push(
  MaterialPageRoute(builder: (_) => NotificationScreen()),
);

// Или через context
navigatorKey.currentContext; // BuildContext если нужен
```

---

## 7. Реальный пример: авторизация

```dart
class AuthGuard extends StatelessWidget {
  final Widget child;
  const AuthGuard({super.key, required this.child});

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<User?>(
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const CircularProgressIndicator();
        }
        if (snapshot.data == null) {
          // Пользователь не авторизован — показываем LoginScreen
          // Используем Future.microtask чтобы избежать setState во время build
          Future.microtask(() => Navigator.pushAndRemoveUntil(
            context,
            MaterialPageRoute(builder: (_) => const LoginScreen()),
            (_) => false,
          ));
        }
        return child;
      },
    );
  }
}
```

---

## 8. Типичные ошибки

| Ошибка                              | Проблема             | Решение                                               |
| ----------------------------------- | -------------------- | ----------------------------------------------------- |
| `Navigator.pop()` на пустой стек    | Exception            | `if (Navigator.canPop(context))`                      |
| Навигация в `initState` напрямую    | Context не готов     | Через `Future.microtask()` или `addPostFrameCallback` |
| `context` после async без `mounted` | BuildContext устарел | `if (!mounted) return` перед навигацией               |
| `WillPopScope` в Flutter 3.12+      | Deprecated           | Использовать `PopScope`                               |

---

## 9. Практические рекомендации

1. **Передавай данные через конструктор** — не через `arguments` / глобальные переменные.
2. **`pushReplacement`** для логина/регистрации — нельзя вернуться назад.
3. **`canPop()`** перед `pop()` — безопаснее.
4. **`GlobalKey<NavigatorState>`** если нужна навигация из сервиса без `BuildContext`.
5. **`PopScope`** вместо `WillPopScope` в новых проектах (Flutter 3.12+).
