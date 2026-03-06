# 13.4 Feature-first структура

## 1. Суть

**Feature-first** — организация кода по функциональным модулям («фичам»), а не по техническим слоям. Каждая фича — самодостаточная папка со своими data/domain/presentation слоями.

```
Layer-first (❌ плохо масштабируется):        Feature-first (✅ рекомендуется):
lib/                                           lib/
  data/                                          features/
    repositories/user_repo.dart                    auth/
    repositories/order_repo.dart                     data/ domain/ presentation/
    models/user.dart                               shop/
  domain/                                            data/ domain/ presentation/
    usecases/get_user.dart                         profile/
  presentation/                                      data/ domain/ presentation/
    screens/profile.dart                         core/
    screens/shop.dart                            main.dart
```

В layer-first при добавлении фичи нужно трогать 3+ папки. В feature-first — только одну.

---

## 2. Структура одной фичи

```
features/
  auth/
    data/
      datasources/
        auth_remote_data_source.dart
        auth_remote_data_source_impl.dart
      models/
        auth_token_dto.dart
        user_dto.dart
      repositories/
        auth_repository_impl.dart
    domain/
      entities/
        auth_token.dart
        user.dart
      repositories/
        auth_repository.dart         ← interface
      usecases/
        login_use_case.dart
        logout_use_case.dart
        register_use_case.dart
    presentation/
      screens/
        login_screen.dart
        register_screen.dart
      viewmodels/
        login_view_model.dart
      widgets/
        social_login_buttons.dart
        password_field.dart
    auth_di.dart                     ← DI регистрация
    auth.dart                        ← barrel-файл (опционально)
```

---

## 3. Core-модуль

Общий код, который используется всеми фичами:

```
core/
  di/
    di.dart                          ← главный setupDependencies()
  network/
    dio_factory.dart
    auth_interceptor.dart
  errors/
    app_exception.dart
    result.dart
  storage/
    secure_storage_service.dart
  utils/
    date_formatter.dart
    validators.dart
  theme/
    app_theme.dart
    app_colors.dart
  router/
    app_router.dart
  widgets/
    loading_overlay.dart
    error_widget.dart
```

---

## 4. Barrel-файлы

Barrel-файл (`auth.dart`) — публичное API фичи. Экспортирует только то, что нужно снаружи.

```dart
// features/auth/auth.dart
export 'domain/entities/user.dart';
export 'domain/repositories/auth_repository.dart';
export 'presentation/screens/login_screen.dart';
export 'presentation/screens/register_screen.dart';
export 'auth_di.dart';

// Используется в другой фиче:
import 'package:myapp/features/auth/auth.dart';
```

**Важно**: Не экспортируй внутренние модели (`UserDto`, `AuthRemoteDataSource`) — это детали реализации.

---

## 5. Взаимодействие между фичами

Фичи **не должны** напрямую зависеть друг от друга. Для связи используют:

```dart
// ❌ Плохо: прямая зависимость между фичами
import 'package:myapp/features/auth/data/repositories/auth_repository_impl.dart';

// ✅ Хорошо: через core или через DI
// В shop-фиче нужен текущий пользователь → инжектируй UserRepository из core
class CartViewModel {
  final UserRepository _userRepo; // интерфейс из core/domain
  final CartRepository _cartRepo;

  CartViewModel({required UserRepository userRepo, required CartRepository cartRepo})
      : _userRepo = userRepo, _cartRepo = cartRepo;
}
```

**Варианты взаимодействия:**

1. Через `core/` — общие сущности и интерфейсы.
2. Через DI — обе фичи знают об одном интерфейсе.
3. Через `EventBus` — асинхронные события.
4. Через Router — навигация с параметрами.

---

## 6. Реальный пример структуры среднего приложения

```
lib/
  features/
    auth/
      data/ domain/ presentation/
      auth_di.dart
    home/
      presentation/screens/home_screen.dart
    catalog/
      data/ domain/ presentation/
      catalog_di.dart
    cart/
      data/ domain/ presentation/
      cart_di.dart
    orders/
      data/ domain/ presentation/
      orders_di.dart
    profile/
      data/ domain/ presentation/
      profile_di.dart
  core/
    di/di.dart
    network/dio_factory.dart
    errors/result.dart
    router/app_router.dart
    theme/app_theme.dart
  main.dart
```

---

## 7. Масштабирование — мультипакетная структура

Для очень крупных проектов каждая фича становится отдельным Dart-пакетом:

```
packages/
  feature_auth/
    lib/
    pubspec.yaml
  feature_catalog/
    lib/
    pubspec.yaml
  core_network/
    lib/
    pubspec.yaml
app/
  lib/
    main.dart     ← собирает всё вместе
  pubspec.yaml   ← path-зависимости на packages/
```

```yaml
# app/pubspec.yaml
dependencies:
  feature_auth:
    path: ../packages/feature_auth
  feature_catalog:
    path: ../packages/feature_catalog
  core_network:
    path: ../packages/core_network
```

---

## 8. Под капотом

- Dart не имеет настоящих модулей — изоляция в feature-first **логическая**, через соглашения.
- Barrel-файлы помогают контролировать публичное API, но не ограничивают импорт технически.
- Мультипакетная структура даёт настоящую изоляцию и независимую компиляцию.

---

## 9. Типичные ошибки

| Ошибка                               | Причина                | Решение                                                        |
| ------------------------------------ | ---------------------- | -------------------------------------------------------------- |
| Фичи импортируют друг друга напрямую | Нет четкой границы     | Выносить общее в `core/`                                       |
| Слишком маленькие фичи               | Один экран = одна фича | Объединять связанные экраны (auth = login + register + forgot) |
| Слишком большие фичи                 | Нет разделения         | Дробить на sub-features                                        |
| Barrel-файл экспортирует всё         | Нет инкапсуляции       | Экспортировать только публичный API                            |

---

## 10. Рекомендации

1. **Фича = бизнес-возможность**: auth, payments, catalog — не технические понятия.
2. **`core/` для кроссфункционального кода**: DI, network, theme, router.
3. **Barrel-файлы** упрощают импорты, но не злоупотребляй ими.
4. **Командная разработка**: каждая команда отвечает за свою feature-папку.
5. **Мультипакет** — оправдан при 5+ разработчиках и 10+ фичах, иначе усложняет CI.
