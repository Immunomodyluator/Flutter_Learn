# 7.5: BLoC/Cubit — поиск рецептов

> Project: FitMenu | Глава 7 — State Management

### 7.5: SearchCubit для поиска рецептов с debounce

🎯 **Цель шага:** Реализовать поиск рецептов через `Cubit` из пакета `flutter_bloc` — с дебаунсингом запроса, состояниями загрузки/результата/ошибки и чистой архитектурой событий.

---

📝 **Техническое задание:**

**Зависимость:**
```yaml
dependencies:
  flutter_bloc: ^8.1.0
```

Реализуй `lib/features/recipes/bloc/search_cubit.dart`.

**Состояния (sealed class / абстрактный класс):**
```dart
abstract class SearchState {}
class SearchInitial extends SearchState {}
class SearchLoading extends SearchState {}
class SearchSuccess extends SearchState {
  SearchSuccess(this.results);
  final List<Recipe> results;
}
class SearchEmpty extends SearchState {}
class SearchError extends SearchState {
  SearchError(this.message);
  final String message;
}
```

**SearchCubit:**
```dart
class SearchCubit extends Cubit<SearchState> {
  SearchCubit(this._repository) : super(SearchInitial());

  final RecipeRepository _repository;
  Timer? _debounce;

  void search(String query) {
    _debounce?.cancel();
    if (query.isEmpty) { emit(SearchInitial()); return; }

    _debounce = Timer(const Duration(milliseconds: 400), () async {
      emit(SearchLoading());
      try {
        final results = await _repository.searchRecipes(query);
        emit(results.isEmpty ? SearchEmpty() : SearchSuccess(results));
      } catch (e) {
        emit(SearchError(e.toString()));
      }
    });
  }

  @override
  Future<void> close() {
    _debounce?.cancel();
    return super.close();
  }
}
```

**SearchScreen:**
- `TextField` с `onChanged: (q) => cubit.search(q)`
- `BlocBuilder<SearchCubit, SearchState>` для отображения результатов
- Разные виджеты для каждого состояния:
  - `SearchInitial` → "Начни вводить название рецепта"
  - `SearchLoading` → `CircularProgressIndicator`
  - `SearchSuccess` → `ListView` с результатами
  - `SearchEmpty` → "Ничего не найдено"
  - `SearchError` → сообщение об ошибке

---

✅ **Критерии приёмки:**
- [ ] Debounce 400мс работает (запрос не отправляется при каждом символе)
- [ ] `_debounce?.cancel()` вызывается в `close()` — нет утечки памяти
- [ ] Все 5 состояний отображаются корректно в UI
- [ ] `BlocProvider` передаёт `SearchCubit` в дерево виджетов
- [ ] `BlocBuilder` перестраивает только список, не весь экран
- [ ] Пустой запрос возвращает в состояние `SearchInitial`

---

💡 **Подсказка:** `BlocProvider` и `BlocBuilder` — основные виджеты. `BlocProvider.value` используй когда cubit создан выше (уже существует). `context.read<SearchCubit>()` — доступ к cubit в обработчиках. `Timer` из `dart:async` отлично подходит для debounce.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/features/recipes/bloc/search_cubit.dart

import 'dart:async';
import 'package:flutter_bloc/flutter_bloc.dart';
// import Recipe, RecipeRepository

// --- Состояния ---
sealed class SearchState {}

class SearchInitial extends SearchState {}
class SearchLoading extends SearchState {}

class SearchSuccess extends SearchState {
  SearchSuccess(this.results);
  final List<Recipe> results;
}

class SearchEmpty extends SearchState {}

class SearchError extends SearchState {
  SearchError(this.message);
  final String message;
}

// --- Cubit ---
class SearchCubit extends Cubit<SearchState> {
  SearchCubit(this._repository) : super(SearchInitial());

  final RecipeRepository _repository;
  Timer? _debounce;

  void search(String query) {
    _debounce?.cancel();

    if (query.trim().isEmpty) {
      emit(SearchInitial());
      return;
    }

    _debounce = Timer(const Duration(milliseconds: 400), () async {
      emit(SearchLoading());
      try {
        final results = await _repository.searchRecipes(query.trim());
        if (results.isEmpty) {
          emit(SearchEmpty());
        } else {
          emit(SearchSuccess(results));
        }
      } catch (e) {
        emit(SearchError('Ошибка поиска: ${e.toString()}'));
      }
    });
  }

  void clear() => emit(SearchInitial());

  @override
  Future<void> close() {
    _debounce?.cancel();
    return super.close();
  }
}
```

```dart
// lib/features/recipes/screens/search_screen.dart

import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

class SearchScreen extends StatelessWidget {
  const SearchScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => SearchCubit(context.read<RecipeRepository>()),
      child: const _SearchView(),
    );
  }
}

class _SearchView extends StatelessWidget {
  const _SearchView();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: TextField(
          autofocus: true,
          decoration: const InputDecoration(
            hintText: 'Поиск рецептов...',
            border: InputBorder.none,
          ),
          onChanged: (q) => context.read<SearchCubit>().search(q),
        ),
      ),
      body: BlocBuilder<SearchCubit, SearchState>(
        builder: (context, state) => switch (state) {
          SearchInitial() => const Center(
              child: Text('Начни вводить название рецепта 🔍'),
            ),
          SearchLoading() => const Center(child: CircularProgressIndicator()),
          SearchEmpty()   => const Center(child: Text('Ничего не найдено 😔')),
          SearchError(message: final msg) => Center(
              child: Text(msg, style: const TextStyle(color: Colors.red)),
            ),
          SearchSuccess(results: final list) => ListView.builder(
              itemCount: list.length,
              itemBuilder: (context, index) => ListTile(
                title: Text(list[index].title),
                subtitle: Text('${list[index].calories} ккал'),
                leading: const Icon(Icons.restaurant),
              ),
            ),
        },
      ),
    );
  }
}
```

</details>
