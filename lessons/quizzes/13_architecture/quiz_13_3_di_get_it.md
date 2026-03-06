# Квиз: DI с get_it

**Тема:** 13.3 — Dependency Injection с get_it  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢
Что такое Service Locator и как зарегистрировать простую зависимость в get_it?

- A) Service Locator — это паттерн, при котором зависимости передаются через конструктор
- B) Service Locator — глобальный реестр, из которого объекты сами запрашивают свои зависимости; регистрация через `GetIt.instance.registerSingleton<T>()`
- C) get_it — это только для Flutter; в чистом Dart он не работает
- D) Service Locator и Dependency Injection — одно и то же

<details>
<summary>Ответ</summary>

**B) Глобальный реестр зависимостей; регистрация через `GetIt.instance`**

```dart
// pubspec.yaml
// dependencies:
//   get_it: ^7.6.0

import 'package:get_it/get_it.dart';

final getIt = GetIt.instance; // глобальная точка доступа

// Регистрация (обычно в main.dart или отдельном injection.dart)
void setupDependencies() {
  // Регистрация конкретного класса
  getIt.registerSingleton<ApiClient>(ApiClient());

  // Регистрация через абстракцию (интерфейс)
  getIt.registerSingleton<UserRepository>(
    UserRepositoryImpl(getIt<ApiClient>()),
  );

  getIt.registerSingleton<GetUserUseCase>(
    GetUserUseCase(getIt<UserRepository>()),
  );
}

void main() {
  setupDependencies();
  runApp(const MyApp());
}
```

**Отличие от Constructor Injection:** объект сам идёт за зависимостью (`getIt<T>()`), а не получает её снаружи. Это упрощает конфигурацию, но снижает явность зависимостей.

</details>

---

### Вопрос 2 🟢
В чём разница между `registerSingleton`, `registerLazySingleton` и `registerFactory`?

- A) Все три создают один экземпляр; разница только в названии
- B) `registerSingleton` создаёт объект при регистрации; `registerLazySingleton` — при первом запросе; `registerFactory` создаёт новый экземпляр каждый раз
- C) `registerFactory` создаёт синглтон; остальные — фабрики
- D) `registerLazySingleton` всегда пересоздаёт объект при каждом вызове

<details>
<summary>Ответ</summary>

**B) Каждый метод определяет время создания и количество экземпляров**

| Метод | Когда создаётся | Количество экземпляров |
|---|---|---|
| `registerSingleton` | При регистрации (eager) | Один на всё приложение |
| `registerLazySingleton` | При первом `getIt<T>()` | Один на всё приложение |
| `registerFactory` | При каждом `getIt<T>()` | Новый экземпляр каждый раз |

```dart
void setupDependencies() {
  // Создаётся сразу при вызове registerSingleton
  getIt.registerSingleton<AppConfig>(AppConfig.fromEnv());

  // Создаётся только при первом getIt<Database>()
  getIt.registerLazySingleton<Database>(() => Database.open());

  // Новый экземпляр при каждом getIt<FormController>()
  getIt.registerFactory<FormController>(() => FormController());
}

// Использование:
final config = getIt<AppConfig>();      // тот же объект
final db1 = getIt<Database>();          // создаётся при первом вызове
final db2 = getIt<Database>();          // тот же объект (db1 == db2)
final form1 = getIt<FormController>();  // новый объект
final form2 = getIt<FormController>();  // ещё один новый объект
```

**Практика:** `registerLazySingleton` — для сервисов и репозиториев; `registerFactory` — для объектов с состоянием (ViewModel, BLoC).

</details>

---

### Вопрос 3 🟢
Как получить зарегистрированную зависимость через get_it? Какие есть способы?

- A) Только через `GetIt.instance.get<T>()`
- B) Через `getIt<T>()`, `getIt.get<T>()` или вызов `getIt<T>` как функции — все варианты эквивалентны
- C) Только через конструктор с аннотацией `@inject`
- D) Через `GetIt.find<T>()` — единственный корректный способ

<details>
<summary>Ответ</summary>

**B) Все перечисленные варианты вызова эквивалентны**

