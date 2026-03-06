# 7.5 Bloc / Cubit

## 1. Суть концепции

**BLoC (Business Logic Component)** — архитектурный паттерн, разделяющий UI и бизнес-логику. События (Events) → Bloc → Состояния (States). Поток событий → поток состояний.

**Cubit** — упрощённая версия Bloc без событий: методы напрямую вызывают `emit()`.

```yaml
dependencies:
  flutter_bloc: ^8.1.0
```

---

## 2. Cubit — простое управление состоянием

```dart
// Состояние
class CounterState {
  final int count;
  const CounterState(this.count);
}

// Cubit
class CounterCubit extends Cubit<CounterState> {
  CounterCubit() : super(const CounterState(0));

  void increment() => emit(CounterState(state.count + 1));
  void decrement() => emit(CounterState(state.count - 1));
  void reset() => emit(const CounterState(0));
}

// Использование
BlocProvider(
  create: (_) => CounterCubit(),
  child: CounterView(),
)

class CounterView extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<CounterCubit, CounterState>(
      builder: (context, state) {
        return Column(
          children: [
            Text('${state.count}'),
            Row(
              children: [
                IconButton(
                  icon: const Icon(Icons.remove),
                  onPressed: () => context.read<CounterCubit>().decrement(),
                ),
                IconButton(
                  icon: const Icon(Icons.add),
                  onPressed: () => context.read<CounterCubit>().increment(),
                ),
              ],
            ),
          ],
        );
      },
    );
  }
}
```

---

## 3. Bloc — Events и States

```dart
// События
abstract class AuthEvent {}
class LoginRequested extends AuthEvent {
  final String email;
  final String password;
  LoginRequested({required this.email, required this.password});
}
class LogoutRequested extends AuthEvent {}

// Состояния (используем sealed class для exhaustive switch)
sealed class AuthState {}
class AuthInitial extends AuthState {}
class AuthLoading extends AuthState {}
class AuthAuthenticated extends AuthState {
  final User user;
  AuthAuthenticated(this.user);
}
class AuthError extends AuthState {
  final String message;
  AuthError(this.message);
}

// Bloc
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  final AuthRepository _repo;

  AuthBloc(this._repo) : super(AuthInitial()) {
    on<LoginRequested>(_onLogin);
    on<LogoutRequested>(_onLogout);
  }

  Future<void> _onLogin(LoginRequested event, Emitter<AuthState> emit) async {
    emit(AuthLoading());
    try {
      final user = await _repo.login(event.email, event.password);
      emit(AuthAuthenticated(user));
    } catch (e) {
      emit(AuthError(e.toString()));
    }
  }

  Future<void> _onLogout(LogoutRequested event, Emitter<AuthState> emit) async {
    await _repo.logout();
    emit(AuthInitial());
  }
}
```

---

## 4. BlocProvider и BlocBuilder

```dart
// Предоставление Bloc
BlocProvider(
  create: (context) => AuthBloc(context.read<AuthRepository>()),
  child: const LoginScreen(),
)

// Множественные блоки
MultiBlocProvider(
  providers: [
    BlocProvider(create: (_) => AuthBloc(AuthRepository())),
    BlocProvider(create: (_) => CartCubit()),
  ],
  child: const MyApp(),
)

// Чтение состояния
class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<AuthBloc, AuthState>(
      builder: (context, state) {
        return switch (state) {
          AuthInitial() => const LoginForm(),
          AuthLoading() => const CircularProgressIndicator(),
          AuthAuthenticated(user: final u) => WelcomeScreen(user: u),
          AuthError(message: final msg) => Column(
            children: [
              const LoginForm(),
              Text(msg, style: const TextStyle(color: Colors.red)),
            ],
          ),
        };
      },
    );
  }
}
```

---

## 5. BlocListener — реакция на состояние без rebuild

