# 5.1 Material 3

## 1. Суть концепции

Material 3 (Material You) — актуальная версия дизайн-системы Google (2021+). Ключевые изменения по сравнению с Material 2:

- Динамические цветовые схемы (генерация из одного цвета)
- Новые компоненты: `NavigationBar`, `FilledButton`, `DropdownMenu`
- Обновлённые формы виджетов (более округлые)
- Поддержка адаптивных цветов (Android 12+ Monet)

---

## 2. Включение Material 3

```dart
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,                    // обязательный флаг
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF6750A4),  // основной цвет
    ),
  ),
)
```

С Flutter 3.16+ `useMaterial3: true` включён по умолчанию.

---

## 3. Новые ключевые компоненты M3

### NavigationBar (вместо BottomNavigationBar)

```dart
Scaffold(
  body: _pages[_currentIndex],
  bottomNavigationBar: NavigationBar(
    selectedIndex: _currentIndex,
    onDestinationSelected: (i) => setState(() => _currentIndex = i),
    indicatorColor: Theme.of(context).colorScheme.secondaryContainer,
    destinations: const [
      NavigationDestination(
        icon: Icon(Icons.home_outlined),
        selectedIcon: Icon(Icons.home),
        label: 'Главная',
      ),
      NavigationDestination(
        icon: Icon(Icons.search),
        label: 'Поиск',
      ),
      NavigationDestination(
        icon: Icon(Icons.person_outline),
        selectedIcon: Icon(Icons.person),
        label: 'Профиль',
      ),
    ],
  ),
)
```

### FilledButton (вместо ElevatedButton)

```dart
// Primary action
FilledButton(onPressed: () {}, child: const Text('Сохранить'))

// Secondary, менее акцентированный
FilledButton.tonal(onPressed: () {}, child: const Text('Вторичное действие'))

// С иконкой
FilledButton.icon(
  onPressed: () {},
  icon: const Icon(Icons.add),
  label: const Text('Создать'),
)
```

### Card — варианты

```dart
// Elevated (по умолчанию) — с тенью
Card(child: Padding(padding: EdgeInsets.all(16), child: Text('Elevated')))

// Filled — заливка без тени
Card.filled(child: Padding(padding: EdgeInsets.all(16), child: Text('Filled')))

// Outlined — граница без заливки
Card.outlined(child: Padding(padding: EdgeInsets.all(16), child: Text('Outlined')))
```

---

## 4. Typography — актуальные стили текста M3

```dart
// M3 заменяет старые bodyText1/bodyText2/headline1 на:
final tt = Theme.of(context).textTheme;

tt.displayLarge    // 57sp
tt.displayMedium   // 45sp
tt.displaySmall    // 36sp
tt.headlineLarge   // 32sp
tt.headlineMedium  // 28sp
tt.headlineSmall   // 24sp
tt.titleLarge      // 22sp — AppBar заголовок
tt.titleMedium     // 16sp — ListTile title
tt.titleSmall      // 14sp
tt.bodyLarge       // 16sp — основной текст
tt.bodyMedium      // 14sp
tt.bodySmall       // 12sp
tt.labelLarge      // 14sp — кнопки
tt.labelMedium     // 12sp
tt.labelSmall      // 11sp
```

---

## 5. Генерация полной темы через Material Theme Builder

```dart
// Инструмент: https://m3.material.io/theme-builder
// Генерирует ColorScheme на основе образца цвета

// Пример ручного задания палитры
const lightColorScheme = ColorScheme(
  brightness: Brightness.light,
  primary: Color(0xFF6750A4),
  onPrimary: Color(0xFFFFFFFF),
  primaryContainer: Color(0xFFEADDFF),
  onPrimaryContainer: Color(0xFF21005D),
  secondary: Color(0xFF625B71),
  onSecondary: Color(0xFFFFFFFF),
  secondaryContainer: Color(0xFFE8DEF8),
  onSecondaryContainer: Color(0xFF1D192B),
  error: Color(0xFFB3261E),
  onError: Color(0xFFFFFFFF),
  surface: Color(0xFFFFFBFE),
  onSurface: Color(0xFF1C1B1F),
  // ... остальные роли
);
```

---

## 6. Динамические цвета (Android 12+)

```dart
// Пакет: dynamic_color
import 'package:dynamic_color/dynamic_color.dart';

DynamicColorBuilder(
  builder: (lightDynamic, darkDynamic) {
    return MaterialApp(
      theme: ThemeData(
        useMaterial3: true,
        colorScheme: lightDynamic ?? ColorScheme.fromSeed(seedColor: Colors.purple),
      ),
      darkTheme: ThemeData(
        useMaterial3: true,
        colorScheme: darkDynamic ?? ColorScheme.fromSeed(
          seedColor: Colors.purple,
          brightness: Brightness.dark,
        ),
      ),
    );
  },
)
```

---

## 7. Типичные ошибки

| Ошибка                        | Проблема             | Решение                                    |
| ----------------------------- | -------------------- | ------------------------------------------ |
| `ElevatedButton` в M3         | Устаревший компонент | Использовать `FilledButton`                |
| `BottomNavigationBar` в M3    | Устаревший компонент | Использовать `NavigationBar`               |
| `bodyText1` / `headline1`     | Удалены в M3         | Использовать `bodyLarge` / `headlineLarge` |
| Не задан `useMaterial3: true` | Смешанный стиль      | Явно указать флаг                          |

---

## 8. Практические рекомендации

1. **Всегда `useMaterial3: true`** для новых проектов.
2. **`ColorScheme.fromSeed`** — не задавай каждый цвет вручную, только seed.
3. **`NavigationBar` вместо `BottomNavigationBar`** — новый стандарт.
4. **`FilledButton.tonal`** для второстепенных действий.
5. **Используй `Card.outlined`** для списков, `Card.filled` для фона.
6. **`dynamic_color`** для поддержки Material You на Android 12+.
