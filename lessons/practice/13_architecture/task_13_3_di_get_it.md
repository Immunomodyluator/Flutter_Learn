# 13.3: Dependency Injection — get_it сервис-локатор

> Project: FitMenu | Глава 13 — Architecture

### 13.3: Настройка get_it для внедрения зависимостей

🎯 **Цель шага:** Настроить `get_it` как сервис-локатор для всех зависимостей FitMenu — DataSource, Repository, UseCase, ViewModel регистрируются один раз и инжектируются везде без `BuildContext`.

---

📝 **Техническое задание:**

**Зависимость:** `get_it: ^7.6.0`

Реализуй `lib/core/di/injection_container.dart`.

```dart
final getIt = GetIt.instance;

Future<void> configureDependencies() async {
  // --- Core ---
  getIt.registerSingleton<DioClient>(DioClient());
  getIt.registerSingleton<AppDatabase>(AppDatabase());

  // --- Data Sources ---
  getIt.registerLazySingleton<RecipeRemoteDataSource>(
    () => RecipeRemoteDataSourceImpl(getIt()),
  );
  getIt.registerLazySingleton<RecipeLocalDataSource>(
    () => RecipeCacheService(),
  );

  // --- Repositories ---
  getIt.registerLazySingleton<RecipeRepository>(
    () => RecipeRepositoryImpl(getIt(), getIt()),
  );

  // --- Use Cases ---
  getIt.registerFactory(() => SearchRecipesUseCase(getIt()));
  getIt.registerFactory(() => GetPopularRecipesUseCase(getIt()));

  // --- ViewModels ---
  getIt.registerFactory(() => RecipesViewModel(getIt()));
}
```

**Типы регистрации:**
- `registerSingleton` — создаётся сразу, живёт всё время
- `registerLazySingleton` — создаётся при первом обращении, потом переиспользуется
- `registerFactory` — новый экземпляр при каждом обращении (для ViewModel)

**Использование в ProviderScope:**
```dart
void main() async {
  await configureDependencies();
  runApp(
    ChangeNotifierProvider(
      create: (_) => getIt<RecipesViewModel>()..loadRecipes(),
      child: const FitMenuApp(),
    ),
  );
}
```

---

✅ **Критерии приёмки:**
- [ ] `configureDependencies()` вызывается в `main()` до `runApp`
- [ ] Синглтоны (DioClient, AppDatabase) зарегистрированы как `Singleton`
- [ ] ViewModels — `Factory` (новый экземпляр для каждого экрана)
- [ ] Repository — `LazySingleton` (один на всё приложение)
- [ ] `getIt<T>()` используется для получения зависимостей вне виджетов
- [ ] Нет прямого `import` конкретных реализаций из Presentation

---

💡 **Подсказка:** `get_it` — синхронный сервис-локатор. Для асинхронной инициализации используй `registerSingletonAsync` или инициализируй вне get_it и передавай готовый инстанс. `getIt.reset()` — полезно в тестах для сброса регистраций между тестами.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/di/injection_container.dart

import 'package:get_it/get_it.dart';

final getIt = GetIt.instance;

Future<void> configureDependencies() async {
  _registerCore();
  _registerDataSources();
  _registerRepositories();
  _registerUseCases();
  _registerViewModels();
}

void _registerCore() {
  getIt
    ..registerSingleton<DioClient>(DioClient())
    ..registerSingletonAsync<AppDatabase>(() async {
      final db = AppDatabase();
      return db;
    })
    ..registerLazySingleton<RecipeCacheService>(() => RecipeCacheService())
    ..registerLazySingleton<UserPreferencesService>(() => UserPreferencesService())
    ..registerLazySingleton<AnalyticsService>(() => AnalyticsService());
}

void _registerDataSources() {
  getIt
    ..registerLazySingleton<RecipeRemoteDataSource>(
      () => RecipeRemoteDataSourceImpl(getIt<DioClient>()),
    )
    ..registerLazySingleton<RecipeLocalDataSource>(
      () => getIt<RecipeCacheService>(),
    );
}

void _registerRepositories() {
  getIt.registerLazySingleton<RecipeRepository>(
    () => RecipeRepositoryImpl(
      getIt<RecipeRemoteDataSource>(),
      getIt<RecipeLocalDataSource>(),
    ),
  );
}

void _registerUseCases() {
  getIt
    ..registerFactory(() => SearchRecipesUseCase(getIt<RecipeRepository>()))
    ..registerFactory(() => GetPopularRecipesUseCase(getIt<RecipeRepository>()));
}

void _registerViewModels() {
  // Factory — новый экземпляр при каждом запросе
  getIt.registerFactory(
    () => RecipesViewModel(
      getIt<SearchRecipesUseCase>(),
      getIt<GetPopularRecipesUseCase>(),
    ),
  );
}
```

</details>
