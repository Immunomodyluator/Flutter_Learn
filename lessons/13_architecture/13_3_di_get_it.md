# 13.3 Dependency Injection — get_it

## 1. Суть

**Dependency Injection (DI)** — паттерн, при котором зависимости (репозитории, сервисы) не создаются внутри классов, а передаются извне. `get_it` — простой service locator для Flutter/Dart без кодогенерации.

```yaml
dependencies:
  get_it: ^7.7.0

  # Опционально: кодогенерация через injectable
  injectable: ^2.4.0

dev_dependencies:
  injectable_generator: ^2.4.0
  build_runner: ^2.4.0
```

---

## 2. Регистрация зависимостей

```dart
// di.dart
import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

void setupDependencies() {

  // --- Singleton: один экземпляр на всё время жизни приложения ---
  getIt.registerSingleton<Dio>(createDio());

  // --- Lazy Singleton: создаётся при первом обращении ---
  getIt.registerLazySingleton<UserRemoteDataSource>(
    () => UserRemoteDataSourceImpl(getIt<Dio>()),
  );

  getIt.registerLazySingleton<UserRepository>(
    () => UserRepositoryImpl(
      remote: getIt<UserRemoteDataSource>(),
      local: getIt<UserLocalDataSource>(),
    ),
  );

  // --- Factory: новый экземпляр при каждом обращении ---
  getIt.registerFactory<ProfileViewModel>(
    () => ProfileViewModel(getIt<UserRepository>()),
  );

  // --- Factory с параметрами ---
  getIt.registerFactoryParam<OrderViewModel, String, void>(
    (orderId, _) => OrderViewModel(
      orderId: orderId,
      repository: getIt<OrderRepository>(),
    ),
  );
}
```

```dart
// main.dart
void main() {
  setupDependencies();
  runApp(const MyApp());
}
```

---

## 3. Получение зависимостей

```dart
// Получить singleton / lazy singleton
final userRepo = getIt<UserRepository>();

// Создать Factory (новый экземпляр)
final vm = getIt<ProfileViewModel>();

// Factory с параметром
final orderVm = getIt<OrderViewModel>(param1: 'order_123');

// Проверить регистрацию
final isRegistered = getIt.isRegistered<UserRepository>();

// Получить асинхронно (если синглтон регистрируется асинхронно)
await getIt.getAsync<Database>();
```

---

## 4. Асинхронная инициализация

```dart
Future<void> setupDependencies() async {
  // Регистрация асинхронного синглтона
  getIt.registerSingletonAsync<Database>(() async {
    final db = await openDatabase('app.db');
    return AppDatabase(db);
  });

  getIt.registerSingletonAsync<SharedPreferences>(
    () => SharedPreferences.getInstance(),
  );

  // Дождаться завершения всех асинхронных синглтонов
  await getIt.allReady();
}

// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await setupDependencies();
  runApp(const MyApp());
}
```

---

## 5. injectable — кодогенерация DI

```dart
// di.dart
import 'package:injectable/injectable.dart';
import 'di.config.dart'; // генерируется

final getIt = GetIt.instance;

@InjectableInit()
Future<void> setupDependencies() => getIt.init();
```

```dart
// Аннотации injectable
@singleton
class Dio { ... }

@lazySingleton
class UserRemoteDataSourceImpl implements UserRemoteDataSource {
  UserRemoteDataSourceImpl(Dio dio); // инжектируется автоматически
}

@injectable
class ProfileViewModel extends ChangeNotifier {
  ProfileViewModel(UserRepository repository);
}

// Среда (dev/prod)
@Environment(Environment.prod)
@LazySingleton(as: UserRemoteDataSource)
class UserRemoteDataSourceImpl implements UserRemoteDataSource { ... }

@Environment(Environment.dev)
@LazySingleton(as: UserRemoteDataSource)
class FakeUserRemoteDataSource implements UserRemoteDataSource { ... }
```

```bash
# Генерация конфига
dart run build_runner build --delete-conflicting-outputs
```

---

## 6. Разделение DI по feature-модулям

```dart
// features/user/user_di.dart
void registerUserDependencies() {
  getIt
    ..registerLazySingleton<UserRemoteDataSource>(
        () => UserRemoteDataSourceImpl(getIt()))
    ..registerLazySingleton<UserRepository>(
        () => UserRepositoryImpl(remote: getIt(), local: getIt()))
    ..registerFactory<ProfileViewModel>(
        () => ProfileViewModel(GetUserUseCase(getIt())));
}

// features/auth/auth_di.dart
void registerAuthDependencies() {
  getIt
    ..registerLazySingleton<AuthRepository>(
        () => AuthRepositoryImpl(getIt()))
    ..registerFactory<LoginViewModel>(
        () => LoginViewModel(getIt()));
}

// di.dart — главный
void setupDependencies() {
  _registerCore();
  registerUserDependencies();
  registerAuthDependencies();
  // ...
}
```

---

## 7. DI + Provider/Riverpod

```dart
// С Provider:
ChangeNotifierProvider(
  create: (_) => getIt<ProfileViewModel>(),
  child: const ProfileScreen(),
)

// С Riverpod:
final profileViewModelProvider = ChangeNotifierProvider(
  (ref) => getIt<ProfileViewModel>(),
);

// Или через Riverpod без get_it (предпочтительно с Riverpod):
final userRepositoryProvider = Provider<UserRepository>(
  (ref) => UserRepositoryImpl(remote: ref.read(userRemoteDataSourceProvider)),
);
```

---

## 8. Под капотом

- `get_it` — это `Map<Type, dynamic>` под капотом (хэш-мап по типам).
- `Singleton` регистрирует экземпляр немедленно.
- `LazySingleton` сохраняет фабрику-функцию, вызывает её при первом `get()`.
- `Factory` вызывает фабрику каждый раз при `get()`.

---

## 9. Типичные ошибки

| Ошибка                                   | Причина                                 | Решение                                                    |
| ---------------------------------------- | --------------------------------------- | ---------------------------------------------------------- |
| `StateError: get_it: ... not registered` | Не зарегистрировано                     | Вызвать `setupDependencies()` до `runApp()`                |
| Circular dependency                      | A зависит от B, B от A                  | Использовать `Provider` или разбить на более мелкие классы |
| Factory для Repository                   | Создаётся новый экземпляр с новым кешем | Репозитории — LazySingleton                                |
| Mock в продакшн-сборке                   | Нет разделения среды                    | Использовать `@Environment` или `if (kDebugMode)`          |

---

## 10. Рекомендации

1. **Singleton/LazySingleton** — для `Repository`, `DataSource`, `Dio`, `Database`.
2. **Factory** — для `ViewModel` (у каждого экрана свой экземпляр).
3. **Feature-модули DI** — каждая feature регистрирует свои зависимости отдельно.
4. **`injectable`** для крупных проектов — исключает ошибки ручной регистрации.
5. **Тесты** — регистрируй mock-реализации в `setUp()`, сбрасывай `getIt.reset()` в `tearDown()`.
