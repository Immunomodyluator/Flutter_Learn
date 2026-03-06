# Квиз: Feature-first архитектура

**Тема:** 13.4 — Feature-first архитектура  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢
В чём принципиальное отличие feature-first от layer-first структуры папок?

- A) Feature-first и layer-first — одно и то же, только с разными названиями папок
- B) Layer-first группирует по техническим слоям (`data/`, `domain/`, `presentation/`); feature-first группирует по бизнес-функциям (`auth/`, `profile/`, `orders/`)
- C) Feature-first подходит только для маленьких проектов
- D) Layer-first — это современный подход; feature-first устарел

<details>
<summary>Ответ</summary>

**B) Layer-first — по слоям; feature-first — по бизнес-функциям**

```
# Layer-first (технические слои на верхнем уровне):
lib/
├── data/
│   ├── repositories/
│   │   ├── user_repository_impl.dart
│   │   └── product_repository_impl.dart
│   └── datasources/
├── domain/
│   ├── entities/
│   ├── repositories/
│   └── usecases/
└── presentation/
    ├── screens/
    └── widgets/

# Feature-first (бизнес-фичи на верхнем уровне):
lib/
├── features/
│   ├── auth/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   ├── profile/
│   │   ├── data/
│   │   ├── domain/
│   │   └── presentation/
│   └── orders/
│       ├── data/
│       ├── domain/
│       └── presentation/
└── core/  # shared: сеть, хранилище, утилиты
```

**Преимущество feature-first:** все файлы одной функциональности находятся рядом — легче навигировать, удалять и переиспользовать фичу целиком.

</details>

---

### Вопрос 2 🟢
Что такое barrel export (index.dart) и зачем он нужен в feature-first архитектуре?

- A) `index.dart` — точка входа Flutter-приложения, аналог `main.dart`
- B) Barrel export — файл, реэкспортирующий публичное API фичи; позволяет скрыть внутренние детали и сократить импорты
- C) `index.dart` автоматически генерируется Flutter SDK
- D) Barrel export замедляет компиляцию — его лучше не использовать

<details>
<summary>Ответ</summary>

**B) Barrel export реэкспортирует публичный API и скрывает внутренние детали**

```dart
// lib/features/auth/index.dart — публичное API фичи auth
export 'presentation/screens/login_screen.dart';
export 'presentation/screens/register_screen.dart';
export 'domain/entities/auth_user.dart';
// НЕ экспортируем внутренние детали:
// ❌ export 'data/datasources/auth_remote_datasource.dart';
// ❌ export 'data/models/auth_response_dto.dart';

// lib/features/orders/index.dart
export 'presentation/screens/orders_screen.dart';
export 'domain/entities/order.dart';
export 'domain/usecases/place_order_usecase.dart';
```

```dart
// БЕЗ barrel export — импорты длинные и хрупкие:
import 'package:my_app/features/auth/presentation/screens/login_screen.dart';
import 'package:my_app/features/auth/domain/entities/auth_user.dart';

// С barrel export — чисто и устойчиво к рефакторингу:
import 'package:my_app/features/auth/index.dart';
// или просто:
import 'package:my_app/features/auth/index.dart';
```

При переименовании/перемещении внутренних файлов достаточно обновить только `index.dart`.

</details>

---

### Вопрос 3 🟢
Как выглядит типичная структура feature-first проекта с несколькими фичами?

- A) Все файлы в одной папке `lib/`, разделены только по типу виджета
- B) Проект делится на `features/` (бизнес-фичи), `core/` (общий код) и `app/` (корень приложения, маршрутизация)
- C) Feature-first требует использования Flutter Flavors
- D) Каждая фича — отдельное Flutter-приложение

<details>
<summary>Ответ</summary>

**B) `features/` + `core/` + `app/` — стандартная структура**

```
lib/
├── app/                        # Корень приложения
│   ├── app.dart                # MaterialApp / корневой виджет
│   ├── router.dart             # Маршрутизация (go_router / auto_route)
│   └── di/
│       └── injection.dart      # DI-инициализация
│
├── core/                       # Общий код (не привязан к фиче)
│   ├── network/
│   │   └── dio_client.dart
│   ├── storage/
│   │   └── local_storage.dart
│   ├── error/
│   │   └── failures.dart
│   └── widgets/                # Переиспользуемые UI-компоненты
│       ├── app_button.dart
│       └── loading_indicator.dart
│
└── features/
    ├── auth/
    │   ├── data/
    │   │   ├── datasources/
    │   │   ├── models/
    │   │   └── repositories/
    │   ├── domain/
    │   │   ├── entities/
    │   │   ├── repositories/
    │   │   └── usecases/
    │   ├── presentation/
    │   │   ├── bloc/
    │   │   ├── screens/
    │   │   └── widgets/
    │   └── index.dart           # Barrel export
    │
    ├── profile/
    │   └── ...
    │
    └── orders/
        └── ...
```

