# 7.4 Riverpod

## 1. Суть концепции

Riverpod — следующее поколение Provider от того же автора (Remi Rousselet). Решает главные проблемы Provider:
- Типобезопасность без `BuildContext`
- Провайдеры — глобальные compile-time константы, не могут "не найтись"
- Поддержка `async` из коробки (FutureProvider, StreamProvider)
- Автоматическое кеширование и инвалидация
- Отличная поддержка тестирования

```yaml
dependencies:
  flutter_riverpod: ^2.5.0
  riverpod_annotation: ^2.3.0  # для кодогенерации (опционально)
dev_dependencies:
  riverpod_generator: ^2.4.0
  build_runner: ^2.4.0
```

---

## 2. ProviderScope — корень приложения

```dart
void main() {
  runApp(
    const ProviderScope(  // обязательная обёртка
      child: MyApp(),
    ),
  );
}
```

---

## 3. Типы провайдеров

```dart
// Provider — неизменяемое значение / зависимость
final apiServiceProvider = Provider<ApiService>((ref) => ApiService());

// StateProvider — простое изменяемое значение
final counterProvider = StateProvider<int>((ref) => 0);
final selectedTabProvider = StateProvider<int>((ref) => 0);

// NotifierProvider — сложное состояние с методами (замена ChangeNotifier)
final cartProvider = NotifierProvider<CartNotifier, CartState>(CartNotifier.new);

// AsyncNotifierProvider — асинхронное состояние
final productsProvider = AsyncNotifierProvider<ProductsNotifier, List<Product>>(
  ProductsNotifier.new,
);

// FutureProvider — одноразовый Future
final userProfileProvider = FutureProvider.family<UserProfile, String>(
  (ref, userId) => ref.read(apiServiceProvider).getUserProfile(userId),
);

// StreamProvider — подписка на Stream
final messagesProvider = StreamProvider.autoDispose<List<Message>>(
  (ref) => FirebaseFirestore.instance.collection('messages').snapshots()
    .map((s) => s.docs.map(Message.fromDoc).toList()),
);
```

---

## 4. Notifier — состояние с логикой

```dart
// Состояние
class CartState {
  final List<Product> items;
  const CartState({this.items = const []});

  double get total => items.fold(0, (s, p) => s + p.price);

  CartState copyWith({List<Product>? items}) =>
      CartState(items: items ?? this.items);
}

// Notifier
class CartNotifier extends Notifier<CartState> {
  @override
  CartState build() => const CartState(); // начальное состояние

  void add(Product product) {
    state = state.copyWith(items: [...state.items, product]);
  }

  void remove(Product product) {
    state = state.copyWith(
      items: state.items.where((p) => p != product).toList(),
    );
  }

  void clear() {
    state = const CartState();
  }
}

final cartProvider = NotifierProvider<CartNotifier, CartState>(CartNotifier.new);
```

---

## 5. Чтение провайдеров в виджетах

```dart
// ConsumerWidget — аналог StatelessWidget с ref
class CartIcon extends ConsumerWidget {
  const CartIcon({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final cart = ref.watch(cartProvider);  // подписка
    return Badge(
      label: Text('${cart.items.length}'),
      child: IconButton(
        icon: const Icon(Icons.shopping_cart),
        onPressed: () => ref.read(cartProvider.notifier).clear(), // действие
      ),
    );
  }
}

// ConsumerStatefulWidget — аналог StatefulWidget с ref
class ProductPage extends ConsumerStatefulWidget {
  const ProductPage({super.key});

  @override
  ConsumerState<ProductPage> createState() => _ProductPageState();
}

class _ProductPageState extends ConsumerState<ProductPage> {
  @override
  void initState() {
    super.initState();
    // ref доступен в initState через ref.read (не ref.watch)
    Future.microtask(() => ref.read(productsProvider));
  }

  @override
  Widget build(BuildContext context) {
    final productsAsync = ref.watch(productsProvider);
    return productsAsync.when(
      data: (products) => ProductGrid(products: products),
      loading: () => const CircularProgressIndicator(),
      error: (e, _) => Text('Ошибка: $e'),
    );
  }
}
```

---

## 6. AsyncNotifier и AsyncValue

```dart
class ProductsNotifier extends AsyncNotifier<List<Product>> {
  @override
  Future<List<Product>> build() async {
    // Вызывается при первом обращении
    return ref.read(apiServiceProvider).getProducts();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() =>
      ref.read(apiServiceProvider).getProducts(),
    );
  }
}

// В виджете
final products = ref.watch(productsProvider);

products.when(
  data: (list) => ListView(...),
  loading: () => const CircularProgressIndicator(),
  error: (err, stack) => ErrorWidget(err),
)

// Или с паттерном .value / .error / .isLoading
if (products.isLoading) return const CircularProgressIndicator();
if (products.hasError) return Text('Ошибка: ${products.error}');
final list = products.requireValue;
```

---

## 7. family — параметризованные провайдеры

```dart
// Провайдер с параметром
final productByIdProvider = FutureProvider.family<Product, String>(
  (ref, id) => ref.read(apiServiceProvider).getProduct(id),
);

// Использование
final product = ref.watch(productByIdProvider('product-123'));
```

---

## 8. autoDispose — автоматическая очистка

```dart
// Провайдер уничтожается когда нет слушателей
final searchResultsProvider = FutureProvider.autoDispose.family<List<Product>, String>(
  (ref, query) async {
    // Отменить предыдущий запрос при новом поиске
    ref.onDispose(() => print('Disposed search: $query'));
    return api.search(query);
  },
);
```

---

## 9. Типичные ошибки

| Ошибка | Проблема | Решение |
|--------|---------|---------|
| `ref.watch` в коллбэке | Нельзя вызывать вне `build` | Использовать `ref.read` в коллбэках |
| Нет `ProviderScope` | `MissingProviderScopeException` | Обернуть `runApp` в `ProviderScope` |
| `ref.watch` в `initState` | Ошибка | Только `ref.read` в `initState` |
| Большой Notifier | Сложно тестировать | Разбить на несколько провайдеров |

---

## 10. Практические рекомендации

1. **`ref.watch` в `build()`**, **`ref.read` в коллбэках и `initState`**.
2. **`autoDispose`** для провайдеров экранов — освобождает память при уходе.
3. **`family`** для параметризованных запросов (по ID).
4. **`AsyncNotifier`** для экранов с загрузкой данных.
5. **Тестируй через `ProviderContainer`** без Flutter — unit-тесты провайдеров.
6. **Riverpod предпочтительнее Provider** для новых проектов.
