# Квиз: Шрифты и иконки

**Тема:** 05.2 — Fonts & Icons  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как подключить кастомный шрифт из assets в Flutter?

- A) `FontFamily.load('MyFont', 'assets/fonts/MyFont.ttf')`
- B) Добавить файл в папку, объявить в `pubspec.yaml` под `flutter.fonts`
- C) `Theme.of(context).addFont('MyFont')`
- D) Скопировать файл в `lib/fonts/`

<details>
<summary>Ответ</summary>

**B) Добавить файл в папку, объявить в `pubspec.yaml` под `flutter.fonts`**

```yaml
flutter:
  fonts:
    - family: Roboto
      fonts:
        - asset: assets/fonts/Roboto-Regular.ttf
        - asset: assets/fonts/Roboto-Bold.ttf
          weight: 700
        - asset: assets/fonts/Roboto-Italic.ttf
          style: italic
```

Затем: `TextStyle(fontFamily: 'Roboto', fontWeight: FontWeight.w700)`.

</details>

---

### Вопрос 2 🟢

Как использовать `GoogleFonts` пакет?

- A) `Text('Hi', style: GoogleFonts.roboto())`
- B) Объявить шрифт в `pubspec.yaml` через `google_fonts`
- C) `Theme.of(context).setGoogleFont('Roboto')`
- D) `import 'google:fonts/roboto.dart'`

<details>
<summary>Ответ</summary>

**A) `Text('Hi', style: GoogleFonts.roboto())`**

`google_fonts` пакет автоматически загружает шрифт из сети (или кэша). `GoogleFonts.roboto(fontSize: 16, fontWeight: FontWeight.bold)` — возвращает `TextStyle`. Для всего app: `ThemeData(textTheme: GoogleFonts.robotoTextTheme())`. Офлайн режим: добавить файлы в assets/fonts и по умолчанию они будут использоваться.

</details>

---

### Вопрос 3 🟢

Что отображает `Icon(Icons.home)`?

- A) PNG изображение
- B) Символ из Material Icons шрифта
- C) SVG иконку
- D) Системную иконку платформы

<details>
<summary>Ответ</summary>

**B) Символ из Material Icons шрифта**

`Icons` — это класс с `IconData` константами ссылающимися на codepoints в Material Icons шрифте, поставляемом с Flutter. `Icon(Icons.home, size: 24, color: Colors.blue)`. Для кастомных иконок: `Icon(IconData(0xe900, fontFamily: 'MyIcons'))`.

</details>

---

### Вопрос 4 🟡

Как применить шрифт глобально для всего приложения?

- A) `ThemeData(fontFamily: 'Inter')`
- B) Обернуть каждый `Text` в кастомный виджет
- C) `MaterialApp(fontFamily: 'Inter')`
- D) `DefaultTextStyle` на уровне `MaterialApp`

<details>
<summary>Ответ</summary>

**A) `ThemeData(fontFamily: 'Inter')`**

```dart
ThemeData(
  fontFamily: 'Inter',
  textTheme: const TextTheme(...),
)
```

Или через Google Fonts: `textTheme: GoogleFonts.interTextTheme(Theme.of(context).textTheme)`. Это применяет шрифт ко всем `Text` виджетам которые используют стили из `TextTheme`.

</details>

---

### Вопрос 5 🟡

Что такое `ImageIcon` и чем отличается от `Icon`?

- A) `ImageIcon` — анимированная иконка
- B) `ImageIcon` использует `ImageProvider` (PNG, SVG через пакет) вместо icon font
- C) `ImageIcon` поддерживает только `AssetImage`
- D) Они идентичны

<details>
<summary>Ответ</summary>

**B) `ImageIcon` использует `ImageProvider` вместо icon font**

```dart
ImageIcon(
  AssetImage('assets/icons/my_icon.png'),
  size: 24,
  color: Colors.blue, // применяется как tint
)
```

`ImageIcon` рисует изображение и применяет `color` как tint (color filter). Полезно для PNG иконок с прозрачностью. Для SVG нужен пакет `flutter_svg` с `SvgPicture.asset`.

</details>

---

### Вопрос 6 🟡

Как создать кастомный `IconData` из своего иконочного шрифта?