```dart
// BlocListener — side effects: навигация, Snackbar, диалоги
BlocListener<AuthBloc, AuthState>(
  listener: (context, state) {
    if (state is AuthAuthenticated) {
      Navigator.pushReplacementNamed(context, '/home');
    }
    if (state is AuthError) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(state.message)),
      );
    }
  },
  child: const LoginForm(),
)

// BlocConsumer = BlocBuilder + BlocListener
BlocConsumer<AuthBloc, AuthState>(
  listener: (context, state) { /* side effects */ },
  builder: (context, state) { /* UI */ },
  // Опционально: buildWhen и listenWhen для фильтрации
  buildWhen: (prev, curr) => curr is! AuthLoading,
  listenWhen: (prev, curr) => curr is AuthAuthenticated || curr is AuthError,
)
```

---

## 6. Реальный пример: загрузка списка

```dart
// State
sealed class ProductsState {}
class ProductsInitial extends ProductsState {}
class ProductsLoading extends ProductsState {}
class ProductsLoaded extends ProductsState {
  final List<Product> products;
  ProductsLoaded(this.products);
}
class ProductsError extends ProductsState {
  final String message;
  ProductsError(this.message);
}

// Event
abstract class ProductsEvent {}
class ProductsLoadRequested extends ProductsEvent {}

// Cubit (упрощённо без Events)
class ProductsCubit extends Cubit<ProductsState> {
  final ProductRepository _repo;

  ProductsCubit(this._repo) : super(ProductsInitial());

  Future<void> load() async {
    emit(ProductsLoading());
    try {
      final products = await _repo.getAll();
      emit(ProductsLoaded(products));
    } catch (e) {
      emit(ProductsError(e.toString()));
    }
  }
}

// Screen
class ProductsScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => ProductsCubit(context.read<ProductRepository>())..load(),
      child: BlocBuilder<ProductsCubit, ProductsState>(
        builder: (context, state) {
          return switch (state) {
            ProductsInitial() || ProductsLoading() =>
              const Center(child: CircularProgressIndicator()),
            ProductsLoaded(products: final list) =>
              ListView.builder(
                itemCount: list.length,
                itemBuilder: (_, i) => ProductCard(product: list[i]),
              ),
            ProductsError(message: final msg) =>
              Center(child: Text('Ошибка: $msg')),
          };
        },
      ),
    );
  }
}
```

---

## 7. Типичные ошибки

| Ошибка                         | Проблема                      | Решение                                        |
| ------------------------------ | ----------------------------- | ---------------------------------------------- |
| Emit после закрытия Bloc       | Ошибка в рантайме             | Использовать `Emitter.onEach` или `isClosed`   |
| Навигация в `builder`          | Вызывается при каждом rebuild | Переносить в `listener`                        |
| Один Bloc для всего приложения | Сложно поддерживать           | Один Bloc = одна функциональность              |
| Состояния без `sealed class`   | Неполный switch               | Использовать `sealed` + исчерпывающий `switch` |

---

## 8. Cubit vs Bloc — когда что выбирать

|                  | Cubit          | Bloc                         |
| ---------------- | -------------- | ---------------------------- |
| Сложность        | Проще          | Сложнее                      |
| Трейсинг событий | Нет            | ✓                            |
| Дебагирование    | Сложнее        | Проще (каждое событие видно) |
| Логирование      | Ручное         | Автоматически                |
| Подходит для     | Простые экраны | Сложная логика               |

---

## 9. Практические рекомендации

1. **Cubit для простых случаев** (переключатели, счётчики, формы).
2. **Bloc для сложных кейсов** с несколькими событиями и сложными переходами.
3. **`sealed class` для состояний** — exhaustive switch без возможности пропустить состояние.
4. **`BlocListener` для side-effects** — навигация, SnackBar, диалоги.
5. **Один Bloc / Cubit = одна ответственность** — не "God Bloc" для всего.
6. **`BlocObserver`** для глобального логирования всех событий и состояний.
