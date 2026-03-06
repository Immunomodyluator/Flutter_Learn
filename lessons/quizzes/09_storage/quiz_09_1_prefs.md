# Квиз: SharedPreferences и локальное хранилище

**Тема:** 09.1 — SharedPreferences  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Что такое `SharedPreferences` и для чего используется?

- A) SQL база данных для Flutter
- B) Key-value хранилище для небольших примитивных данных (настройки, флаги, токены)
- C) Файловое хранилище для медиа
- D) Зашифрованное хранилище для паролей

<details>
<summary>Ответ</summary>

**B) Key-value хранилище для примитивных данных**

`SharedPreferences` хранит: `String`, `int`, `double`, `bool`, `List<String>`. Персистентно между сессиями приложения. На Android — `SharedPreferences` XML файл; на iOS — `NSUserDefaults`. Не для конфиденциальных данных (не зашифровано).

</details>

---

### Вопрос 2 🟢

Как сохранить и получить строку через `SharedPreferences`?

- A) `SharedPreferences.save('key', 'value')` / `SharedPreferences.load('key')`
- B) `prefs.setString('key', 'value')` / `prefs.getString('key')`
- C) `prefs['key'] = 'value'` / `prefs['key']`
- D) `prefs.put('key', 'value')` / `prefs.get('key')`

<details>
<summary>Ответ</summary>

**B) `prefs.setString(key, value)` / `prefs.getString(key)`**

```dart
// Инициализация:
final prefs = await SharedPreferences.getInstance();

// Сохранить:
await prefs.setString('username', 'Alice');
await prefs.setBool('isDark', true);
await prefs.setInt('counter', 42);

// Получить:
final name = prefs.getString('username'); // String? (null если нет)
final isDark = prefs.getBool('isDark') ?? false; // с default
```

</details>

---

### Вопрос 3 🟢

Как удалить значение из `SharedPreferences`?

- A) `prefs.setNull('key')`
- B) `prefs.remove('key')` или `prefs.clear()` (все данные)
- C) `prefs.delete('key')`
- D) Удалить вручную — нет API удаления

<details>
<summary>Ответ</summary>

**B) `prefs.remove('key')` или `prefs.clear()`**

```dart
await prefs.remove('username'); // удалить конкретный ключ
await prefs.clear(); // удалить всё (используй осторожно!)

// Проверить наличие:
final hasKey = prefs.containsKey('username');
```

</details>

---

### Вопрос 4 🟡

Как хранить объект (класс) в `SharedPreferences`?

- A) Нельзя — только примитивы
- B) Сериализовать в JSON строку: `prefs.setString('user', jsonEncode(user.toJson()))` и `User.fromJson(jsonDecode(prefs.getString('user')!))`
- C) `prefs.setObject('user', user)`
- D) Через `objectBox` адаптер

<details>
<summary>Ответ</summary>

**B) Сериализовать в JSON строку**

```dart
// Сохранить:
await prefs.setString('user', jsonEncode(user.toJson()));

// Получить:
final userJson = prefs.getString('user');
if (userJson != null) {
  final user = User.fromJson(jsonDecode(userJson));
}
```

Для сложных структур или больших данных — используй Hive, Isar, или SQLite.

</details>

---

### Вопрос 5 🟡

Как инициализировать `SharedPreferences` один раз при старте приложения?

- A) Инициализировать при каждом использовании
- B) В `main()`: `final prefs = await SharedPreferences.getInstance(); runApp(...)` — передать через Provider/DI
- C) `SharedPreferences.init()` в `MaterialApp`
- D) Flutter инициализирует автоматически

<details>
<summary>Ответ</summary>

