# 1.5: Null Safety

> **Проект:** FitMenu — трекер питания и рецептов
> **Глава:** 1 — Введение в Dart

---

### 1.5: Null Safety

🎯 **Цель шага:** Сделать модели FitMenu безопасными относительно `null` — разграничить обязательные и опциональные поля в `Recipe` и `UserProfile`, применить `?.`, `??`, `!` и `late` осознанно.

📝 **Техническое задание:**

Обнови `Recipe` и создай `UserProfile`:

1. `Recipe.imageUrl` — `String?` (фото может отсутствовать), геттер `String get safeImageUrl` возвращает placeholder через `??`
2. `Recipe.description` — `String?`, геттер `String get safeDescription` с дефолтом
3. `UserProfile` с `late String displayName` (заполняется после Firebase Auth)
4. Функция `String safeImageUrl(Recipe? recipe)` — цепочка `?.` + `??` на nullable Recipe
5. Функция `Future<UserProfile?> fetchProfile(String? uid)` — возвращает `null` если uid не передан; обработай результат без `!`

✅ **Критерии приёмки:**
- [ ] `Recipe` компилируется с `String?` полями без ошибок
- [ ] `safeImageUrl(null)` не крашит приложение — возвращает placeholder
- [ ] `late displayName` заполняется **до** первого чтения (нет `LateInitializationError`)
- [ ] Нет ни одного `!` без комментария «почему здесь null невозможен»
- [ ] Условие `if (profile != null)` используется вместо `profile!`

💡 **Подсказка:** Если видишь `!` — спроси себя: «Что произойдёт если здесь всё-таки null?». Если ответ «крэш в проде» — замени на `if` или `??`.

<details>
<summary>🛠 Эталонное решение (Нажми, чтобы проверить себя)</summary>

```dart
const String kPlaceholderImage =
    'https://fitmenufood.app/images/placeholder.jpg';

class Recipe {
  final String id;
  final String title;
  final double calories;
  final String? imageUrl;    // опциональное — фото может не быть
  final String? description; // опциональное — краткое описание

  const Recipe({
    required this.id,
    required this.title,
    required this.calories,
    this.imageUrl,
    this.description,
  });

  // Никогда не вернёт null — безопасно для UI
  String get safeImageUrl => imageUrl ?? kPlaceholderImage;
  String get safeDescription => description ?? 'Описание отсутствует';
}

class UserProfile {
  final String uid;
  late String displayName; // устанавливается после Firebase Auth
  String? photoUrl;        // аватар может не быть

  UserProfile({required this.uid});

  String get visibleName =>
      displayName.isEmpty ? 'Пользователь' : displayName;
}

/// Null-безопасная функция — принимает nullable Recipe
String safeImageUrl(Recipe? recipe) =>
    recipe?.imageUrl ?? kPlaceholderImage;

/// Симулирует загрузку профиля
Future<UserProfile?> fetchProfile(String? uid) async {
  await Future.delayed(const Duration(milliseconds: 200));
  if (uid == null) return null; // нет авторизации

  final profile = UserProfile(uid: uid);
  profile.displayName = 'Иван'; // late инициализация до первого чтения
  return profile;
}

Future<void> main() async {
  // Recipe без картинки и описания
  const recipe = Recipe(id: '1', title: 'Овсянка', calories: 320);
  print(recipe.safeImageUrl);     // placeholder URL
  print(recipe.safeDescription);  // дефолт

  // nullable Recipe через ?.
  Recipe? maybeRecipe;
  print(safeImageUrl(maybeRecipe)); // placeholder, не крэш

  // Цепочка ?. на nullable
  final len = maybeRecipe?.title.length ?? 0;
  print('Длина: $len'); // 0

  // Обработка nullable Future без !
  final profile = await fetchProfile('uid_abc');
  if (profile != null) {
    print('👤 ${profile.visibleName}');
    print('Фото: ${profile.photoUrl?.length ?? 0} символов в URL');
  } else {
    print('Пользователь не авторизован');
  }

  // Незалогиненный пользователь
  final anon = await fetchProfile(null);
  print('Анонимный uid: ${anon?.uid ?? 'отсутствует'}');
}
```

</details>
