# Квиз: Flutter Web

**Тема:** 17.1 — Flutter for Web  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как собрать Flutter Web приложение?

- A) `flutter build web` — единственный вариант
- B) `flutter build web` — production сборка; `flutter run -d chrome` — режим разработки
- C) `flutter web build --release`
- D) Требуется отдельная установка Flutter Web SDK

<details>
<summary>Ответ</summary>

**B) `flutter build web` для production; `flutter run -d chrome` для разработки**

```bash
# Режим разработки:
flutter run -d chrome

# Production сборка (CanvasKit renderer по умолчанию):
flutter build web --release

# Выбор renderer:
flutter build web --web-renderer canvaskit  # CanvasKit (по умолчанию для prod)
flutter build web --web-renderer html       # HTML renderer
flutter build web --web-renderer auto       # авто по типу устройства

# Результат в build/web/:
# index.html, main.dart.js, flutter_service_worker.js, ...

# Хостинг (любой статический хостинг):
# Firebase Hosting, Vercel, Netlify, GitHub Pages, Nginx
```

</details>

---

### Вопрос 2 🟢

В чём разница между CanvasKit и HTML рендерером?

- A) Это одно и то же с разными названиями
- B) CanvasKit — WebAssembly + Skia, пиксельно-точный, больший размер загрузки; HTML — нативные HTML/CSS элементы, меньше загрузки, хуже совместимость
- C) HTML рендерер только для мобильных браузеров
- D) CanvasKit работает без WebAssembly

<details>
<summary>Ответ</summary>

**B) CanvasKit = WASM + Skia; HTML = нативные элементы**

| | CanvasKit | HTML |
|---|---------|------|
| Технология | WebAssembly + Skia | HTML/CSS/Canvas2D |
| Начальный размер | +1.5 MB (canvaskit.wasm) | Меньше |
| Рендеринг | Идентичен мобильному | Могут быть отличия |
| Шрифты | Встроены в WASM | Системные |
| Производительность | Высокая | Средняя |
| Совместимость | Требует WebAssembly | Шире |

```bash
# Auto renderer: CanvasKit на десктопе, HTML на мобильных браузерах:
flutter build web --web-renderer auto

# Уменьшить размер загрузки CanvasKit:
flutter build web --web-renderer canvaskit \
  --dart-define=FLUTTER_WEB_CANVASKIT_URL=/canvaskit/
```

</details>

---

### Вопрос 3 🟢

Как настроить PWA (Progressive Web App) для Flutter Web?

- A) Flutter Web автоматически становится PWA без настройки
- B) `web/manifest.json` + `flutter_service_worker.js` генерируется автоматически; настроить иконки, название, цвет в manifest
- C) `flutter build web --pwa`
- D) PWA требует отдельный пакет

<details>
<summary>Ответ</summary>

**B) `web/manifest.json` уже создан при создании проекта**

```json
// web/manifest.json:
{
  "name": "My Flutter App",
  "short_name": "MyApp",
  "start_url": ".",
  "display": "standalone",
  "background_color": "#FFFFFF",
  "theme_color": "#1976D2",
  "description": "Описание приложения",
  "orientation": "portrait-primary",
  "prefer_related_applications": false,
  "icons": [
    {"src": "icons/Icon-192.png", "sizes": "192x192", "type": "image/png"},
    {"src": "icons/Icon-512.png", "sizes": "512x512", "type": "image/png"},
    {"src": "icons/Icon-maskable-192.png", "sizes": "192x192",
     "type": "image/png", "purpose": "maskable"},
    {"src": "icons/Icon-maskable-512.png", "sizes": "512x512",
     "type": "image/png", "purpose": "maskable"}
  ]
}
```

Service Worker генерируется автоматически при `flutter build web`.

</details>

---

### Вопрос 4 🟡

Каковы ограничения Flutter Web с точки зрения SEO?

- A) Flutter Web отлично индексируется поисковиками
- B) CanvasKit рендерит в Canvas — текст невидим для поисковиков; HTML рендерер лучше для SEO; критический контент требует Server-Side Rendering
- C) SEO работает только с HTML renderer
- D) Flutter Web автоматически генерирует meta теги

<details>
<summary>Ответ</summary>

