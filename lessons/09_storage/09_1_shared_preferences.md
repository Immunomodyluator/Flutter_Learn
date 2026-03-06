# 09.1 SharedPreferences — хранение пар ключ-значение

## 1. Суть

`SharedPreferences` — обёртка над `NSUserDefaults` (iOS) и `SharedPreferences` (Android). Хранит примитивные типы данных в виде пар ключ-значение. Данные сохраняются между перезапусками приложения.

```yaml
# pubspec.yaml
dependencies:
  shared_preferences: ^2.2.2
```

---

## 2. Как используется во Flutter

Используется для хранения пользовательских настроек: тема, язык, флаги onboarding, последние поиски. **Не используй для токенов авторизации** — они не зашифрованы.

---

## 3. Основной синтаксис

```dart
import 'package:shared_preferences/shared_preferences.dart';

// Получить экземпляр (async, кешируется платформой)
final prefs = await SharedPreferences.getInstance();

// Запись
await prefs.setString('username', 'Ivan');
await prefs.setInt('age', 25);
await prefs.setBool('isDarkMode', true);
await prefs.setDouble('rating', 4.5);
await prefs.setStringList('recentSearches', ['Flutter', 'Dart']);

// Чтение (синхронно — данные уже загружены в память)
final username = prefs.getString('username');           // String?
final age = prefs.getInt('age') ?? 0;                   // int
final isDark = prefs.getBool('isDarkMode') ?? false;    // bool
final rating = prefs.getDouble('rating') ?? 0.0;        // double
final searches = prefs.getStringList('recentSearches'); // List<String>?

// Удаление
await prefs.remove('username');

// Очистить всё
await prefs.clear();

// Проверить наличие ключа
final hasKey = prefs.containsKey('username'); // bool
```

---

## 4. Реальный пример — сервис настроек

```dart
class SettingsService {
  static const String _themeKey = 'theme_mode';
  static const String _languageKey = 'language_code';
  static const String _onboardingKey = 'onboarding_done';

  final SharedPreferences _prefs;

  const SettingsService(this._prefs);

  // Статический factory для инициализации
  static Future<SettingsService> create() async {
    final prefs = await SharedPreferences.getInstance();
    return SettingsService(prefs);
  }

  // Theme
  ThemeMode get themeMode {
    final value = _prefs.getString(_themeKey) ?? 'system';
    return ThemeMode.values.firstWhere(
      (e) => e.name == value,
      orElse: () => ThemeMode.system,
    );
  }

  Future<void> setThemeMode(ThemeMode mode) =>
      _prefs.setString(_themeKey, mode.name);

  // Language
  String get languageCode => _prefs.getString(_languageKey) ?? 'ru';

  Future<void> setLanguageCode(String code) =>
      _prefs.setString(_languageKey, code);

  // Onboarding
  bool get isOnboardingDone => _prefs.getBool(_onboardingKey) ?? false;

  Future<void> completeOnboarding() => _prefs.setBool(_onboardingKey, true);
}

// --- Использование в main.dart ---
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final settings = await SettingsService.create();

  runApp(
    ChangeNotifierProvider(
      create: (_) => ThemeProvider(settings),
      child: const MyApp(),
    ),
  );
}

// --- ThemeProvider ---
class ThemeProvider extends ChangeNotifier {
  final SettingsService _settings;

  ThemeProvider(this._settings);

  ThemeMode get themeMode => _settings.themeMode;

  Future<void> toggleTheme() async {
    final newMode = _settings.themeMode == ThemeMode.dark
        ? ThemeMode.light
        : ThemeMode.dark;
    await _settings.setThemeMode(newMode);
    notifyListeners();
  }
}
```

---

## 5. flutter_secure_storage — зашифрованное хранение

```yaml
# pubspec.yaml
dependencies:
  flutter_secure_storage: ^9.0.0
```

```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class TokenStorage {
  static const _storage = FlutterSecureStorage(
    aOptions: AndroidOptions(encryptedSharedPreferences: true),
    iOptions: IOSOptions(
      accessibility: KeychainAccessibility.first_unlock,
    ),
  );

  static const String _accessTokenKey = 'access_token';
  static const String _refreshTokenKey = 'refresh_token';

  Future<void> saveTokens({
    required String accessToken,
    required String refreshToken,
  }) async {
    await Future.wait([
      _storage.write(key: _accessTokenKey, value: accessToken),
      _storage.write(key: _refreshTokenKey, value: refreshToken),
    ]);
  }

  Future<String?> get accessToken => _storage.read(key: _accessTokenKey);
  Future<String?> get refreshToken => _storage.read(key: _refreshTokenKey);

  Future<void> clearTokens() async {
    await Future.wait([
      _storage.delete(key: _accessTokenKey),
      _storage.delete(key: _refreshTokenKey),
    ]);
  }
}
```

---

## 6. Под капотом

- `SharedPreferences.getInstance()` при первом вызове читает весь файл с диска, потом кешируется.
- Запись (`set*`) асинхронна, но чтение синхронно (из кеша в памяти).
- **Не потокобезопасен** — не вызывай из разных Isolate.
- `flutter_secure_storage` использует Keychain (iOS) и EncryptedSharedPreferences / Keystore (Android).

---

## 7. Типичные ошибки

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `MissingPluginException` | Не вызван `WidgetsFlutterBinding.ensureInitialized()` | Добавь в `main()` перед `getInstance()` |
| Данные теряются при обновлении | Ключ изменён в коде | Храни все ключи как константы |
| Хранишь JSON в `setString` | Нет встроенного `setObject` | Храни как `jsonEncode(obj.toJson())` |
| Токены в SharedPreferences | Не зашифровано | Используй `flutter_secure_storage` |
| `null` при первом запуске | Ключа ещё нет | Всегда используй `?? defaultValue` |

---

## 8. Рекомендации

1. **Ключи — через константы**: `static const String _key = 'key'` — никогда строки вручную в коде.
2. **Не храни секреты**: токены → `flutter_secure_storage`, пароли → только хеш или отдай серверу.
3. **Инициализируй один раз** в `main()`, передавай через DI (Provider/get_it).
4. **Не используй для больших данных** — подходит только для настроек и флагов.
5. **`WidgetsFlutterBinding.ensureInitialized()`** обязателен перед любым async в `main()`.
