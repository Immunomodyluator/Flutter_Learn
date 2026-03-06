# 5.2 Кастомные шрифты и иконки

## 1. Суть концепции

Flutter позволяет подключать сторонние шрифты (Google Fonts или локальные `.ttf`/`.otf` файлы) и кастомные иконки (SVG → шрифт иконок). Шрифт подключается через `pubspec.yaml` и используется в `TextStyle` или теме.

---

## 2. Google Fonts — самый простой способ

```yaml
# pubspec.yaml
dependencies:
  google_fonts: ^6.0.0
```

```dart
import 'package:google_fonts/google_fonts.dart';

// Использование напрямую
Text(
  'Привет',
  style: GoogleFonts.roboto(fontSize: 16, fontWeight: FontWeight.bold),
)

Text(
  'Заголовок',
  style: GoogleFonts.poppins(fontSize: 24),
)

// В теме приложения
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,
    textTheme: GoogleFonts.rubikTextTheme(),  // применяет шрифт ко всем стилям
  ),
)
```

---

## 3. Локальные шрифты

```
# Структура файлов
assets/
  fonts/
    Montserrat-Regular.ttf
    Montserrat-Bold.ttf
    Montserrat-Italic.ttf
```

```yaml
# pubspec.yaml
flutter:
  fonts:
    - family: Montserrat
      fonts:
        - asset: assets/fonts/Montserrat-Regular.ttf
          weight: 400
        - asset: assets/fonts/Montserrat-Bold.ttf
          weight: 700
        - asset: assets/fonts/Montserrat-Italic.ttf
          style: italic
```

```dart
// В коде
const TextStyle montserratBold = TextStyle(
  fontFamily: 'Montserrat',
  fontWeight: FontWeight.w700,
  fontSize: 18,
);

// В теме
ThemeData(
  fontFamily: 'Montserrat',  // шрифт по умолчанию для всего приложения
)
```

---

## 4. TextStyle — все параметры

```dart
const TextStyle(
  fontFamily: 'Montserrat',
  fontSize: 16,
  fontWeight: FontWeight.w600,
  fontStyle: FontStyle.italic,
  color: Colors.black87,
  height: 1.5,           // межстрочный интервал (множитель fontSize)
  letterSpacing: 0.5,    // межбуквенный интервал
  wordSpacing: 2.0,      // межсловный интервал
  decoration: TextDecoration.underline,
  decorationColor: Colors.blue,
  decorationStyle: TextDecorationStyle.dashed,
  shadows: [Shadow(color: Colors.black26, blurRadius: 4)],
  overflow: TextOverflow.ellipsis, // обрезка текста
)
```

---

## 5. Иконки — встроенные

```dart
// Material Icons (встроены в Flutter)
const Icon(Icons.home)
const Icon(Icons.search)
Icon(Icons.favorite, color: Colors.red, size: 32)

// Cupertino Icons
const Icon(CupertinoIcons.heart)
const Icon(CupertinoIcons.search)

// С бейджем (Material 3)
Badge(
  label: const Text('3'),
  child: const Icon(Icons.notifications),
)
```

---

## 6. Кастомные иконки из SVG-шрифта

Сервис [FlutterIcon](https://www.fluttericon.com/) или [IcoMoon](https://icomoon.io/) конвертирует SVG в шрифт иконок.

```yaml
# pubspec.yaml
flutter:
  fonts:
    - family: MyIcons
      fonts:
        - asset: assets/fonts/MyIcons.ttf
```

```dart
// Генерируемый класс иконок
class MyIcons {
  static const IconData logo = IconData(0xe900, fontFamily: 'MyIcons');
  static const IconData star = IconData(0xe901, fontFamily: 'MyIcons');
}

// Использование
const Icon(MyIcons.logo, size: 32)
```

---

## 7. SVG иконки через пакет flutter_svg

```yaml
dependencies:
  flutter_svg: ^2.0.0
```

```dart
import 'package:flutter_svg/flutter_svg.dart';

// Из assets
SvgPicture.asset(
  'assets/icons/logo.svg',
  width: 48,
  height: 48,
  colorFilter: ColorFilter.mode(Colors.white, BlendMode.srcIn),
)

// Из сети
SvgPicture.network('https://example.com/icon.svg')
```

---

## 8. RichText — смешанные стили текста

```dart
RichText(
  text: TextSpan(
    style: Theme.of(context).textTheme.bodyLarge,
    children: const [
      TextSpan(text: 'Обычный текст '),
      TextSpan(
        text: 'жирный',
        style: TextStyle(fontWeight: FontWeight.bold),
      ),
      TextSpan(text: ' и '),
      TextSpan(
        text: 'цветной',
        style: TextStyle(color: Colors.blue, decoration: TextDecoration.underline),
      ),
    ],
  ),
)
```

---

## 9. Типичные ошибки

| Ошибка                                | Проблема                           | Решение                                           |
| ------------------------------------- | ---------------------------------- | ------------------------------------------------- |
| Шрифт не отображается                 | Не добавлен в pubspec.yaml         | Проверить отступы в yaml, `flutter pub get`       |
| Иконка отображается как прямоугольник | Неверный codepoint                 | Проверить hex в классе иконок                     |
| `GoogleFonts` медленно загружается    | Скачивает шрифт при первом запуске | Добавить шрифт в assets для оффлайн-использования |
| Шрифт не применяется к числам         | Отдельный fontFamily для цифр      | Проверить fontFamily в TextStyle                  |

---

## 10. Практические рекомендации

1. **Google Fonts для прототипирования**, локальные шрифты для продакшна.
2. **Один шрифт в `ThemeData.fontFamily`** — единообразие без повторений.
3. **`const TextStyle`** для статических стилей — компилируется заранее.
4. **`textTheme.bodyLarge?.copyWith(...)`** а не создавать новый `TextStyle` с нуля.
5. **SVG через `flutter_svg`** — не нужно конвертировать в PNG разных размеров.