**B) Canvas = невидим для поисковиков; нужен SSR или специальный подход**

```html
<!-- web/index.html — добавить базовые meta теги: -->
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Описание приложения">
  <meta property="og:title" content="Название">
  <meta property="og:description" content="Описание">
  <meta property="og:image" content="/preview.jpg">
  <title>My App</title>
</head>
```

```dart
// Обновлять title программно:
import 'package:flutter_web_plugins/flutter_web_plugins.dart';

// Для динамического SEO — использовать SSR фреймворки:
// - Dart Frog с pre-rendering
// - Или отдельный SSR сервер, Flutter только для SPA части
```

Для SEO-критичных приложений: Flutter Web лучше для SPA/admin панелей, а не для публичных сайтов.

</details>

---

### Вопрос 5 🟡

Как настроить URL стратегию (url_strategy) во Flutter Web?

- A) URL стратегия настраивается в nginx
- B) `usePathUrlStrategy()` — убирает `#` из URL; `setUrlStrategy(HashUrlStrategy())` — возвращает hash URLs
- C) `flutter build web --url-strategy path`
- D) Стратегия URL не влияет на Flutter Web

<details>
<summary>Ответ</summary>

**B) `usePathUrlStrategy()` для path-based URLs**

```dart
// main.dart:
import 'package:flutter_web_plugins/url_strategy.dart';

void main() {
  // Убрать # из URL (myapp.com/about вместо myapp.com/#/about):
  usePathUrlStrategy();
  runApp(const MyApp());
}

// ВАЖНО: при path URL strategy нужна настройка сервера!
// Nginx:
// location / {
//   try_files $uri $uri/ /index.html;
// }

// Firebase Hosting (firebase.json):
// "rewrites": [{"source": "**", "destination": "/index.html"}]

// Hash стратегия (по умолчанию) — работает без настройки сервера:
// myapp.com/#/about — браузер не делает запрос к серверу
```

</details>

---

### Вопрос 6 🟡

Как обрабатывать разные размеры экрана в Flutter Web?

- A) Использовать только `MediaQuery.of(context).size`
- B) Комбинировать `LayoutBuilder`, `MediaQuery`, адаптивные виджеты; `kIsWeb` константа; разные layout для мобильного/планшетного/десктопного
- C) Flutter Web всегда рендерит как мобильное приложение
- D) `ResponsiveBuilder` — единственный инструмент

<details>
<summary>Ответ</summary>

**B) LayoutBuilder + MediaQuery + адаптивные breakpoints**

```dart
import 'package:flutter/foundation.dart' show kIsWeb;

class ResponsiveLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth >= 1200) {
          return const DesktopLayout();
        } else if (constraints.maxWidth >= 768) {
          return const TabletLayout();
        } else {
          return const MobileLayout();
        }
      },
    );
  }
}

// Breakpoints:
class Breakpoints {
  static const mobile = 768.0;
  static const tablet = 1024.0;
  static const desktop = 1440.0;
}

// Адаптивный Scaffold:
Scaffold(
  drawer: isMobile ? Drawer(...) : null,
  body: Row(children: [
    if (!isMobile) NavigationRail(...),
    Expanded(child: content),
  ]),
)
```

</details>

---

### Вопрос 7 🟡

Как уменьшить размер Flutter Web приложения?

- A) Размер нельзя контролировать
- B) `--tree-shake-icons`, деferred loading, оптимизация assets, выбор HTML renderer для лёгких приложений, WASM tree-shaking
- C) Только через сжатие nginx
- D) `flutter build web --minimize`

<details>
<summary>Ответ</summary>

**B) Tree-shaking + deferred loading + оптимизация**

```bash
# --tree-shake-icons: удалить неиспользуемые иконки Material:
flutter build web --release --tree-shake-icons

# Типичный размер:
# HTML renderer: ~200-400 KB gzip
# CanvasKit: ~1.5-2 MB gzip

# Deferred imports в Dart (lazy loading):
import 'package:my_app/heavy_feature.dart' deferred as heavy;

void openHeavyFeature() async {
  await heavy.loadLibrary(); // загружается только при вызове
  heavy.showHeavyFeature();
}

# Оптимизация изображений:
# - WebP вместо PNG/JPEG
# - flutter build web создаёт gzip/brotli версии .js файлов

# Анализ размера:
flutter build web --analyze-size
```