```dart
final getIt = GetIt.instance;

// Все три способа идентичны:
final repo1 = getIt<UserRepository>();
final repo2 = getIt.get<UserRepository>();
final repo3 = getIt<UserRepository>(); // GetIt реализует call()

// Использование в виджете / ViewModel:
class UserViewModel extends ChangeNotifier {
  // Вариант 1: в теле класса
  final _useCase = getIt<GetUserUseCase>();

  // Вариант 2: через конструктор (предпочтительнее для тестирования)
  final GetUserUseCase _useCase2;
  UserViewModel() : _useCase2 = getIt<GetUserUseCase>();
}

// Проверка регистрации перед получением:
if (getIt.isRegistered<SomeService>()) {
  final service = getIt<SomeService>();
}

// Асинхронное получение (если зарегистрировано через registerSingletonAsync):
final asyncService = await getIt.getAsync<HeavyService>();
```

</details>

---

### Вопрос 4 🟡
Как использовать пакет `injectable` для автоматической кодогенерации DI-графа?

- A) `injectable` полностью заменяет get_it; они несовместимы
- B) `injectable` генерирует функцию `configureDependencies()` на основе аннотаций `@injectable`, `@singleton`, `@lazySingleton`
- C) `injectable` работает только с `riverpod`, не с `get_it`
- D) Аннотации `injectable` применяются к полям классов, а не к самим классам

<details>
<summary>Ответ</summary>

**B) injectable генерирует `configureDependencies()` по аннотациям**

```yaml
# pubspec.yaml
dependencies:
  get_it: ^7.6.0
  injectable: ^2.3.0

dev_dependencies:
  injectable_generator: ^2.4.0
  build_runner: ^2.4.0
```

```dart
// lib/injection.dart
import 'package:get_it/get_it.dart';
import 'package:injectable/injectable.dart';
import 'injection.config.dart'; // сгенерированный файл

final getIt = GetIt.instance;

@InjectableInit()
Future<void> configureDependencies() async =>
    getIt.init(); // вызывает сгенерированный код

// Аннотации на классах:
@injectable
class GetUserUseCase {
  final UserRepository _repo;
  GetUserUseCase(this._repo); // injectable разберётся с зависимостями
}

@singleton
class ApiClient {
  final String baseUrl;
  ApiClient(@Named('baseUrl') this.baseUrl);
}

@lazySingleton
class UserRepositoryImpl implements UserRepository {
  final ApiClient _client;
  UserRepositoryImpl(this._client);
}

// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await configureDependencies();
  runApp(const MyApp());
}

// Кодогенерация:
// dart run build_runner build --delete-conflicting-outputs
```

</details>

---

### Вопрос 5 🟡
Как зарегистрировать внешние зависимости (например, `Dio`, `SharedPreferences`) через `@module` в injectable?

- A) Внешние зависимости нельзя регистрировать через injectable — только ручная регистрация
- B) Создаётся абстрактный класс с аннотацией `@module`; методы с аннотациями возвращают внешние объекты
- C) `@module` используется только для тестовых двойников
- D) Внешние зависимости регистрируются через `@environment`

<details>
<summary>Ответ</summary>

**B) Абстрактный класс с `@module` предоставляет внешние зависимости**

```dart
import 'package:dio/dio.dart';
import 'package:injectable/injectable.dart';
import 'package:shared_preferences/shared_preferences.dart';

@module
abstract class AppModule {
  // Синглтон: создаётся один раз при запуске
  @singleton
  Dio get dio => Dio(BaseOptions(baseUrl: 'https://api.example.com'));

  // Lazy синглтон: создаётся при первом использовании
  @lazySingleton
  AppRouter get router => AppRouter();

  // Асинхронная регистрация
  @preResolve  // дождаться завершения до старта приложения
  @singleton
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();
}

// После кодогенерации injectable автоматически:
// 1. Вызовет эти методы для создания объектов
// 2. Передаст их в конструкторы классов, которые их требуют

@lazySingleton
class UserLocalDataSourceImpl implements UserLocalDataSource {
  final SharedPreferences _prefs; // будет передан из AppModule
  UserLocalDataSourceImpl(this._prefs);
}
```

</details>

---

### Вопрос 6 🟡
Как реализовать разные окружения (dev/prod/test) через `@Environment` в injectable?

- A) injectable не поддерживает разные окружения; нужно использовать `flutter_flavors`
- B) Используются именованные окружения через `@Environment('dev')` / `@prod`; при вызове `configureDependencies(env: 'prod')` регистрируются только нужные реализации
- C) Окружения задаются только через переменные среды ОС
- D) `@dev` и `@prod` — это аннотации Flutter SDK, не injectable

<details>
<summary>Ответ</summary>

**B) Именованные окружения через `@Environment` и параметр `env` в `configureDependencies`**

