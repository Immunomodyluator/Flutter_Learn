# 5. Темы и стилизация

## 1. Суть концепции

Flutter использует систему тем (`ThemeData`) для централизованного управления визуальным стилем приложения. Тема задаётся один раз в `MaterialApp` и автоматически применяется ко всем виджетам через дерево.

---

## 2. Базовая настройка темы

```dart
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF6B4EFF),  // основной цвет → генерирует всю палитру
      brightness: Brightness.light,
    ),
    textTheme: const TextTheme(
      headlineLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
      bodyLarge: TextStyle(fontSize: 16, height: 1.5),
    ),
  ),
  darkTheme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(
      seedColor: const Color(0xFF6B4EFF),
      brightness: Brightness.dark,
    ),
  ),
  themeMode: ThemeMode.system, // светлая/тёмная по настройкам системы
  home: const MyHomePage(),
)
```

---

## 3. Использование темы в виджетах

```dart
@override
Widget build(BuildContext context) {
  final theme = Theme.of(context);
  final colorScheme = theme.colorScheme;
  final textTheme = theme.textTheme;

  return Container(
    color: colorScheme.surface,
    child: Column(
      children: [
        Text('Заголовок', style: textTheme.headlineMedium),
        Text('Описание', style: textTheme.bodyMedium?.copyWith(
          color: colorScheme.onSurfaceVariant,
        )),
        FilledButton(
          style: FilledButton.styleFrom(
            backgroundColor: colorScheme.primary,
          ),
          onPressed: () {},
          child: Text('Кнопка', style: TextStyle(color: colorScheme.onPrimary)),
        ),
      ],
    ),
  );
}
```

---

## 4. ColorScheme — палитра Material 3

| Ключевые роли | Назначение                         |
| ------------- | ---------------------------------- |
| `primary`     | Основные кнопки, активные элементы |
| `onPrimary`   | Текст/иконки поверх primary        |
| `secondary`   | Акценты второго уровня             |
| `surface`     | Фоны карточек, экранов             |
| `onSurface`   | Основной текст                     |
| `error`       | Ошибки                             |
| `outline`     | Границы полей, разделители         |

```dart
final cs = Theme.of(context).colorScheme;
// cs.primary, cs.secondary, cs.error, cs.surface...
```

---

## 5. Кастомизация отдельных виджетов через тему

```dart
ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),

  // Кнопки
  filledButtonTheme: FilledButtonThemeData(
    style: FilledButton.styleFrom(
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(8)),
      minimumSize: const Size.fromHeight(52),
    ),
  ),

  // AppBar
  appBarTheme: const AppBarTheme(
    centerTitle: true,
    elevation: 0,
    scrolledUnderElevation: 1,
  ),

  // TextField
  inputDecorationTheme: InputDecorationTheme(
    border: OutlineInputBorder(borderRadius: BorderRadius.circular(12)),
    contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
  ),

  // Card
  cardTheme: CardTheme(
    elevation: 0,
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(16),
      side: BorderSide(color: Colors.grey.shade200),
    ),
  ),
)
```

---

## 6. Локальное переопределение темы

```dart
// Theme — создаёт поддерево с изменённой темой
Theme(
  data: Theme.of(context).copyWith(
    iconTheme: const IconThemeData(color: Colors.white, size: 24),
  ),
  child: Row(
    children: const [
      Icon(Icons.star),  // будет белой
      Icon(Icons.heart), // будет белой
    ],
  ),
)

// Быстрое изменение стиля кнопки
FilledButton(
  style: FilledButton.styleFrom(
    backgroundColor: Colors.red,
    foregroundColor: Colors.white,
  ),
  onPressed: () {},
  child: const Text('Удалить'),
)
```

---

## 7. Типичные ошибки

| Ошибка                        | Проблема                   | Решение                                           |
| ----------------------------- | -------------------------- | ------------------------------------------------- |
| Хардкодинг цветов             | `Colors.blue` везде        | Использовать `colorScheme.primary`                |
| `ThemeData.light()`           | Устаревший способ          | `ThemeData(useMaterial3: true, colorScheme: ...)` |
| Разные шрифты в разных местах | Несогласованность UI       | Задать шрифт в `textTheme` глобально              |
| Не используется `copyWith`    | Теряются другие стили темы | `theme.textStyle?.copyWith(...)`                  |

---

## 8. Практические рекомендации

1. **`useMaterial3: true`** — флаг для использования актуального дизайна Material 3.
2. **`ColorScheme.fromSeed`** автоматически создаёт полную палитру из одного цвета.
3. **Никаких хардкодинных `Colors.blue`** — используй семантические роли через `colorScheme`.
4. **Один объект `ThemeData`** — не переопределяй тему в каждом виджете вручную.
5. **`theme.copyWith(...)`** для локальных переопределений внутри виджета.