</details>

---

### Вопрос 8 🔴

Как реализовать deep links (URL routing) в Flutter Web?

- A) Deep links недоступны в Flutter Web
- B) `go_router` обрабатывает URL → виджеты; `GoRouter.of(context).go('/path')` для навигации; браузерная кнопка Назад работает автоматически
- C) Только hash-based routing
- D) `Navigator.pushNamed()` достаточно для Web

<details>
<summary>Ответ</summary>

**B) `go_router` с полноценным URL routing**

```dart
final router = GoRouter(
  routes: [
    GoRoute(
      path: '/',
      builder: (_, __) => const HomeScreen(),
    ),
    GoRoute(
      path: '/products',
      builder: (_, __) => const ProductsScreen(),
      routes: [
        GoRoute(
          path: ':id',  // /products/123
          builder: (_, state) => ProductDetailScreen(
            id: state.pathParameters['id']!,
          ),
        ),
      ],
    ),
    GoRoute(
      path: '/search',
      builder: (_, state) => SearchScreen(
        query: state.uri.queryParameters['q'],  // /search?q=flutter
      ),
    ),
  ],
  // Редирект при неавторизованном доступе:
  redirect: (context, state) {
    final isLoggedIn = ref.read(authProvider).isLoggedIn;
    if (!isLoggedIn && state.uri.path != '/login') return '/login';
    return null;
  },
);
```

</details>

---

### Вопрос 9 🔴

Как использовать Web-специфичные API (JavaScript interop) из Flutter?

- A) Flutter Web не имеет доступа к JS API
- B) `dart:js_interop` (Dart 3+) или `package:web`; `@JS()` аннотации для биндингов; `allowInterop()` для коллбеков
- C) Только через platform channels
- D) `external` функции без аннотаций

<details>
<summary>Ответ</summary>

**B) `dart:js_interop` + `package:web`**

```dart
// Dart 3+ dart:js_interop:
import 'dart:js_interop';

@JS('window.localStorage.setItem')
external void setLocalStorageItem(String key, String value);

@JS('window.localStorage.getItem')
external String? getLocalStorageItem(String key);

// package:web — типизированные биндинги к Web API:
import 'package:web/web.dart';

void useBrowserAPI() {
  // Clipboard API:
  window.navigator.clipboard.writeText('Hello Web!').toDart;

  // Geolocation:
  window.navigator.geolocation.getCurrentPosition(
    (position) { ... }.toJS,
  );

  // Web Audio:
  final ctx = AudioContext();
  final oscillator = ctx.createOscillator();
}

// Условная компиляция для Web/Native:
import 'package:flutter/foundation.dart' show kIsWeb;

void doSomething() {
  if (kIsWeb) {
    // web-specific code
  } else {
    // mobile code
  }
}
```

</details>

---

### Вопрос 10 🔴

Как настроить Content Security Policy (CSP) для Flutter Web?

- A) CSP не нужен для Flutter Web
- B) Добавить CSP заголовки на сервере; CanvasKit требует `wasm-unsafe-eval`; инлайн скрипты нужен `nonce` или `unsafe-inline`
- C) `flutter build web --csp` автоматически настраивает
- D) CSP блокирует Flutter Web полностью

<details>
<summary>Ответ</summary>

**B) CSP заголовки на сервере с учётом требований Flutter**

```
Минимальный CSP для Flutter CanvasKit:

Content-Security-Policy:
  default-src 'self';
  script-src 'self' 'unsafe-inline' 'wasm-unsafe-eval';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: blob:;
  font-src 'self' data:;
  connect-src 'self' https://fonts.gstatic.com;
  worker-src 'self' blob:;
```

```dart
// Генерация nonce для безопасного CSP (без unsafe-inline):
// В web/index.html использовать CSP nonce:
// <script nonce="{{flutter_nonce}}">

// Firebase Hosting (firebase.json):
// "headers": [{
//   "source": "**",
//   "headers": [{
//     "key": "Content-Security-Policy",
//     "value": "default-src 'self'; script-src 'self' 'wasm-unsafe-eval'..."
//   }]
// }]

// Nginx:
// add_header Content-Security-Policy "...";
```

</details>
