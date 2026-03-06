# Квиз: MVVM паттерн

**Тема:** 13.1 — MVVM Architecture  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Какие три слоя составляют MVVM?

- A) Model, View, Variable
- B) Model (данные/логика), View (UI), ViewModel (посредник — состояние и команды)
- C) Manager, Viewer, Module
- D) MVC = MVVM

<details>
<summary>Ответ</summary>

**B) Model, View, ViewModel**

| Слой          | Ответственность                                      |
| ------------- | ---------------------------------------------------- |
| **Model**     | Данные, Repository, API вызовы, бизнес-логика        |
| **ViewModel** | Состояние UI, команды, обработка событий             |
| **View**      | Отображение состояния, делегирует действия ViewModel |

View ← ViewModel ← Model. View не знает о Model напрямую.

</details>

---

### Вопрос 2 🟢

Как реализовать простой ViewModel с `ChangeNotifier`?

- A) `ViewModel extends StatefulWidget`
- B) `ViewModel extends ChangeNotifier` + `notifyListeners()` при изменении состояния + `ChangeNotifierProvider` для предоставления
- C) `abstract class ViewModel implements Listenable`
- D) `ViewModel with ChangeNotifierMixin`

<details>
<summary>Ответ</summary>

**B) `extends ChangeNotifier` + `notifyListeners()`**

```dart
class ProductViewModel extends ChangeNotifier {
  final ProductRepository _repository;

  ProductViewModel(this._repository);

  List<Product> _products = [];
  bool _isLoading = false;
  String? _error;

  List<Product> get products => _products;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> loadProducts() async {
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      _products = await _repository.getProducts();
    } catch (e) {
      _error = e.toString();
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}
```

</details>

---

### Вопрос 3 🟢

Что такое Repository паттерн и зачем он нужен?

- A) Хранилище для Flutter виджетов
- B) Абстракция источника данных — ViewModel не знает откуда данные (API, кэш, DB); упрощает тестирование
- C) Замена Provider для состояния
- D) Паттерн только для Firebase

<details>
<summary>Ответ</summary>

**B) Абстракция источника данных для ViewModel**

```dart
// Абстракция:
abstract class ProductRepository {
  Future<List<Product>> getProducts();
  Future<Product> getProduct(String id);
}

// Реализации:
class RemoteProductRepository implements ProductRepository {
  final ApiClient _api;
  @override Future<List<Product>> getProducts() => _api.get('/products');
}

class MockProductRepository implements ProductRepository {
  @override Future<List<Product>> getProducts() async => [
    Product(id: '1', name: 'Test Product'),
  ];
}

// ViewModel зависит только от абстракции:
class ProductViewModel extends ChangeNotifier {
  ProductViewModel(ProductRepository repository);
}
```

</details>

---

### Вопрос 4 🟡

Как разделить обязанности между ViewModel и Repository?

- A) ViewModel делает все HTTP запросы
- B) ViewModel — UI состояние и бизнес-логика экрана; Repository — источник данных (fetch, cache, sync); Domain-слой (UseCase) — опционально
- C) Repository = ViewModel
- D) Repository только для локального кэша

<details>
<summary>Ответ</summary>

**B) ViewModel — UI state; Repository — data access**

```dart
// Repository: только доступ к данным
class UserRepository {
  final ApiClient _api;
  final UserDao _dao; // local DB

  Future<User> getUser(String id) async {
    final local = await _dao.findById(id);
    if (local != null && !local.isStale) return local;
    final remote = await _api.getUser(id);
    await _dao.upsert(remote);
    return remote;
  }
}

// ViewModel: UI-специфичная логика
class ProfileViewModel extends ChangeNotifier {
  String get formattedJoinDate =>
      DateFormat('dd MMMM yyyy').format(_user!.createdAt);

  bool get canEdit => currentUser?.id == _user?.id;

  Future<void> updateAvatar(File image) async {
    // Валидация UI-уровня:
    if (image.lengthSync() > 5 * 1024 * 1024) {
      _error = 'Файл слишком большой (макс. 5 MB)';
      notifyListeners();
      return;
    }
    await _repository.updateAvatar(image);
  }
}
```

