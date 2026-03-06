# 8.4: Обработка ошибок — Result паттерн

> Project: FitMenu | Глава 8 — Networking

### 8.4: Result<T> и sealed AppError для надёжной обработки ошибок

🎯 **Цель шага:** Заменить бросание исключений на явный `Result<T>` паттерн в слое репозитория — ошибки становятся частью типа возвращаемого значения и проверяются компилятором.

---

📝 **Техническое задание:**

Реализуй `lib/core/utils/result.dart`.

**Result (sealed class Dart 3):**
```dart
sealed class Result<T> { const Result(); }

final class Success<T> extends Result<T> {
  const Success(this.value);
  final T value;
}

final class Failure<T> extends Result<T> {
  const Failure(this.error);
  final AppError error;
}
```

**AppError (sealed иерархия):**
```dart
sealed class AppError { const AppError(this.message); final String message; }
final class NetworkError    extends AppError { const NetworkError(super.m, {required this.statusCode}); final int statusCode; }
final class NotFoundError   extends AppError { const NotFoundError()  : super('Не найдено'); }
final class UnauthorizedError extends AppError { const UnauthorizedError() : super('Требуется авторизация'); }
final class TimeoutError    extends AppError { const TimeoutError()   : super('Таймаут запроса'); }
final class UnknownError    extends AppError { const UnknownError(super.m); }
```

**Extension методы:**
```dart
extension ResultX<T> on Result<T> {
  bool get isSuccess => this is Success<T>;
  T? get valueOrNull => switch (this) { Success(value: final v) => v, _ => null };
  R when<R>({required R Function(T) success, required R Function(AppError) failure}) =>
      switch (this) { Success(value: final v) => success(v), Failure(error: final e) => failure(e) };
}
```

**RecipeRepository с Result:**
```dart
Future<Result<List<Recipe>>> searchRecipes(String query) async {
  try {
    final dtos = await _api.searchRecipes(query);
    return Success(dtos.toDomain());
  } on ApiException catch (e) { return Failure(_mapError(e)); }
  catch (e) { return Failure(UnknownError(e.toString())); }
}
```

**UI обработка:**
```dart
final result = await repository.searchRecipes(query);
result.when(
  success: (recipes) => emit(SearchSuccess(recipes)),
  failure: (err) => emit(SearchError(err.message)),
);
```

---

✅ **Критерии приёмки:**
- [ ] `Result<T>` — `sealed class` с `Success` и `Failure`
- [ ] `AppError` — sealed иерархия, компилятор проверяет исчерпываемость `switch`
- [ ] `RecipeRepository` возвращает `Result`, не бросает исключений
- [ ] `when()` extension обрабатывает оба случая
- [ ] `ApiException` конвертируется в `AppError` в репозитории
- [ ] Pattern matching `switch (result) { case Success... }` компилируется

---

💡 **Подсказка:** `sealed class` + `final class` в Dart 3 обеспечивают exhaustive switch. Пакет `fpdart` предоставляет готовые `Either<L, R>` если нужна функциональная библиотека. `Result` паттерн исключает забытую обработку ошибок — компилятор предупредит.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/utils/result.dart

sealed class Result<T> { const Result(); }

final class Success<T> extends Result<T> {
  const Success(this.value);
  final T value;
}

final class Failure<T> extends Result<T> {
  const Failure(this.error);
  final AppError error;
}

extension ResultX<T> on Result<T> {
  bool get isSuccess => this is Success<T>;
  bool get isFailure => this is Failure<T>;

  T? get valueOrNull => switch (this) { Success(value: final v) => v, _ => null };
  AppError? get errorOrNull => switch (this) { Failure(error: final e) => e, _ => null };

  R when<R>({
    required R Function(T value) success,
    required R Function(AppError error) failure,
  }) =>
      switch (this) {
        Success(value: final v) => success(v),
        Failure(error: final e) => failure(e),
      };

  Result<R> map<R>(R Function(T) f) => switch (this) {
        Success(value: final v) => Success(f(v)),
        Failure(error: final e) => Failure(e),
      };
}

// Иерархия ошибок
sealed class AppError {
  const AppError(this.message);
  final String message;
}

final class NetworkError extends AppError {
  const NetworkError(super.message, {required this.statusCode});
  final int statusCode;
}

final class NotFoundError   extends AppError { const NotFoundError()      : super('Ресурс не найден'); }
final class UnauthorizedError extends AppError { const UnauthorizedError() : super('Требуется авторизация'); }
final class TimeoutError    extends AppError { const TimeoutError()        : super('Таймаут запроса'); }
final class UnknownError    extends AppError { const UnknownError(super.message); }
```

```dart
// Репозиторий
class RecipeRepository {
  const RecipeRepository(this._api);
  final RecipeApiService _api;

  Future<Result<List<Recipe>>> searchRecipes(String query) async {
    try {
      final dtos = await _api.searchRecipes(query);
      return Success(dtos.map((d) => d.toDomain()).toList());
    } on ApiException catch (e) {
      return Failure(_mapError(e));
    } catch (e) {
      return Failure(UnknownError(e.toString()));
    }
  }

  AppError _mapError(ApiException e) => switch (e.statusCode) {
        401 => const UnauthorizedError(),
        404 => const NotFoundError(),
        408 => const TimeoutError(),
        _   => NetworkError(e.message, statusCode: e.statusCode),
      };
}
```

</details>