**B) Инициализировать в `main()` и передать через DI**

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final prefs = await SharedPreferences.getInstance();
  runApp(
    ProviderScope(
      overrides: [sharedPrefsProvider.overrideWithValue(prefs)],
      child: MyApp(),
    ),
  );
}
```

`WidgetsFlutterBinding.ensureInitialized()` обязателен перед асинхронными платформенными вызовами в `main()`.

</details>

---

### Вопрос 6 🟡

Чем `shared_preferences` отличается от `flutter_secure_storage`?

- A) Они идентичны
- B) `shared_preferences` — незащищённое хранилище; `flutter_secure_storage` — шифрует данные через Keychain (iOS) / KeyStore (Android)
- C) `flutter_secure_storage` только для iOS
- D) `shared_preferences` быстрее и рекомендуется для всего

<details>
<summary>Ответ</summary>

**B) `flutter_secure_storage` шифрует данные**

Используй `flutter_secure_storage` для: токенов доступа, паролей, приватных ключей, персональных данных. `shared_preferences` для: настроек интерфейса, пользовательских предпочтений, non-sensitive флагов.

```dart
const storage = FlutterSecureStorage();
await storage.write(key: 'auth_token', value: token);
final token = await storage.read(key: 'auth_token');
```

</details>

---

### Вопрос 7 🟡

Что такое `SharedPreferencesAsync` (Flutter 3.22+)?

- A) Асинхронная обёртка для SharedPreferences
- B) Новая версия API без необходимости вызывать `getInstance()` — все операции async по умолчанию, лучше для concurrent доступа
- C) SharedPreferences для Server-Side Rendering
- D) Устаревшая версия SharedPreferences

<details>
<summary>Ответ</summary>

**B) Новая версия API без `getInstance()` — все операции async**

```dart
// Старый API:
final prefs = await SharedPreferences.getInstance();
await prefs.setString('key', 'value');

// Новый API (3.22+):
final prefs = SharedPreferencesAsync();
await prefs.setString('key', 'value');
final value = await prefs.getString('key');
```

Рекомендован для новых проектов — лучшая поддержка изоляций.

</details>

---

### Вопрос 8 🔴

Как реализовать реактивное хранилище поверх `SharedPreferences` (виджет автоматически обновляется при изменении)?

- A) SharedPreferences встроенно поддерживает реактивность
- B) Обернуть в `ChangeNotifier` или `StateNotifier` — при изменении вызывать `notifyListeners()` + сохранять в prefs
- C) `SharedPreferences.stream(key)` возвращает Stream
- D) Через `ValueListenableBuilder`

<details>
<summary>Ответ</summary>

**B) Обернуть в `ChangeNotifier`/`StateNotifier`**

```dart
class SettingsNotifier extends ChangeNotifier {
  SettingsNotifier(this._prefs) {
    _isDark = _prefs.getBool('isDark') ?? false;
  }
  final SharedPreferences _prefs;
  late bool _isDark;
  bool get isDark => _isDark;

  Future<void> toggleDark() async {
    _isDark = !_isDark;
    await _prefs.setBool('isDark', _isDark);
    notifyListeners(); // UI обновляется
  }
}
```

</details>

---

### Вопрос 9 🔴

Как тестировать код использующий `SharedPreferences`?

- A) Тестировать на реальном устройстве — нельзя мокировать
- B) `SharedPreferences.setMockInitialValues({})` — устанавливает в-памяти mock реализацию
- C) Создать свой класс-обёртку и мокировать его
- D) B и C оба корректных подхода

<details>
<summary>Ответ</summary>

**D) B и C оба корректны — B проще, C гибче**

```dart
// Вариант B:
setUp(() {
  SharedPreferences.setMockInitialValues({'counter': 5});
});

testWidgets('loads saved counter', (tester) async {
  final prefs = await SharedPreferences.getInstance();
  expect(prefs.getInt('counter'), 5);
});
```

Вариант C — абстракция + mockito/mocktail для большей гибкости в unit тестах.

</details>

---

### Вопрос 10 🔴

Почему не стоит использовать `SharedPreferences` для больших объёмов данных?

- A) SharedPreferences поддерживает любой объём данных
- B) На iOS вся база `NSUserDefaults` загружается в память при старте; для больших данных возникают задержки запуска, OOM, и долгие операции записи
- C) SharedPreferences ограничен 100 ключами
- D) Ограничение только в Release режиме

<details>
<summary>Ответ</summary>

**B) Вся база загружается в память при старте → задержки при больших данных**

Рекомендации по размеру:

- **SharedPreferences**: < 100 KB, < 50 ключей, примитивные значения
- **Hive/Isar**: неструктурированные или средние данные
- **SQLite/Drift**: реляционные данные, сложные запросы
- **Файловая система**: медиа, большие JSON файлы

</details>
