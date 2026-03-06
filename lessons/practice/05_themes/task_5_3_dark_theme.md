# 5.3: Тёмная тема — переключатель и ThemeMode

> Project: FitMenu | Глава 5 — Themes

### 5.3: Переключатель тёмной темы через ThemeMode и Provider

🎯 **Цель шага:** Добавить в FitMenu переключатель тёмной/светлой темы: пользователь нажимает тумблер в настройках — приложение мгновенно меняет тему. Состояние сохраняется в `SharedPreferences` и восстанавливается при запуске.

---

📝 **Техническое задание:**

Реализуй управление темой через `ThemeNotifier` (ChangeNotifier).

**Файлы:**
- `lib/core/theme/theme_notifier.dart` — ChangeNotifier с ThemeMode
- `lib/features/settings/screens/settings_screen.dart` — экран с тумблером

**ThemeNotifier:**
```dart
class ThemeNotifier extends ChangeNotifier {
  ThemeMode _themeMode = ThemeMode.system;
  ThemeMode get themeMode => _themeMode;

  Future<void> loadTheme() async { ... } // читает из SharedPreferences
  Future<void> setTheme(ThemeMode mode) async { ... } // сохраняет + notifyListeners
  void toggle() { ... } // переключает light ↔ dark
}
```

- Используй ключ `'theme_mode'` в SharedPreferences
- Сохраняй как строку: `'light'`, `'dark'`, `'system'`

**main.dart:**
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  final notifier = ThemeNotifier();
  await notifier.loadTheme();

  runApp(
    ChangeNotifierProvider.value(
      value: notifier,
      child: const FitMenuApp(),
    ),
  );
}
```

**FitMenuApp:**
```dart
class FitMenuApp extends StatelessWidget {
  Widget build(BuildContext context) {
    final themeMode = context.watch<ThemeNotifier>().themeMode;
    return MaterialApp(
      theme: AppTheme.lightTheme,
      darkTheme: AppTheme.darkTheme,
      themeMode: themeMode,
      ...
    );
  }
}
```

**SettingsScreen:**
- `SwitchListTile` с заголовком "Тёмная тема"
- `value` = `themeMode == ThemeMode.dark`
- При переключении вызывает `notifier.toggle()`
- Дополнительно: `SegmentedButton<ThemeMode>` с тремя вариантами (Светлая / Авто / Тёмная)

---

✅ **Критерии приёмки:**
- [ ] Тема переключается мгновенно без перезапуска приложения
- [ ] Выбранная тема сохраняется в `SharedPreferences`
- [ ] При повторном запуске приложение стартует с сохранённой темой
- [ ] `ThemeNotifier` — `ChangeNotifier`, не StatefulWidget
- [ ] `context.watch<ThemeNotifier>()` используется в `FitMenuApp` для реактивности
- [ ] `SwitchListTile` корректно отражает текущее состояние темы

---

💡 **Подсказка:** `WidgetsFlutterBinding.ensureInitialized()` должен вызываться до любых async-операций в `main()`. `context.watch<T>()` — сокращение для `Provider.of<T>(context)` с `listen: true`. `SegmentedButton` — новый Material 3 виджет, хорошая замена `ToggleButtons`.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/theme/theme_notifier.dart

import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart';

class ThemeNotifier extends ChangeNotifier {
  static const _key = 'theme_mode';

  ThemeMode _themeMode = ThemeMode.system;
  ThemeMode get themeMode => _themeMode;

  bool get isDark => _themeMode == ThemeMode.dark;

  Future<void> loadTheme() async {
    final prefs = await SharedPreferences.getInstance();
    final stored = prefs.getString(_key) ?? 'system';
    _themeMode = _fromString(stored);
    // Не вызываем notifyListeners() при инициализации — main() ещё не запущен
  }

  Future<void> setTheme(ThemeMode mode) async {
    if (_themeMode == mode) return;
    _themeMode = mode;
    final prefs = await SharedPreferences.getInstance();
    await prefs.setString(_key, _toString(mode));
    notifyListeners();
  }

  void toggle() {
    setTheme(_themeMode == ThemeMode.dark ? ThemeMode.light : ThemeMode.dark);
  }

  ThemeMode _fromString(String value) => switch (value) {
        'light'  => ThemeMode.light,
        'dark'   => ThemeMode.dark,
        _        => ThemeMode.system,
      };

  String _toString(ThemeMode mode) => switch (mode) {
        ThemeMode.light  => 'light',
        ThemeMode.dark   => 'dark',
        ThemeMode.system => 'system',
        _ => 'system',
      };
}
```

```dart
// lib/features/settings/screens/settings_screen.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
// import 'package:fitmenu/core/theme/theme_notifier.dart';

class SettingsScreen extends StatelessWidget {
  const SettingsScreen({super.key});

  @override
  Widget build(BuildContext context) {
    final notifier = context.watch<ThemeNotifier>();

    return Scaffold(
      appBar: AppBar(title: const Text('Настройки')),
      body: ListView(
        children: [
          // Простой тумблер
          SwitchListTile(
            title: const Text('Тёмная тема'),
            subtitle: const Text('Включить тёмный режим оформления'),
            secondary: const Icon(Icons.dark_mode),
            value: notifier.themeMode == ThemeMode.dark,
            onChanged: (_) => notifier.toggle(),
          ),

          const Divider(),

          // Segmented Button (Material 3)
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text(
                  'Режим темы',
                  style: Theme.of(context).textTheme.titleSmall,
                ),
                const SizedBox(height: 8),
                SegmentedButton<ThemeMode>(
                  segments: const [
                    ButtonSegment(
                      value: ThemeMode.light,
                      icon: Icon(Icons.light_mode),
                      label: Text('Светлая'),
                    ),
                    ButtonSegment(
                      value: ThemeMode.system,
                      icon: Icon(Icons.brightness_auto),
                      label: Text('Авто'),
                    ),
                    ButtonSegment(
                      value: ThemeMode.dark,
                      icon: Icon(Icons.dark_mode),
                      label: Text('Тёмная'),
                    ),
                  ],
                  selected: {notifier.themeMode},
                  onSelectionChanged: (modes) => notifier.setTheme(modes.first),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

</details>