</details>

---

### Вопрос 4 🟡
Как обеспечить изоляцию между фичами — что можно и что нельзя импортировать?

- A) Фичи могут свободно импортировать друг друга — это нормально
- B) Фичи не должны напрямую импортировать внутренние файлы других фич; взаимодействие — только через `index.dart` (публичное API) или через `core/`
- C) Полная изоляция невозможна без вынесения каждой фичи в отдельный пакет
- D) Изоляция достигается только через интерфейсы в `core/`

<details>
<summary>Ответ</summary>

**B) Только через публичное API (`index.dart`) или через `core/`**

```dart
// ❌ Нарушение изоляции: orders импортирует внутренний файл auth
import 'package:my_app/features/auth/data/models/auth_token_dto.dart';
import 'package:my_app/features/auth/presentation/bloc/auth_bloc.dart';

// ✅ Правильно: через публичный barrel export
import 'package:my_app/features/auth/index.dart'; // только то, что экспортировано

// ✅ Через core/ — для действительно общего кода
import 'package:my_app/core/auth/current_user.dart'; // если нужно в нескольких фичах
```

```dart
// Пример: фича orders использует данные пользователя из auth
// НЕ через прямой импорт внутренностей auth, а через абстракцию в core:

// core/auth/current_user_provider.dart
abstract class CurrentUserProvider {
  AuthUser? getCurrentUser();
}

// features/auth/... реализует CurrentUserProvider
// features/orders/... зависит от CurrentUserProvider из core
class PlaceOrderUseCase {
  final CurrentUserProvider _currentUser; // зависимость из core, не из auth
  final OrderRepository _orders;

  PlaceOrderUseCase(this._currentUser, this._orders);
}
```

Правило: **зависимость от фичи — через её `index.dart`; общая логика — в `core/`**.

</details>

---

### Вопрос 5 🟡
Как организовать shared/common код в feature-first архитектуре, чтобы он не превратился в «мусорную корзину»?

- A) Весь общий код нужно класть в `lib/utils/` без дополнительной структуры
- B) `core/` разделяется по ответственности (network, storage, widgets, error, theme); туда попадает только то, что используется минимум в двух фичах
- C) Общий код запрещён — всё должно быть внутри фич
- D) `core/` — синоним `utils/`; структура внутри не нужна

<details>
<summary>Ответ</summary>

**B) `core/` разбит по ответственности, попадает только переиспользуемое**

```
lib/core/
├── network/
│   ├── dio_client.dart         # Настроенный Dio
│   ├── network_info.dart       # Проверка соединения
│   └── api_exception.dart      # Сетевые исключения
│
├── storage/
│   ├── secure_storage.dart     # flutter_secure_storage обёртка
│   └── local_cache.dart        # SharedPreferences обёртка
│
├── error/
│   ├── failures.dart           # Базовые классы Failure
│   └── exceptions.dart         # Базовые исключения
│
├── theme/
│   ├── app_theme.dart
│   ├── app_colors.dart
│   └── app_text_styles.dart
│
├── widgets/                    # UI-компоненты без бизнес-логики
│   ├── app_button.dart
│   ├── app_text_field.dart
│   └── error_widget.dart
│
└── extensions/                 # Dart-расширения
    ├── string_extensions.dart
    └── datetime_extensions.dart
```

**Правило «Rule of Three»:** класть в `core/` только то, что уже реально используется в 2+ фичах. Не делать `core/` «на вырост».

</details>

---

### Вопрос 6 🟡
Когда стоит выносить фичу в отдельный Dart-пакет, а когда достаточно папки в монорепо?

- A) Всегда нужно выносить в пакет — это единственный способ обеспечить изоляцию
- B) Отдельный пакет оправдан при: переиспользовании в нескольких приложениях, необходимости независимого версионирования, большой команде с чёткими границами ответственности
- C) Выносить в пакет нужно только UI-компоненты
- D) Dart-пакеты не поддерживают зависимости между собой

<details>
<summary>Ответ</summary>

**B) Отдельный пакет оправдан при переиспользовании, версионировании или большой команде**

