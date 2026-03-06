# 5.2: Шрифты и иконки — кастомная типографика FitMenu

> Project: FitMenu | Глава 5 — Themes

### 5.2: Подключение Google Fonts (Nunito) и кастомных иконок

🎯 **Цель шага:** Подключить шрифт Nunito через пакет `google_fonts`, настроить `TextTheme` приложения и интегрировать кастомный набор иконок (calories, macros) через `flutter_svg` или `IconData`.

---

📝 **Техническое задание:**

**Часть 1 — Google Fonts:**

Добавь зависимость в `pubspec.yaml`:
```yaml
dependencies:
  google_fonts: ^6.1.0
```

Создай `AppTextTheme` в `lib/core/theme/app_text_theme.dart`:
- Используй `GoogleFonts.nunitoTextTheme()` как базу
- Настрой кастомные стили:
  - `displayLarge`: Nunito, 32px, w800 (для заголовков экранов)
  - `titleMedium`: Nunito, 16px, w700 (для названий рецептов)
  - `bodyMedium`: Nunito, 14px, w400 (основной текст)
  - `labelSmall`: Nunito, 11px, w500 (для тегов и мета-информации)
- Интегрируй в `AppTheme._buildTheme()`: `textTheme: AppTextTheme.build(colorScheme)`

**Часть 2 — SVG-иконки:**

Добавь:
```yaml
dependencies:
  flutter_svg: ^2.0.0
```

Создай `lib/core/assets/app_icons.dart` с константами путей:
```dart
class AppIcons {
  static const calories  = 'assets/icons/calories.svg';
  static const protein   = 'assets/icons/protein.svg';
  static const fat       = 'assets/icons/fat.svg';
  static const carbs     = 'assets/icons/carbs.svg';
}
```

Создай папку `assets/icons/` и добавь простые SVG-иконки (можно использовать любые из Material Symbols или нарисовать вручную).

Зарегистрируй assets в `pubspec.yaml`:
```yaml
flutter:
  assets:
    - assets/icons/
```

Создай виджет `NutrientIcon` в `lib/core/widgets/nutrient_icon.dart`:
- Параметры: `String svgPath`, `Color color`, `double size`
- Использует `SvgPicture.asset` с `colorFilter`

---

✅ **Критерии приёмки:**
- [ ] Шрифт Nunito применяется глобально (виден в эмуляторе)
- [ ] `GoogleFonts.nunitoTextTheme()` используется как база, а не единичные стили
- [ ] SVG-иконки корректно отображаются с кастомным цветом через `colorFilter`
- [ ] Пути к assets зарегистрированы в `pubspec.yaml`
- [ ] `AppTextTheme.build()` принимает `colorScheme` для правильных цветов текста
- [ ] Нет хардкода `fontFamily: 'Nunito'` в отдельных виджетах — только через тему

---

💡 **Подсказка:** `GoogleFonts.nunitoTextTheme(base)` принимает базовую `TextTheme` и возвращает её с Nunito-шрифтом. Чтобы применить к обеим темам (light/dark), передавай `ThemeData.textTheme` — он уже содержит правильные цвета для яркости. `SvgPicture.asset` с `colorFilter: ColorFilter.mode(color, BlendMode.srcIn)` — правильный способ перекрасить SVG.

---

<details>
<summary>🛠 Эталонное решение (нажми, чтобы проверить себя)</summary>

```dart
// lib/core/theme/app_text_theme.dart

import 'package:flutter/material.dart';
import 'package:google_fonts/google_fonts.dart';

class AppTextTheme {
  AppTextTheme._();

  static TextTheme build(ColorScheme colorScheme) {
    final base = ThemeData(brightness: colorScheme.brightness).textTheme;
    final nunito = GoogleFonts.nunitoTextTheme(base);

    return nunito.copyWith(
      displayLarge: nunito.displayLarge?.copyWith(
        fontSize: 32,
        fontWeight: FontWeight.w800,
        color: colorScheme.onSurface,
      ),
      titleMedium: nunito.titleMedium?.copyWith(
        fontSize: 16,
        fontWeight: FontWeight.w700,
        color: colorScheme.onSurface,
      ),
      bodyMedium: nunito.bodyMedium?.copyWith(
        fontSize: 14,
        fontWeight: FontWeight.w400,
        color: colorScheme.onSurface,
      ),
      labelSmall: nunito.labelSmall?.copyWith(
        fontSize: 11,
        fontWeight: FontWeight.w500,
        color: colorScheme.onSurfaceVariant,
      ),
    );
  }
}
```

```dart
// lib/core/assets/app_icons.dart

class AppIcons {
  AppIcons._();

  static const calories = 'assets/icons/calories.svg';
  static const protein  = 'assets/icons/protein.svg';
  static const fat      = 'assets/icons/fat.svg';
  static const carbs    = 'assets/icons/carbs.svg';
}
```

```dart
// lib/core/widgets/nutrient_icon.dart

import 'package:flutter/material.dart';
import 'package:flutter_svg/flutter_svg.dart';

class NutrientIcon extends StatelessWidget {
  const NutrientIcon({
    super.key,
    required this.svgPath,
    required this.color,
    this.size = 24,
  });

  final String svgPath;
  final Color color;
  final double size;

  @override
  Widget build(BuildContext context) {
    return SvgPicture.asset(
      svgPath,
      width: size,
      height: size,
      colorFilter: ColorFilter.mode(color, BlendMode.srcIn),
    );
  }
}
```

```yaml
# pubspec.yaml (добавить в dependencies и flutter.assets)
dependencies:
  google_fonts: ^6.1.0
  flutter_svg: ^2.0.0

flutter:
  assets:
    - assets/icons/
```

</details>
