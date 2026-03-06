# 5.1: Material 3 тема — AppTheme для FitMenu

> Project: FitMenu | Глава 5 — Themes

### 5.1: Настройка Material Design 3 в ThemeData

🎯 **Цель шага:** Создать централизованный `AppTheme` для FitMenu на базе Material 3: с фирменной цветовой схемой (зелёный + белый), закруглёнными компонентами и правильной типографикой.

---

📝 **Техническое задание:**

Реализуй тему в файле `lib/core/theme/app_theme.dart`.

**Требования:**
- Включи Material 3: `useMaterial3: true`
- Создай `ColorScheme.fromSeed(seedColor: Color(0xFF4CAF50))` для светлой темы
- Настрой `AppBarTheme`: `centerTitle: true`, `elevation: 0`, titleTextStyle с `fontWeight: w600`
- Настрой `CardTheme`: `elevation: 2`, `shape: RoundedRectangleBorder(borderRadius: 12)`
- Настрой `ElevatedButton.styleFrom`: `shape` с `borderRadius: 12`, `padding: 16×14`
- Настрой `InputDecorationTheme`: `border: OutlineInputBorder(borderRadius: 12)`, `filled: true`
- Создай статический класс `AppTheme` с двумя геттерами: `lightTheme` и `darkTheme`

**Структура класса:**
```dart
class AppTheme {
  AppTheme._(); // приватный конструктор

  static ThemeData get lightTheme => ThemeData(...)
  static ThemeData get darkTheme  => ThemeData(...)
}
```

**В `main.dart` подключи:**
```dart
MaterialApp(
  theme: AppTheme.lightTheme,
  darkTheme: AppTheme.darkTheme,
  themeMode: ThemeMode.system,
  ...
)
```

---

✅ **Критерии приёмки:**
- [ ] `useMaterial3: true` включён в обеих темах
- [ ] `ColorScheme.fromSeed` используется (не `primarySwatch`)
- [ ] Тема применяется глобально через `MaterialApp.theme`
- [ ] Карточки имеют скруглённые углы (borderRadius 12) по умолчанию
- [ ] AppBar без тени (`elevation: 0`)
- [ ] Приложение переключается между светлой/тёмной темой по настройке системы

---

💡 **Подсказка:** `ColorScheme.fromSeed` автоматически генерирует всю цветовую палитру из одного цвета-источника. Для кастомизации отдельных цветов используй `.copyWith(secondary: ...)`. `ThemeData.copyWith` позволяет переиспользовать lightTheme как основу для darkTheme.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/theme/app_theme.dart

import 'package:flutter/material.dart';

class AppTheme {
  AppTheme._();

  static const _seedColor = Color(0xFF4CAF50); // FitMenu Green

  static ThemeData get lightTheme => _buildTheme(
        colorScheme: ColorScheme.fromSeed(
          seedColor: _seedColor,
          brightness: Brightness.light,
        ),
      );

  static ThemeData get darkTheme => _buildTheme(
        colorScheme: ColorScheme.fromSeed(
          seedColor: _seedColor,
          brightness: Brightness.dark,
        ),
      );

  static ThemeData _buildTheme({required ColorScheme colorScheme}) {
    return ThemeData(
      useMaterial3: true,
      colorScheme: colorScheme,

      // AppBar
      appBarTheme: AppBarTheme(
        centerTitle: true,
        elevation: 0,
        scrolledUnderElevation: 1,
        backgroundColor: colorScheme.surface,
        foregroundColor: colorScheme.onSurface,
        titleTextStyle: TextStyle(
          color: colorScheme.onSurface,
          fontSize: 18,
          fontWeight: FontWeight.w600,
        ),
      ),

      // Card
      cardTheme: CardTheme(
        elevation: 2,
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(12),
        ),
        clipBehavior: Clip.antiAlias,
      ),

      // ElevatedButton
      elevatedButtonTheme: ElevatedButtonThemeData(
        style: ElevatedButton.styleFrom(
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(12),
          ),
          padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 14),
          textStyle: const TextStyle(fontWeight: FontWeight.w600),
        ),
      ),

      // TextButton
      textButtonTheme: TextButtonThemeData(
        style: TextButton.styleFrom(
          shape: RoundedRectangleBorder(
            borderRadius: BorderRadius.circular(8),
          ),
        ),
      ),

      // Input fields
      inputDecorationTheme: InputDecorationTheme(
        filled: true,
        fillColor: colorScheme.surfaceVariant.withOpacity(0.5),
        border: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
          borderSide: BorderSide.none,
        ),
        focusedBorder: OutlineInputBorder(
          borderRadius: BorderRadius.circular(12),
          borderSide: BorderSide(color: colorScheme.primary, width: 2),
        ),
        contentPadding:
            const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
      ),

      // Chip
      chipTheme: ChipThemeData(
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
        labelStyle: const TextStyle(fontSize: 12),
      ),
    );
  }
}
```

</details>