```
# Монорепо с несколькими пакетами (melos):
my_app/
├── apps/
│   ├── mobile/          # Flutter-приложение
│   └── web/             # Flutter Web
│
└── packages/
    ├── core/            # Общий Dart-пакет
    │   └── pubspec.yaml
    ├── feature_auth/    # Фича как пакет
    │   └── pubspec.yaml
    ├── feature_orders/
    │   └── pubspec.yaml
    └── design_system/   # UI kit как пакет
        └── pubspec.yaml
```

```yaml
# packages/feature_auth/pubspec.yaml
name: feature_auth
version: 1.0.0

dependencies:
  core:
    path: ../core   # локальная зависимость
  get_it: ^7.6.0
```

**Когда НЕ стоит выносить в пакет:**
- Небольшая команда (1–3 человека)
- Фича не переиспользуется в других приложениях
- Overhead от управления зависимостями превышает пользу

</details>

---

### Вопрос 7 🟡
Как совместить feature-first структуру с Clean Architecture внутри каждой фичи?

- A) Они несовместимы — нужно выбрать одно
- B) Feature-first — это структура папок верхнего уровня; внутри каждой фичи применяется Clean Architecture (data/domain/presentation) со своими слоями
- C) Clean Architecture предполагает layer-first — внутри фич слоёв нет
- D) Внутри фичи достаточно двух папок: `ui/` и `logic/`

<details>
<summary>Ответ</summary>

**B) Feature-first — верхний уровень; Clean Architecture — внутри каждой фичи**

```
lib/features/orders/
│
├── data/
│   ├── datasources/
│   │   ├── orders_remote_datasource.dart
│   │   └── orders_local_datasource.dart
│   ├── models/
│   │   └── order_dto.dart
│   └── repositories/
│       └── orders_repository_impl.dart   # реализация
│
├── domain/
│   ├── entities/
│   │   └── order.dart                    # чистый Dart
│   ├── repositories/
│   │   └── orders_repository.dart        # абстракция
│   └── usecases/
│       ├── get_orders_usecase.dart
│       └── place_order_usecase.dart
│
├── presentation/
│   ├── bloc/
│   │   ├── orders_bloc.dart
│   │   ├── orders_event.dart
│   │   └── orders_state.dart
│   ├── screens/
│   │   ├── orders_list_screen.dart
│   │   └── order_detail_screen.dart
│   └── widgets/
│       └── order_card.dart
│
└── index.dart   # экспортирует только публичное API
```

Зависимости внутри фичи соблюдают Dependency Rule Clean Architecture: `presentation → domain ← data`.

</details>

---

### Вопрос 8 🔴
Как управлять зависимостями между фичами, чтобы не создать циклические зависимости?

- A) Циклические зависимости между фичами — норма; Dart их разрешает
- B) Общие контракты выносятся в `core/`; фичи зависят от `core/`, но не друг от друга напрямую; для передачи данных используются callback/события
- C) Зависимости между фичами передаются через глобальные переменные
- D) Фичи должны знать о реализациях друг друга — это единственный способ взаимодействия

<details>
<summary>Ответ</summary>

**B) Контракты в `core/`, событийное взаимодействие, без прямых зависимостей фич друг на друга**

```
# Допустимый граф зависимостей:
feature_auth   ──▶  core
feature_orders ──▶  core
feature_profile ──▶ core

# ❌ Недопустимо:
feature_orders ──▶ feature_auth  (прямая зависимость фичи на фичу)
feature_auth ──▶ feature_orders  (циклическая зависимость)
```

```dart
// core/contracts/auth_contract.dart — контракт в core
abstract class AuthStateProvider {
  Stream<AuthStatus> get authStatusStream;
  AuthUser? get currentUser;
}

// feature_auth предоставляет реализацию (через DI)
@singleton
class AuthBloc extends Bloc<AuthEvent, AuthState>
    implements AuthStateProvider { ... }

// feature_orders использует контракт из core, не зная об AuthBloc
class OrdersBloc extends Bloc<OrdersEvent, OrdersState> {
  final AuthStateProvider _authProvider; // из core
  OrdersBloc(this._authProvider, ...) {
    _authProvider.authStatusStream.listen((status) {
      if (status == AuthStatus.unauthenticated) add(const ClearOrders());
    });
  }
}
```

**Паттерны для межфичевого взаимодействия:**
1. **Shared контракты в `core/`** — абстракции, реализуемые одной фичей и потребляемые другой
2. **Event bus** — pub/sub через `Stream` или пакет `event_bus`
3. **Callback через Navigator** — передача результата при навигации

