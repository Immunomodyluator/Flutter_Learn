# 5.3 Тёмная тема

## 1. Суть концепции

Flutter поддерживает светлую, тёмную тему и автоматическое переключение по настройкам системы. Тёмная тема задаётся через `darkTheme` в `MaterialApp` и активируется через `themeMode`.

---

## 2. Базовая настройка светлой и тёмной темы

```dart
MaterialApp(
  // Светлая тема
  theme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF6B4EFF),
      brightness: Brightness.light,
    ),
  ),
  // Тёмная тема
  darkTheme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF6B4EFF),
      brightness: Brightness.dark,
    ),
  ),
  // Режим переключения
  themeMode: ThemeMode.system,  // .light / .dark / .system
  home: const MyApp(),
)
```

---

## 3. Управление темой через провайдер состояния

```dart
// Провайдер темы
class ThemeProvider extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system;

  ThemeMode get themeMode => _themeMode;

  bool get isDark => _themeMode == ThemeMode.dark;

  void setThemeMode(ThemeMode mode) {
    _themeMode = mode;
    notifyListeners();
  }

  Future<void> loadSavedTheme() async {
    final prefs = await SharedPreferences.getInstance();
    final saved = prefs.getString('theme_mode') ?? 'system';
    _themeMode = switch (saved) {
      'dark' => ThemeMode.dark,
      'light' => ThemeMode.light,
      _ => ThemeMode.system,
    };
    notifyListeners();
  }

  Future<void> saveAndSetTheme(ThemeMode mode) async {
    _themeMode = mode;
    notifyListeners();
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString('theme_mode', mode.name);
  }
}

// В MaterialApp
ChangeNotifierProvider(
  create: (_) => ThemeProvider()..loadSavedTheme(),
  child: Consumer<ThemeProvider>(
    builder: (context, themeProvider, _) {
      return MaterialApp(
        theme: lightTheme,
        darkTheme: darkTheme,
        themeMode: themeProvider.themeMode,
        home: const MyHomePage(),
      );
    },
  ),
)
```

---

## 4. Переключатель темы на экране настроек

```dart
class ThemeSettingsWidget extends StatelessWidget {
  const ThemeSettingsWidget({super.key});

  @override
  Widget build(BuildContext context) {
    final themeProvider = context.watch<ThemeProvider>();

    return Column(
      children: ThemeMode.values.map((mode) {
        final labels = {
          ThemeMode.system: 'Системная',
          ThemeMode.light: 'Светлая',
          ThemeMode.dark: 'Тёмная',
        };
        return RadioListTile<ThemeMode>(
          title: Text(labels[mode]!),
          value: mode,
          groupValue: themeProvider.themeMode,
          onChanged: (v) => themeProvider.saveAndSetTheme(v!),
        );
      }).toList(),
    );
  }
}
```

---

## 5. Адаптивные цвета в виджетах

```dart
// Всегда используй colorScheme — он адаптируется к теме автоматически
@override
Widget build(BuildContext context) {
  final cs = Theme.of(context).colorScheme;

  return Container(
    color: cs.surface,          // будет светлым или тёмным
    child: Text(
      'Текст',
      style: TextStyle(color: cs.onSurface),  // всегда читаемый
    ),
  );
}

// Определить текущую тему
final isDark = Theme.of(context).brightness == Brightness.dark;
// или
final isDark = MediaQuery.of(context).platformBrightness == Brightness.dark;
```

---

## 6. Изображения и иконки для тёмной темы

```dart
// Разные изображения для тем
Image.asset(
  Theme.of(context).brightness == Brightness.dark
      ? 'assets/logo_dark.png'
      : 'assets/logo_light.png',
)

// SVG с адаптивным цветом
SvgPicture.asset(
  'assets/logo.svg',
  colorFilter: ColorFilter.mode(
    Theme.of(context).colorScheme.onSurface,
    BlendMode.srcIn,
  ),
)
```

---

## 7. Типичные ошибки

| Ошибка                                       | Проблема                             | Решение                                          |
| -------------------------------------------- | ------------------------------------ | ------------------------------------------------ |
| Хардкодинг `Colors.white` или `Colors.black` | Не работает в тёмной теме            | Использовать `colorScheme.surface` / `onSurface` |
| `Color(0xFFFFFFFF)` напрямую                 | Не адаптируется                      | Использовать `colorScheme` роли                  |
| Нет `darkTheme` в `MaterialApp`              | Системная тёмная тема не применяется | Обязательно добавить `darkTheme:`                |
| Проверка `isDark` вместо colorScheme         | Дублирование логики                  | Использовать семантические цвета                 |

---

## 8. Практические рекомендации

1. **Никаких `Colors.white` / `Colors.black`** в UI — только `colorScheme.surface`, `colorScheme.onSurface`.
2. **`ThemeMode.system` по умолчанию** — уважай предпочтения пользователя.
3. **Сохраняй выбор темы** в `SharedPreferences` — иначе сбрасывается при перезапуске.
4. **Тестируй обе темы** в Widget-тестах — передавай `brightness: Brightness.dark` в `ThemeData`.
5. **`ColorScheme.fromSeed` с brightness** генерирует согласованную палитру для тёмной темы.