```dart
// Константы окружений
abstract class Env {
  static const dev = 'dev';
  static const prod = 'prod';
  static const test = 'test';
}

// Dev-реализация (только для 'dev' окружения)
@Environment(Env.dev)
@LazySingleton(as: UserRepository)
class FakeUserRepository implements UserRepository {
  @override
  Future<User?> getUserById(String id) async =>
      User(id: id, name: 'Test User', email: Email('test@dev.com'));
}

// Prod-реализация
@Environment(Env.prod)
@LazySingleton(as: UserRepository)
class UserRepositoryImpl implements UserRepository {
  final ApiClient _client;
  UserRepositoryImpl(this._client);
  // ... реальная реализация
}

// Регистрация для нужного окружения
void main() async {
  const environment = String.fromEnvironment(
    'ENV',
    defaultValue: Env.prod,
  );
  await configureDependencies(environment: environment);
  runApp(const MyApp());
}

// Запуск с нужным окружением:
// flutter run --dart-define=ENV=dev
```

</details>

---

### Вопрос 7 🟡
Как передать runtime-параметры в фабричную зависимость через get_it?

- A) get_it не поддерживает параметры — нужно использовать другой контейнер
- B) Через `registerFactoryParam<T, P1, P2>()` — до двух параметров передаются при вызове `getIt<T>(param1: value)`
- C) Параметры передаются только через глобальные переменные
- D) Нужно создать отдельную фабрику и зарегистрировать её как синглтон

<details>
<summary>Ответ</summary>

**B) `registerFactoryParam` принимает до двух типизированных параметров**

```dart
// Регистрация с параметром
void setupDependencies() {
  // P1 = String (userId), P2 = void (не нужен второй параметр)
  getIt.registerFactoryParam<UserViewModel, String, void>(
    (userId, _) => UserViewModel(
      userId: userId,
      useCase: getIt<GetUserUseCase>(),
    ),
  );

  // Два параметра: categoryId (String) + pageSize (int)
  getIt.registerFactoryParam<ProductListViewModel, String, int>(
    (categoryId, pageSize) => ProductListViewModel(
      categoryId: categoryId,
      pageSize: pageSize,
      repository: getIt<ProductRepository>(),
    ),
  );
}

// Получение с передачей параметров
class UserScreen extends StatelessWidget {
  final String userId;
  const UserScreen({required this.userId, super.key});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      // param1 передаётся в фабрику
      create: (_) => getIt<UserViewModel>(param1: userId),
      child: const UserScreenBody(),
    );
  }
}
```

В `injectable` это поддерживается через аннотацию `@factoryParam`:
```dart
@injectable
class UserViewModel {
  @factoryParam
  final String userId;
  final GetUserUseCase _useCase;

  UserViewModel(this.userId, this._useCase);
}
```

</details>

---

### Вопрос 8 🔴
Как тестировать код, использующий get_it, — как подменить зависимости в тестах?

- A) Код с get_it нетестируем — это главный недостаток Service Locator
- B) В тестах вызывается `getIt.reset()` для очистки, затем регистрируются мок-реализации; или использовать `allowReassignment = true` для перезаписи
- C) Тесты с get_it требуют запуска эмулятора Flutter
- D) Подмена зависимостей возможна только через переменные среды

<details>
<summary>Ответ</summary>

**B) `getIt.reset()` + регистрация моков перед каждым тестом**

```dart
import 'package:get_it/get_it.dart';
import 'package:mocktail/mocktail.dart';
import 'package:test/test.dart';

class MockUserRepository extends Mock implements UserRepository {}

void main() {
  late MockUserRepository mockRepo;

  setUp(() {
    // Полная очистка реестра перед каждым тестом
    getIt.reset();

    mockRepo = MockUserRepository();

    // Регистрируем мок вместо реальной реализации
    getIt.registerSingleton<UserRepository>(mockRepo);
    getIt.registerSingleton<GetUserUseCase>(
      GetUserUseCase(getIt<UserRepository>()),
    );
  });

  tearDown(() => getIt.reset());

  test('ViewModel загружает пользователя', () async {
    final user = User(id: '1', name: 'Иван', email: Email('i@test.com'));
    when(() => mockRepo.getUserById('1')).thenAnswer((_) async => user);

    final vm = UserViewModel(getIt<GetUserUseCase>());
    await vm.loadUser('1');

    expect(vm.user, equals(user));
  });
}

// Альтернатива: передавать зависимости в конструктор (лучшая практика)
// Тогда get_it не нужен в тестах вообще:
final vm = UserViewModel(GetUserUseCase(mockRepo)); // без getIt
```

