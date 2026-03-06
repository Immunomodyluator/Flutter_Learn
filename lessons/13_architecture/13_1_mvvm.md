# 13.1 MVVM в Flutter

## 1. Суть

**MVVM** (Model-View-ViewModel) — паттерн, в котором:

- **Model** — данные и бизнес-логика (Repository, UseCase, Entity).
- **View** — виджет, отображает состояние и передаёт события.
- **ViewModel** — посредник: держит состояние UI, вызывает логику, не знает о виджетах.

```
View  ──(события)──→  ViewModel  ──→  Repository/UseCase
View  ←─(состояние)─  ViewModel  ←──  Repository/UseCase
```

---

## 2. ViewModel с ChangeNotifier

```dart
// --- user_entity.dart ---
class User {
  final String id;
  final String name;
  final String email;
  const User({required this.id, required this.name, required this.email});
}

// --- user_repository.dart (абстракция) ---
abstract class UserRepository {
  Future<User> fetchUser(String id);
}

// --- profile_view_model.dart ---
class ProfileViewModel extends ChangeNotifier {
  final UserRepository _repository;
  ProfileViewModel(this._repository);

  User? _user;
  bool _isLoading = false;
  String? _error;

  User? get user => _user;
  bool get isLoading => _isLoading;
  String? get error => _error;

  Future<void> loadUser(String id) async {
    _isLoading = true;
    _error = null;
    notifyListeners();

    try {
      _user = await _repository.fetchUser(id);
    } catch (e) {
      _error = 'Не удалось загрузить профиль: $e';
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }

  void clearError() {
    _error = null;
    notifyListeners();
  }
}
```

```dart
// --- profile_screen.dart ---
class ProfileScreen extends StatelessWidget {
  final String userId;
  const ProfileScreen({super.key, required this.userId});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      create: (_) => ProfileViewModel(getIt<UserRepository>())..loadUser(userId),
      child: const _ProfileView(),
    );
  }
}

class _ProfileView extends StatelessWidget {
  const _ProfileView();

  @override
  Widget build(BuildContext context) {
    final vm = context.watch<ProfileViewModel>();

    if (vm.isLoading) return const Center(child: CircularProgressIndicator());
    if (vm.error != null) {
      return Center(
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            Text(vm.error!),
            ElevatedButton(
              onPressed: () => context.read<ProfileViewModel>().loadUser('123'),
              child: const Text('Повторить'),
            ),
          ],
        ),
      );
    }

    final user = vm.user!;
    return Column(
      children: [
        Text(user.name, style: Theme.of(context).textTheme.headlineMedium),
        Text(user.email),
      ],
    );
  }
}
```

---

## 3. ViewModel с Riverpod

```dart
// --- providers.dart ---
@riverpod
class ProfileViewModel extends _$ProfileViewModel {
  @override
  Future<User> build(String userId) async {
    return ref.read(userRepositoryProvider).fetchUser(userId);
  }

  Future<void> reload(String userId) async {
    state = const AsyncValue.loading();
    state = await AsyncValue.guard(
      () => ref.read(userRepositoryProvider).fetchUser(userId),
    );
  }
}

// --- profile_screen.dart ---
class ProfileScreen extends ConsumerWidget {
  final String userId;
  const ProfileScreen({super.key, required this.userId});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final profileAsync = ref.watch(profileViewModelProvider(userId));

    return profileAsync.when(
      loading: () => const Center(child: CircularProgressIndicator()),
      error: (e, _) => Center(child: Text('Ошибка: $e')),
      data: (user) => Column(
        children: [
          Text(user.name),
          Text(user.email),
        ],
      ),
    );
  }
}
```

---

## 4. Состояние ViewModel — паттерн UiState

```dart
// Вместо трёх отдельных полей — одно sealed-состояние
sealed class ProfileState {
  const ProfileState();
}

class ProfileInitial extends ProfileState {}
class ProfileLoading extends ProfileState {}
class ProfileSuccess extends ProfileState {
  final User user;
  const ProfileSuccess(this.user);
}
class ProfileError extends ProfileState {
  final String message;
  const ProfileError(this.message);
}

// ViewModel
class ProfileViewModel extends ChangeNotifier {
  final UserRepository _repository;
  ProfileViewModel(this._repository);

  ProfileState _state = ProfileInitial();
  ProfileState get state => _state;

  void _emit(ProfileState s) {
    _state = s;
    notifyListeners();
  }

  Future<void> loadUser(String id) async {
    _emit(ProfileLoading());
    try {
      final user = await _repository.fetchUser(id);
      _emit(ProfileSuccess(user));
    } catch (e) {
      _emit(ProfileError('Ошибка загрузки: $e'));
    }
  }
}

// Виджет
Widget build(BuildContext context) {
  final vm = context.watch<ProfileViewModel>();
  return switch (vm.state) {
    ProfileInitial() => const SizedBox(),
    ProfileLoading() => const CircularProgressIndicator(),
    ProfileSuccess(:final user) => Text(user.name),
    ProfileError(:final message) => Text(message, style: TextStyle(color: Colors.red)),
  };
}
```

---

## 5. Под капотом

- `ChangeNotifier.notifyListeners()` обходит всех подписчиков и перестраивает виджеты, читающие этот provider.
- `context.watch` → `Selector` под капотом: слушает все изменения ViewModel.
- `context.read` → единоразовый доступ без подписки (используй в обработчиках событий).
- `context.select` → подписка только на часть состояния: экономит ребилды.

---

## 6. Типичные ошибки

| Ошибка                                | Причина                             | Решение                                         |
| ------------------------------------- | ----------------------------------- | ----------------------------------------------- |
| Логика в виджете                      | Бизнес-логика прямо в `build()`     | Перенести в ViewModel                           |
| `context.read` в `build()`            | Нет реакции на изменения            | Использовать `context.watch`                    |
| ViewModel знает о виджетах            | Импорт `flutter/material.dart` в VM | Убрать Flutter-зависимости из VM                |
| `notifyListeners()` после `dispose()` | Вызов после удаления виджета        | Проверять `mounted` или использовать Future-try |

---

## 7. Рекомендации

1. **ViewModel — чистый Dart**: никаких `BuildContext`, `Widget`, импортов Flutter.
2. **Один ViewModel на экран** — не создавай ViewModel на каждый мелкий виджет.
3. **Sealed состояния** (`ProfileState`) лучше трёх флагов — exhaustive switch невозможно забыть.
4. **`context.select`** для точечных подписок — снижает количество ребилдов.
5. **Riverpod** — предпочтительный выбор для MVVM в крупных проектах: compile-safe, тестируемо.
