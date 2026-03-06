# 1.5 Null Safety

## 1. Суть концепции

Относится к: **система типов, null safety, компилятор**.

Sound Null Safety — одна из главных особенностей современного Dart (введена в Dart 2.12). Суть: **компилятор гарантирует**, что non-nullable переменная никогда не будет `null`. Это устраняет целый класс ошибок типа `Null check operator used on a null value` ещё на этапе разработки.

Ключевая идея: тип `String` и тип `String?` — **два разных типа**. Если ты объявил `String name`, компилятор не позволит присвоить `null`.

---

## 2. Как используется во Flutter

- Поля виджетов: `required` vs nullable optional параметры
- Данные из API: `String? email` — почта может отсутствовать
- `BuildContext` — всегда non-nullable, Flutter гарантирует это
- `TextEditingController.text` — всегда `String`, не `String?`
- FutureBuilder / StreamBuilder — `snapshot.data` возвращает `T?`

---

## 3. Синтаксис и базовый пример

```dart
// Non-nullable — не может быть null
String name = 'Flutter';
int count = 0;
// name = null; // ОШИБКА КОМПИЛЯЦИИ

// Nullable — может быть null, добавляем ?
String? nickname;         // null по умолчанию
int? optionalAge;

// Проверка на null
if (nickname != null) {
  print(nickname.length); // Dart знает: здесь nickname точно не null (smart cast)
}

// Оператор ?? — значение по умолчанию если null
String display = nickname ?? 'No nickname';

// Оператор ?. — безопасный вызов метода
int? len = nickname?.length; // null если nickname == null

// Оператор ! — утверждение "я знаю, что это не null"
// ОПАСНО: выбросит исключение если всё-таки null
String definitelyNotNull = nickname!;

// late — non-nullable поле, инициализируемое позже
late String userId;
// ... где-то позже:
userId = fetchUserId();
print(userId); // безопасно если инициализировано
```

### Операторы быстро

```dart
String? x = maybeNull();

x ?? 'default'        // если x == null → 'default'
x?.length             // если x == null → null, иначе x.length
x!                    // утверждаем что не null (crash если null)
x ??= 'default'       // присвоить 'default' если x == null
```

---

## 4. Реальный пример из Flutter

```dart
class UserProfile {
  final String id;          // всегда есть
  final String displayName; // всегда есть
  final String? avatarUrl;  // опционально — пользователь мог не загрузить фото
  final String? bio;        // опционально

  const UserProfile({
    required this.id,
    required this.displayName,
    this.avatarUrl,  // nullable — по умолчанию null, не нужен required
    this.bio,
  });
}

class ProfileWidget extends StatelessWidget {
  final UserProfile profile;

  const ProfileWidget({super.key, required this.profile});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        // Аватар: если null — показываем placeholder
        CircleAvatar(
          backgroundImage: profile.avatarUrl != null
              ? NetworkImage(profile.avatarUrl!) // ! безопасен: мы проверили выше
              : null,
          child: profile.avatarUrl == null
              ? const Icon(Icons.person)
              : null,
        ),

        Text(profile.displayName),

        // Bio: показываем только если есть
        if (profile.bio != null)
          Text(
            profile.bio!, // ! безопасен внутри if (bio != null)
            style: const TextStyle(color: Colors.grey),
          ),

        // Или через ?. и ?? — короче
        Text(profile.bio ?? 'Bio not provided'),
      ],
    );
  }
}
```

---

## 5. Что происходит под капотом

### Sound vs Unsound

**Sound** null safety означает: если компилятор говорит что что-то non-nullable — это **гарантировано** на 100%. Нет ситуации где non-nullable вдруг окажется null в runtime (в отличие от TypeScript где null safety лишь "best effort").

### Smart cast (flow analysis)

```dart
String? value = getValue();

// Здесь компилятор НЕ знает, что value не null
// value.length — ошибка компиляции

if (value == null) return;

// Здесь компилятор ЗНАЕТ: value точно String (не null)
print(value.length); // ок, без !
```

Dart анализирует поток выполнения и "сужает" тип после проверки. Это называется **promotion**.

### `late` и null safety

`late` — обещание компилятору "я инициализирую это до первого чтения". Компилятор верит, но в runtime добавляется проверка:

```dart
late String name;
print(name); // LateInitializationError: Field 'name' has not been initialized
```

Используется для полей `State`, которые инициализируются в `initState()`.

---

## 6. Типичные ошибки

| Ошибка                                         | Почему возникает                                       | Правильно                                         |
| ---------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------- |
| `user!.name` без проверки                      | `!` без уверенности — crash в runtime                  | Проверить через `if (user != null)` или `?.`      |
| `late` поле без инициализации до использования | `LateInitializationError`                              | Инициализировать в `initState()` гарантированно   |
| `snapshot.data!` в FutureBuilder               | `data` null пока Future не завершён                    | `snapshot.hasData ? snapshot.data! : placeholder` |
| Поле nullable там где оно всегда есть          | `required String? name` — противоречие                 | Если всегда есть — `required String name`         |
| Цепочка `?.` скрывает null                     | `a?.b?.c?.d` — в конце получаем null, непонятно откуда | Разбить на явные проверки для диагностики         |

---

## 7. Практические рекомендации

1. **`required` для обязательных параметров** — делает API виджета явным и самодокументирующимся.
2. **Минимизируй использование `!`** — каждый `!` это потенциальный crash. Предпочитай `??`, `?.`, `if (x != null)`.
3. **Nullable поле в модели = осознанное решение** — если `String? email` значит бизнес-логика допускает отсутствие email. Документируй это.
4. **`late` для AnimationController** — стандартный паттерн: `late AnimationController _controller;` в State, инициализация в `initState`.
5. **Promotion вместо повторных проверок** — после `if (x == null) return;` компилятор промоутит тип, дальше `!` не нужен.
6. **`??=` для ленивой инициализации** — `_cache ??= expensiveCompute()` выполняет compute только один раз.
7. **Не используй nullable там где значение всегда присутствует** — это ложная гибкость и лишние проверки везде.
