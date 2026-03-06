# 7.3 Provider

## 1. Суть концепции

Provider — самый популярный пакет управления состоянием во Flutter (от Remi Rousselet, официально рекомендован Flutter team). Построен поверх InheritedWidget, но значительно упрощает API.

```yaml
dependencies:
  provider: ^6.1.0
```

---

## 2. ChangeNotifier — основа Provider

```dart
// Модель (бизнес-логика + состояние)
class CartNotifier extends ChangeNotifier {
  final List<Product> _items = [];

  List<Product> get items => List.unmodifiable(_items);
  int get count => _items.length;
  double get total => _items.fold(0, (sum, p) => sum + p.price);

  void add(Product product) {
    _items.add(product);
    notifyListeners();  // уведомить UI об изменении
  }

  void remove(Product product) {
    _items.remove(product);
    notifyListeners();
  }

  void clear() {
    _items.clear();
    notifyListeners();
  }
}
```

---

## 3. Размещение Provider в дереве

```dart
// Один провайдер
ChangeNotifierProvider(
  create: (_) => CartNotifier(),
  child: MaterialApp(home: const HomeScreen()),
)

// Несколько провайдеров
MultiProvider(
  providers: [
    ChangeNotifierProvider(create: (_) => CartNotifier()),
    ChangeNotifierProvider(create: (_) => AuthNotifier()),
    ChangeNotifierProvider(create: (_) => ThemeNotifier()),
    Provider<ApiService>(create: (_) => ApiService()),  // без ChangeNotifier
  ],
  child: MaterialApp(home: const HomeScreen()),
)
```

---

## 4. Чтение данных из Provider

```dart
// context.watch<T>() — подписаться, rebuild при изменении
class CartBadge extends StatelessWidget {
  const CartBadge({super.key});

  @override
  Widget build(BuildContext context) {
    final cart = context.watch<CartNotifier>();
    return Badge(
      label: Text('${cart.count}'),
      child: const Icon(Icons.shopping_cart),
    );
  }
}

// context.read<T>() — прочитать без подписки (в callback-ах)
class AddToCartButton extends StatelessWidget {
  final Product product;
  const AddToCartButton({super.key, required this.product});

  @override
  Widget build(BuildContext context) {
    return FilledButton(
      onPressed: () => context.read<CartNotifier>().add(product), // не rebuild
      child: const Text('В корзину'),
    );
  }
}

// context.select<T, R>() — подписаться только на часть состояния
class CartTotal extends StatelessWidget {
  const CartTotal({super.key});

  @override
  Widget build(BuildContext context) {
    // Rebuild только когда total изменится, а не при любом изменении CartNotifier
    final total = context.select<CartNotifier, double>((cart) => cart.total);
    return Text('Итого: ${total.toStringAsFixed(2)} ₽');
  }
}
```

---

## 5. Consumer — альтернатива context.watch

```dart
// Consumer<T> — только выделенная часть дерева перестраивается
Scaffold(
  appBar: AppBar(
    title: const Text('Магазин'),
    actions: [
      Consumer<CartNotifier>(
        builder: (context, cart, child) {
          return Badge(
            label: Text('${cart.count}'),
            child: child!, // статичная часть не перестраивается
          );
        },
        child: const Icon(Icons.shopping_cart), // статичный child
      ),
    ],
  ),
  body: const ProductList(),
)
```

---

## 6. Реальный пример: авторизация

```dart
class AuthNotifier extends ChangeNotifier {
  User? _user;
  bool _isLoading = false;

  User? get user => _user;
  bool get isLoading => _isLoading;
  bool get isLoggedIn => _user != null;

  Future<void> login(String email, String password) async {
    _isLoading = true;
    notifyListeners();

    try {
      _user = await AuthApi.login(email, password);
    } catch (e) {
      rethrow;
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  Future<void> logout() async {
    await AuthApi.logout();
    _user = null;
    notifyListeners();
  }
}

// В виджете
class HomeScreen extends StatelessWidget {
  const HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final auth = context.watch<AuthNotifier>();

    if (!auth.isLoggedIn) {
      return const LoginScreen();
    }

    return Scaffold(
      appBar: AppBar(
        title: Text('Привет, ${auth.user!.name}'),
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: () => context.read<AuthNotifier>().logout(),
          ),
        ],
      ),
      body: const MainContent(),
    );
  }
}
```

---

## 7. ProxyProvider — зависимость между провайдерами

```dart
MultiProvider(
  providers: [
    Provider<AuthService>(create: (_) => AuthService()),
    // ApiService зависит от AuthService
    ProxyProvider<AuthService, ApiService>(
      update: (_, authService, __) => ApiService(authService.token),
    ),
    // UserRepository зависит от ApiService
    ProxyProvider<ApiService, UserRepository>(
      update: (_, api, __) => UserRepository(api),
    ),
  ],
  child: MyApp(),
)
```

---

## 8. Типичные ошибки

| Ошибка | Проблема | Решение |
|--------|---------|---------|
| `context.watch` в коллбэке | Ошибка рантайма | Использовать `context.read` в коллбэках |
| `notifyListeners()` в dispose | Ошибка после удаления | Проверять `isDisposed` или не вызывать после `dispose` |
| Provider не найден | `ProviderNotFoundException` | Разместить Provider выше в дереве |
| Слишком большой ChangeNotifier | Всё перестраивается при любом изменении | Разделить на несколько или использовать `select` |

---

## 9. Практические рекомендации

1. **`context.read` в коллбэках**, **`context.watch` в `build()`**.
2. **`context.select`** для оптимизации — подписывайся только на нужное поле.
3. **`Provider` выше `MaterialApp`** для глобального состояния.
4. **`ChangeNotifier` = только состояние и логика** — без UI зависимостей.
5. **`MultiProvider`** для нескольких провайдеров — не вкладывай руками.
6. **Рассмотри Riverpod** при росте сложности — более мощный и типобезопасный.