**Рекомендация:** виджеты/ViewModel лучше получать зависимости через конструктор (Constructor Injection), get_it вызывать только на верхнем уровне (виджет-провайдер, `main.dart`).

</details>

---

### Вопрос 9 🔴
Service Locator vs Constructor Injection — в чём принципиальная разница и когда использовать каждый подход?

- A) Они идентичны; разница только синтаксическая
- B) Constructor Injection явно объявляет зависимости в сигнатуре — код более тестируем; Service Locator скрывает зависимости, но удобнее в корне композиции и виджетах
- C) Service Locator всегда лучше, так как не требует длинных конструкторов
- D) Constructor Injection применим только в Domain-слое, Service Locator — только в Presentation

<details>
<summary>Ответ</summary>

**B) Constructor Injection — явный контракт; Service Locator — удобен на точках входа**

```dart
// ❌ Service Locator внутри бизнес-логики — скрытые зависимости
class OrderService {
  Future<void> placeOrder(Order order) async {
    final repo = getIt<OrderRepository>(); // зависимость невидима снаружи
    final notifier = getIt<NotificationService>();
    await repo.save(order);
    await notifier.notify(order.userId);
  }
}
// Проблема: для теста нужно настраивать getIt, невозможно понять зависимости
// по сигнатуре конструктора.

// ✅ Constructor Injection — явные зависимости
class OrderService {
  final OrderRepository _repo;
  final NotificationService _notifier;

  // Все зависимости видны в конструкторе
  OrderService(this._repo, this._notifier);

  Future<void> placeOrder(Order order) async {
    await _repo.save(order);
    await _notifier.notify(order.userId);
  }
}
// Тест: OrderService(MockOrderRepository(), MockNotificationService())

// ✅ Service Locator уместен в точке входа (виджет/экран)
class OrderScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider(
      // Здесь getIt уместен — это корень композиции для виджета
      create: (_) => getIt<OrderViewModel>(),
      child: const OrderScreenBody(),
    );
  }
}
```

**Правило:** используйте Constructor Injection для Domain и Data; get_it/Service Locator — только как «корень композиции» в виджетах и `main.dart`.

</details>

---

### Вопрос 10 🔴
Как реализовать асинхронную инициализацию зависимости через `registerSingletonAsync` и гарантировать готовность перед стартом UI?

- A) `registerSingletonAsync` не существует — асинхронные зависимости нужно инициализировать вручную
- B) Использовать `registerSingletonAsync` с `Future`-фабрикой, затем `await getIt.allReady()` перед `runApp`
- C) Асинхронные зависимости не могут быть синглтонами
- D) `allReady()` блокирует поток UI — использовать нельзя

<details>
<summary>Ответ</summary>

**B) `registerSingletonAsync` + `await getIt.allReady()` перед запуском приложения**

```dart
// lib/injection.dart
Future<void> configureDependencies() async {
  // Синхронные зависимости
  getIt.registerSingleton<AppConfig>(AppConfig.fromEnv());

  // Асинхронная регистрация: база данных требует async-инициализации
  getIt.registerSingletonAsync<AppDatabase>(() async {
    final db = AppDatabase();
    await db.initialize(); // выполняется асинхронно
    return db;
  });

  // SharedPreferences — классический пример асинхронной зависимости
  getIt.registerSingletonAsync<SharedPreferences>(
    () => SharedPreferences.getInstance(),
  );

  // Зависимость, которая ждёт готовности другой
  getIt.registerSingletonWithDependencies<UserLocalDataSource>(
    () => UserLocalDataSourceImpl(getIt<SharedPreferences>()),
    dependsOn: [SharedPreferences], // будет создан после SharedPreferences
  );

  // Ждём инициализации ВСЕХ async-синглтонов
  await getIt.allReady();
}

// main.dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await configureDependencies(); // всё готово до старта UI
  runApp(const MyApp());
}

// Альтернатива: FutureBuilder для ленивой инициализации с splash-экраном
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: FutureBuilder(
        future: getIt.allReady(), // ждём готовности в UI
        builder: (ctx, snap) {
          if (!snap.hasData) return const SplashScreen();
          return const HomeScreen();
        },
      ),
    );
  }
}
```

</details>