</details>

---

### Вопрос 5 🟡

Как реализовать MVVM с Riverpod вместо ChangeNotifier?

- A) Riverpod не поддерживает MVVM
- B) `NotifierProvider` / `AsyncNotifierProvider` — Notifier = ViewModel; provider = доступ к состоянию; `ref.read(provider.notifier)` — методы ViewModel
- C) `StateProvider` достаточен
- D) Только `StateNotifierProvider`

<details>
<summary>Ответ</summary>

**B) `AsyncNotifierProvider` с Notifier как ViewModel**

```dart
@riverpod
class ProductsViewModel extends _$ProductsViewModel {
  @override
  Future<List<Product>> build() async {
    return ref.read(productRepositoryProvider).getProducts();
  }

  Future<void> addProduct(Product product) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(() async {
      await ref.read(productRepositoryProvider).create(product);
      return ref.read(productRepositoryProvider).getProducts();
    });
  }

  Future<void> deleteProduct(String id) async { ... }
}

// View:
final vm = ref.watch(productsViewModelProvider);
vm.when(
  data: (products) => ProductList(products: products),
  loading: () => const CircularProgressIndicator(),
  error: (e, _) => ErrorWidget(e),
)
```

</details>

---

### Вопрос 6 🟡

Как тестировать ViewModel без сети и базы данных?

- A) Запускать реальные HTTP запросы в тестах
- B) Мокировать Repository зависимость — `MockProductRepository` или `mocktail`; тестировать состояния ViewModel изолированно
- C) Использовать integration test на реальном устройстве
- D) ViewModel нельзя тестировать в unit тестах

<details>
<summary>Ответ</summary>

**B) Mock Repository + unit test ViewModel**

```dart
// Мок:
class MockProductRepository extends Mock implements ProductRepository {}

void main() {
  late ProductViewModel vm;
  late MockProductRepository mockRepo;

  setUp(() {
    mockRepo = MockProductRepository();
    vm = ProductViewModel(mockRepo);
  });

  test('loadProducts sets isLoading then populates list', () async {
    final products = [Product(id: '1', name: 'Test')];
    when(() => mockRepo.getProducts()).thenAnswer((_) async => products);

    expect(vm.isLoading, false);
    final future = vm.loadProducts();
    expect(vm.isLoading, true);
    await future;
    expect(vm.isLoading, false);
    expect(vm.products, products);
    expect(vm.error, null);
  });

  test('loadProducts sets error on failure', () async {
    when(() => mockRepo.getProducts()).thenThrow(Exception('Network error'));
    await vm.loadProducts();
    expect(vm.error, isNotNull);
    expect(vm.products, isEmpty);
  });
}
```

</details>

---

### Вопрос 7 🟡

Как передавать события из View во ViewModel правильно?

- A) View напрямую обращается к Model
- B) View вызывает методы ViewModel (`vm.submit()`, `vm.filter(query)`) — no logic in View
- C) ViewModel подписывается на Stream из View
- D) `GlobalKey<ViewModelState>()`

<details>
<summary>Ответ</summary>

**B) View вызывает методы ViewModel — вся логика в ViewModel**

```dart
// View — только отображение и делегация:
class LoginScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext ctx, WidgetRef ref) {
    final state = ref.watch(loginViewModelProvider);
    final vm = ref.read(loginViewModelProvider.notifier);

    return Column(children: [
      TextField(
        onChanged: vm.onEmailChanged, // делегировать
        decoration: InputDecoration(errorText: state.emailError),
      ),
      TextField(
        onChanged: vm.onPasswordChanged,
        decoration: InputDecoration(errorText: state.passwordError),
      ),
      ElevatedButton(
        onPressed: state.isValid ? vm.submit : null, // ViewModel решает
        child: state.isLoading ? CircularProgressIndicator() : Text('Войти'),
      ),
      if (state.error != null) ErrorText(state.error!),
    ]);
  }
}
```

