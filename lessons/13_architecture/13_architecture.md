# 13. Архитектура Flutter-приложений

## 1. Зачем нужна архитектура?

Без архитектуры код превращается в «спагетти»: логика смешивается с UI, тесты невозможны, функции дублируются. Хорошая архитектура обеспечивает:

- **Разделение ответственности** — каждый класс делает одно дело.
- **Тестируемость** — логика изолирована от виджетов.
- **Масштабируемость** — новые фичи не ломают существующие.
- **Читаемость** — любой разработчик понимает, где что лежит.

---

## 2. Эволюция архитектур в Flutter

```
Виджет с setState → BLoC/Provider → MVVM → Clean Architecture
     (просто)         (реактив)     (слои)  (максимум изоляции)
```

| Подход                      | Когда подходит               |
| --------------------------- | ---------------------------- |
| `setState`                  | Прототипы, учебные проекты   |
| `Provider + ChangeNotifier` | Средние приложения           |
| **MVVM**                    | Средние и крупные приложения |
| **Clean Architecture**      | Крупные командные проекты    |

---

## 3. Ключевые слои (Clean Architecture)

```
┌───────────────────────────────────┐
│  Presentation (UI + ViewModel)    │  ← виджеты, VM, Bloc/Cubit
├───────────────────────────────────┤
│  Domain (бизнес-логика)           │  ← UseCases, Entities, Repository interface
├───────────────────────────────────┤
│  Data (данные)                    │  ← Repository impl, DataSource, DTO
└───────────────────────────────────┘
```

**Правило зависимости**: каждый слой знает только о слое ниже. Domain не знает ни о Flutter, ни об HTTP.

---

## 4. Структуры проекта

### Feature-first (рекомендуется)

```
lib/
  features/
    auth/
      data/
        auth_repository_impl.dart
        auth_remote_data_source.dart
      domain/
        auth_repository.dart       ← абстракция
        login_use_case.dart
        user_entity.dart
      presentation/
        login_screen.dart
        login_view_model.dart
    home/
      ...
  core/
    di/                            ← зависимости
    network/                       ← Dio, interceptors
    errors/                        ← Result, AppException
    utils/
  main.dart
```

### Layer-first (классический)

```
lib/
  data/
    repositories/
    datasources/
    models/
  domain/
    repositories/
    usecases/
    entities/
  presentation/
    screens/
    viewmodels/
    widgets/
```

Feature-first предпочтительнее при командной разработке — фича живёт в одной папке.

---

## 5. Dependency Injection — `get_it`

```yaml
dependencies:
  get_it: ^7.7.0
  injectable: ^2.4.0 # опционально: кодогенерация DI
```

```dart
// di.dart — регистрация зависимостей
final getIt = GetIt.instance;

void setupDependencies() {
  // Синглтоны (создаются один раз)
  getIt.registerSingleton<Dio>(createDio());
  getIt.registerSingleton<AuthRepository>(AuthRepositoryImpl(getIt()));

  // Lazy singleton (создаётся при первом обращении)
  getIt.registerLazySingleton<UserRepository>(() => UserRepositoryImpl(getIt()));

  // Factory (новый экземпляр каждый раз)
  getIt.registerFactory<LoginViewModel>(() => LoginViewModel(getIt()));
}

// Использование:
final repo = getIt<AuthRepository>();
```

---

## 6. Обзор тем раздела

| Файл                         | Тема                                                   |
| ---------------------------- | ------------------------------------------------------ |
| `13_1_mvvm.md`               | MVVM: ViewModel + ChangeNotifier / Riverpod            |
| `13_2_clean_architecture.md` | Clean Architecture: слои, UseCases, Repository pattern |
| `13_3_di_get_it.md`          | Dependency Injection: get_it, injectable               |
| `13_4_feature_first.md`      | Feature-first структура, Barrel-файлы, модульность     |

---

## 7. Рекомендации

1. **Начни с MVVM** — это основа, которую легко расширить до Clean Architecture.
2. **Feature-first** в команде — каждая команда работает над своей папкой.
3. **Не усложняй** ради усложнения — для маленьких проектов Clean Architecture избыточна.
4. **Domain-слой чистый** — никаких импортов Flutter (только `dart:core`).
5. **DI с первого дня** — `get_it` не требует кодогенерации и прост в рефакторинге.