</details>

---

### Вопрос 9 🔴
Как организовать навигацию в модульной feature-first архитектуре?

- A) Каждый экран знает о всех остальных экранах и напрямую вызывает `Navigator.push`
- B) Каждая фича объявляет свои маршруты (RouteFactory/GoRoute); центральный роутер в `app/` собирает их вместе; фичи не знают о навигации между собой
- C) Навигация в модульной архитектуре невозможна — нужно монолитное приложение
- D) Все маршруты объявляются внутри `core/`, фичи импортируют их оттуда

<details>
<summary>Ответ</summary>

**B) Фичи объявляют свои маршруты; центральный роутер собирает граф**

```dart
// features/auth/auth_routes.dart
import 'package:go_router/go_router.dart';
import 'index.dart';

class AuthRoutes {
  static const login = '/login';
  static const register = '/register';

  static List<GoRoute> get routes => [
    GoRoute(
      path: login,
      builder: (_, __) => const LoginScreen(),
    ),
    GoRoute(
      path: register,
      builder: (_, __) => const RegisterScreen(),
    ),
  ];
}

// features/orders/orders_routes.dart
class OrdersRoutes {
  static const list = '/orders';
  static const detail = '/orders/:id';

  static List<GoRoute> get routes => [
    GoRoute(
      path: list,
      builder: (_, __) => const OrdersScreen(),
      routes: [
        GoRoute(
          path: ':id',
          builder: (_, state) => OrderDetailScreen(
            orderId: state.pathParameters['id']!,
          ),
        ),
      ],
    ),
  ];
}

// app/router.dart — центральный роутер собирает маршруты всех фич
final appRouter = GoRouter(
  initialLocation: AuthRoutes.login,
  routes: [
    ...AuthRoutes.routes,
    ...OrdersRoutes.routes,
    ...ProfileRoutes.routes,
  ],
  redirect: (context, state) {
    final isAuthenticated = getIt<AuthStateProvider>().currentUser != null;
    if (!isAuthenticated && !state.matchedLocation.startsWith('/login')) {
      return AuthRoutes.login;
    }
    return null;
  },
);
```

**Ключевой принцип:** фичи не знают пути других фич — навигация между фичами происходит через callback или событие, которое обрабатывает роутер.

</details>

---

### Вопрос 10 🔴
Как управлять несколькими Dart-пакетами в монорепо с помощью Melos?

- A) Melos — это пакет для управления состоянием, не для монорепо
- B) Melos позволяет запускать команды (test, build, pub get) сразу во всех пакетах монорепо через `melos.yaml`; поддерживает версионирование и публикацию
- C) Melos работает только с pub.dev-пакетами, не с локальными
- D) Для Flutter монорепо Melos не нужен — достаточно `pubspec.yaml` с `path` зависимостями

<details>
<summary>Ответ</summary>

**B) Melos — инструмент для управления командами и версионированием в монорепо**

```yaml
# melos.yaml (корень монорепо)
name: my_app_workspace

packages:
  - apps/**       # Flutter-приложения
  - packages/**   # Dart/Flutter пакеты

command:
  version:
    # Автоматическое версионирование по CHANGELOG
    workspaceChangelog: true

scripts:
  # Запустить тесты во всех пакетах
  test:
    run: flutter test
    exec:
      concurrency: 4  # параллельно в 4 пакетах

  # Анализ кода во всех пакетах
  analyze:
    run: flutter analyze
    exec:
      concurrency: 6

  # Кодогенерация (build_runner) только в пакетах с build_runner
  codegen:
    run: dart run build_runner build --delete-conflicting-outputs
    packageFilters:
      dependsOn: build_runner  # только пакеты с этой зависимостью
```

```bash
# Установка Melos
dart pub global activate melos

# Инициализация — связывает пакеты локально (pub get во всех)
melos bootstrap

# Запуск скриптов
melos run test        # тесты во всех пакетах
melos run analyze     # анализ во всех пакетах
melos run codegen     # кодогенерация только где нужно

# Фильтрация: запуск только в конкретном пакете
melos run test --scope="feature_auth"

# Версионирование (Conventional Commits → CHANGELOG → pubspec version)
melos version
```

**Преимущества Melos перед ручным управлением:**
- `melos bootstrap` настраивает локальные `path`-зависимости автоматически
- Параллельный запуск команд экономит время CI
- Версионирование по Conventional Commits с автогенерацией CHANGELOG

</details>