</details>

---

### Вопрос 8 🔴

Как реализовать одноразовые события (navigation, snackbar) из ViewModel?

- A) Прямой вызов `Navigator.push()` в ViewModel
- B) Использовать `Stream<Event>` / `SingleLiveEvent` паттерн / Riverpod `ref.read(router)` — ViewModel не должен владеть BuildContext
- C) `GlobalKey<NavigatorState>` в ViewModel
- D) `ChangeNotifier.addListener()` только для навигации

<details>
<summary>Ответ</summary>

**B) События через Stream или результат метода; ViewModel без BuildContext**

```dart
// Паттерн 1: Stream событий
class LoginViewModel extends ChangeNotifier {
  final _events = StreamController<LoginEvent>.broadcast();
  Stream<LoginEvent> get events => _events.stream;

  Future<void> submit() async {
    try {
      await _authRepo.login(email, password);
      _events.add(LoginEvent.success); // View слушает и навигирует
    } catch (e) {
      _events.add(LoginEvent.error(e.toString()));
    }
  }

  @override
  void dispose() {
    _events.close();
    super.dispose();
  }
}

// View слушает:
@override
void initState() {
  super.initState();
  _vm.events.listen((event) {
    if (event == LoginEvent.success) router.go('/home');
  });
}
```

</details>

---

### Вопрос 9 🔴

Как управлять жизненным циклом ViewModel (инициализация при создании экрана, cleanup)?

- A) Только `initState()` в View
- B) `Riverpod autoDispose` автоматически удаляет ViewModel; `Notifier.build()` = init; `override dispose()` = cleanup
- C) ViewModel живёт всё время приложения
- D) `ViewModelLifecycle.attach(screen)`

<details>
<summary>Ответ</summary>

**B) `autoDispose` + `build()` для init + `dispose()` для cleanup**

```dart
// Автоматический lifecycle:
@Riverpod(keepAlive: false) // autoDispose = true
class DetailViewModel extends _$DetailViewModel {
  Timer? _refreshTimer;

  @override
  Future<ProductDetail> build(String productId) async {
    // Запустить периодическое обновление:
    _refreshTimer = Timer.periodic(const Duration(minutes: 5), (_) {
      ref.invalidateSelf();
    });

    // Cleanup при уничтожении:
    ref.onDispose(() {
      _refreshTimer?.cancel();
    });

    return ref.read(productRepositoryProvider).getDetail(productId);
  }
}
```

</details>

---

### Вопрос 10 🔴

Как реализовать пагинацию в MVVM?

- A) Загружать все данные сразу
- B) ViewModel хранит `_page`, `_hasMore`, `_isLoadingMore`; `loadNextPage()` при scrollToEnd; `PaginatedState` data class
- C) `ListView.builder` автоматически пагинирует
- D) Использовать `infinite_scroll` пакет без ViewModel

<details>
<summary>Ответ</summary>

**B) ViewModel управляет состоянием пагинации**

```dart
class ProductsViewModel extends Notifier<ProductsState> {
  @override
  ProductsState build() => const ProductsState();

  Future<void> loadNextPage() async {
    if (state.isLoadingMore || !state.hasMore) return;

    state = state.copyWith(isLoadingMore: true);

    try {
      final newItems = await ref.read(productRepositoryProvider)
          .getProducts(page: state.nextPage, pageSize: 20);

      state = state.copyWith(
        products: [...state.products, ...newItems],
        nextPage: state.nextPage + 1,
        hasMore: newItems.length == 20,
        isLoadingMore: false,
      );
    } catch (e) {
      state = state.copyWith(error: e.toString(), isLoadingMore: false);
    }
  }
}

// View — NotificationListener для scroll:
NotificationListener<ScrollEndNotification>(
  onNotification: (notification) {
    if (notification.metrics.extentAfter < 200) {
      ref.read(productsViewModelProvider.notifier).loadNextPage();
    }
    return false;
  },
  child: ListView.builder(itemBuilder: ...),
)
```

</details>