- A) `IconData.fromFont('MyIcons', 'home')`
- B) `IconData(0xe900, fontFamily: 'MyIcons', fontPackage: null)`
- C) `CustomIcon(font: 'MyIcons', codepoint: 0xe900)`
- D) Генерировать через `flutter_icons` пакет

<details>
<summary>Ответ</summary>

**B) `IconData(0xe900, fontFamily: 'MyIcons', fontPackage: null)`**

Codepoint берётся из генератора иконочного шрифта (IcoMoon, Fontello). Шрифт подключается через `pubspec.yaml`. Практика — создать Dart класс с константами:

```dart
class MyIcons {
  static const home = IconData(0xe900, fontFamily: 'MyIcons');
  static const profile = IconData(0xe901, fontFamily: 'MyIcons');
}
```

</details>

---

### Вопрос 7 🟡

Как отобразить SVG файл в Flutter?

- A) `Image.asset('icon.svg')`
- B) Встроенная поддержка SVG через `Icon`
- C) `SvgPicture.asset('assets/icons/logo.svg')` из пакета `flutter_svg`
- D) `SvgIcon(path: 'assets/icons/logo.svg')`

<details>
<summary>Ответ</summary>

**C) `SvgPicture.asset('assets/icons/logo.svg')` из пакета `flutter_svg`**

Flutter не поддерживает SVG нативно. `flutter_svg` парсит SVG и рисует через `Canvas`. `SvgPicture.asset(path, width: 24, colorFilter: ColorFilter.mode(Colors.blue, BlendMode.srcIn))`. Для сложных SVG с анимацией — `rive` или `lottie` пакеты.

</details>

---

### Вопрос 8 🔴

Как правильно настроить `fontFeatures` для включения лигатур в тексте?

- A) `TextStyle(ligatures: true)`
- B) `TextStyle(fontFeatures: [FontFeature.enable('liga'), FontFeature.enable('calt')])`
- C) `TextStyle(fontVariant: FontVariant.ligatures)`
- D) Лигатуры включены автоматически если шрифт поддерживает

<details>
<summary>Ответ</summary>

**B) `TextStyle(fontFeatures: [FontFeature.enable('liga'), FontFeature.enable('calt')])`**

`FontFeature` управляет OpenType функциями шрифта: `liga` (стандартные лигатуры), `calt` (контекстуальные альтернативы), `smcp` (small caps), `onum` (old-style numerals), `tnum` (tabular numbers). Полезно для профессиональной типографики.

</details>

---

### Вопрос 9 🔴

Что делает `fontVariations` в `TextStyle`?

- A) Переключает между начертаниями шрифта
- B) Управляет осями вариативного шрифта (variable font) — вес, ширина, наклон через непрерывные значения
- C) Задаёт альтернативные символы
- D) Управляет межсимвольными интервалами

<details>
<summary>Ответ</summary>

**B) Управляет осями вариативного шрифта через непрерывные значения**

```dart
TextStyle(
  fontVariations: [
    FontVariation('wght', 350.0), // вес: 100–900
    FontVariation('wdth', 85.0),  // ширина: compressed→expanded
  ],
)
```

Variable fonts (напр. Inter, Roboto Flex) содержат всё семейство в одном файле. `FontVariation.weight(350)` — удобный конструктор для оси веса.

</details>

---

### Вопрос 10 🔴

Как закэшировать шрифты `google_fonts` для работы офлайн после установки?

- A) Шрифты кэшируются автоматически в SharedPreferences
- B) `google_fonts` кэширует в файловую систему приложения автоматически; для бандлинга — скопировать файлы в `assets/fonts` и они будут использоваться без сети
- C) `GoogleFonts.config.offline = true`
- D) Использовать `flutter_cache_manager` вручную

<details>
<summary>Ответ</summary>

**B) `google_fonts` кэширует автоматически; для полного офлайна — бандлить в assets**

`google_fonts` сначала ищет файл в `assets/fonts/{family}/font.ttf` → затем в кэше приложения → загружает из сети. Для production приложений без гарантированного интернета рекомендуется скачать `.ttf` файлы и добавить в assets по конвенции именования `google_fonts`.

</details>
