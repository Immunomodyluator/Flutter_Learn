# Квиз: Flutter DevTools

**Тема:** 15.1 — DevTools & Performance Profiling  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Как запустить Flutter DevTools?

- A) `flutter analyze --devtools`
- B) `flutter run` → URL в консоли; или `dart devtools`; или через IDE (VS Code / Android Studio)
- C) `flutter devtools start`
- D) Только через Android Studio

<details>
<summary>Ответ</summary>

**B) Автоматически при `flutter run`; или `dart devtools`**

```bash
# Запустить приложение — DevTools URL в консоли:
flutter run
# → "An Observatory debugger and profiler on Chrome is available at: ..."

# Запустить DevTools отдельно:
dart devtools

# Или через flutter:
flutter pub global activate devtools
flutter pub global run devtools

# В VS Code: Ctrl+Shift+P → "Flutter: Open DevTools"
# В Android Studio: View → Tool Windows → Flutter Inspector
```

DevTools включает: Widget Inspector, Performance, CPU Profiler, Memory, Network, Logging.

</details>

---

### Вопрос 2 🟢

Что показывает Performance Overlay в Flutter?

- A) Только FPS счётчик
- B) Два графика: верхний — Raster thread (GPU), нижний — UI thread (Dart); зелёные полосы = в пределах 16ms; красные = джанки
- C) Время загрузки приложения
- D) Использование памяти по времени

<details>
<summary>Ответ</summary>

**B) Два потока: Raster и UI thread**

```dart
// Включить Performance Overlay программно:
MaterialApp(
  showPerformanceOverlay: true,
  ...
)

// Или через DevTools: Performance → Toggle Performance Overlay
```

| Поток | Задача | Причины красного |
|-------|--------|-----------------|
| **UI (Dart)** | Построение виджетов, логика | Тяжёлые build() методы, синхронный IO |
| **Raster (GPU)** | Рендеринг на GPU | Сложные эффекты, saveLayer, большие текстуры |

Цель: оба < 16ms (60 FPS) или < 8ms (120 FPS).

</details>

---

### Вопрос 3 🟢

Что такое Widget Inspector в DevTools?

- A) Инструмент для проверки грамматики в виджетах
- B) Визуальный инструмент для исследования дерева виджетов: выбор виджета на экране, просмотр свойств, layout bounds
- C) Отладчик для исправления кода виджетов
- D) Только для StatefulWidget

<details>
<summary>Ответ</summary>

**B) Визуальный исследователь дерева виджетов**

Возможности Widget Inspector:
- **Select widget**: кликнуть на виджет в приложении — подсветка в дереве
- **Widget details tree**: полное дерево с типами и свойствами
- **Layout Explorer**: визуализация Flex, Column, Row — размеры и constraints
- **Slow Animations**: замедлить анимации в 5× для отладки
- **Repaint Rainbow**: цветные границы при каждой перерисовке слоя
- **Debug Paint**: границы виджетов, padding, clip regions

```dart
// Программно включить отладочные режимы:
import 'package:flutter/rendering.dart';

debugPaintSizeEnabled = true;    // границы виджетов
debugRepaintRainbowEnabled = true; // перерисовки
debugPrintLayouts = true;
```

</details>

---

### Вопрос 4 🟡

Как использовать Timeline view в DevTools для поиска узких мест?

- A) Timeline показывает только сетевые запросы
- B) Запись трейса → поиск фреймов > 16ms → разворачивание стека вызовов → нахождение дорогостоящих операций
- C) Timeline = CPU Profiler — одно и то же
- D) Только для release режима

<details>
<summary>Ответ</summary>

**B) Анализ фреймов и стека вызовов**

```
DevTools → Performance → Record → воспроизвести действие → Stop

В Timeline:
1. Найти красные/жёлтые фреймы (> 16ms)
2. Кликнуть на фрейм → видны UI и Raster секции
3. Разворачивать события → найти дорогостоящие build/layout/paint
4. Смотреть: Widget.build, RenderObject.layout, Canvas.drawXxx

Оптимальный фрейм:
- UI thread: < 8ms
- Raster thread: < 8ms
- Всего: < 16ms для 60 FPS
```

```dart
// Добавить кастомные метки в Timeline:
import 'dart:developer';

Timeline.startSync('MyExpensiveOperation');
// ... тяжёлый код ...
Timeline.finishSync();
```

</details>

---

### Вопрос 5 🟡

Как анализировать утечки памяти в Memory profiler?

- A) Memory profiler показывает только использование RAM
- B) Snapshot → сравнение двух снапшотов → анализ retained objects → поиск неосвобождённых Listeners/Controllers
- C) Только через Android Studio Memory Profiler
- D) `flutter memory --leak-detection`

<details>
<summary>Ответ</summary>

**B) Snapshots + сравнение + анализ retained objects**

```
DevTools → Memory → Heap Snapshot:
1. Снапшот 1 (начало)
2. Выполнить подозрительное действие (открыть/закрыть экран)
3. Снапшот 2 (после GC: кнопка "GC" в DevTools)
4. Сравнить: найти объекты, которые не были освобождены

Частые утечки в Flutter:
- StreamSubscription не отменён в dispose()
- AnimationController не disposed
- ChangeNotifier listener не удалён
- Timer не отменён
- GlobalKey держит BuildContext
```

```dart
// Правильный dispose:
@override
void dispose() {
  _subscription.cancel();
  _controller.dispose();
  _timer?.cancel();
  _changeNotifier.removeListener(_onChanged);
  super.dispose();
}
```

</details>

---

### Вопрос 6 🟡

Как использовать CPU Profiler в DevTools?

- A) CPU Profiler показывает использование процессора устройством
- B) Профилирование Dart кода: запись сэмплов → flame graph → hotspot функции, которые занимают больше всего времени
- C) Только для нативного (Android/iOS) кода
- D) CPU профилирование доступно только в release режиме

<details>
<summary>Ответ</summary>

**B) Flame graph для поиска hotspot-функций**

```
DevTools → CPU Profiler → Record → действие → Stop

Flame graph:
- По горизонтали: время (% от total)
- По вертикали: стек вызовов
- Широкие блоки = много времени
- Клик → фокус на функцию

Вкладки:
- Bottom up: от листьев стека — где реально тратится время
- Call tree: сверху вниз — какие функции вызывают что
- Method table: сортировка по self/total time
```

```dart
// Маркеры для удобства в flame graph:
import 'dart:developer';

void expensiveMethod() {
  Timeline.startSync('expensiveMethod', arguments: {'input': value});
  // ...
  Timeline.finishSync();
}
```

</details>

---

### Вопрос 7 🟡

Как использовать Network view в DevTools?

- A) Network view только для WebSocket
- B) Отображает HTTP запросы с URL, статусом, размером, временем; показывает headers и body; помогает отлаживать API вызовы
- C) Network view работает только в debug режиме на Android
- D) `flutter network --monitor`

<details>
<summary>Ответ</summary>

**B) Мониторинг HTTP трафика приложения**

```
DevTools → Network:
- Список запросов с URL, метод, статус, размер, время
- Клик на запрос → Headers, Response body, Timing
- Фильтрация по URL
- Поиск медленных запросов

Ограничение: работает только для http/dart:io HttpClient.
Для Dio добавить interceptor логирования дополнительно.
```

```dart
// Dio logging interceptor:
dio.interceptors.add(LogInterceptor(
  requestBody: true,
  responseBody: true,
));

// Или использовать dio_smart_retry + DevTools Network одновременно
```

Полезно для: отладки CORS, проверки заголовков авторизации, анализа времени ответа.

</details>

---

### Вопрос 8 🔴

Как профилировать startup time (время запуска) Flutter приложения?

- A) `flutter run --measure-startup`
- B) `flutter run --trace-startup` → анализ `start_up_info.json`; DevTools Performance → импорт; или `flutter build` + systrace
- C) Startup time нельзя профилировать
- D) Только через Xcode Instruments / Android Studio

<details>
<summary>Ответ</summary>

**B) `--trace-startup` + анализ JSON**

```bash
# Записать трейс запуска:
flutter run --trace-startup --profile

# Результат: build/start_up_info.json:
{
  "engineEnterTimestampMicros": 96025565262,
  "timeToFirstFrameMicros": 2171978,
  "timeToFrameworkInitMicros": 514585,
  "timeAfterFrameworkInitMicros": 1657393
}

# Трейс в Chrome: 
flutter run --trace-startup --profile
# Открыть chrome://tracing → Load → build/*.timeline_summary.json
```

Оптимизации для запуска:
- Deferred imports (`import 'package:x/x.dart' deferred as x`)
- Минимизировать работу в `main()` до `runApp()`
- Splash screen через native (не Flutter)

</details>

---

### Вопрос 9 🔴

Как использовать `dart:developer` для добавления кастомной диагностики в DevTools?

- A) `dart:developer` — только для печати в консоль
- B) `Timeline`, `log()`, `inspect()`, `Service.getVM()` — богатое API для кастомных метрик и интеграции с DevTools
- C) Только `print()` для отладки
- D) `dart:developer` недоступен во Flutter

<details>
<summary>Ответ</summary>

**B) Богатое API для диагностики**

```dart
import 'dart:developer';

// Структурированное логирование (видно в DevTools → Logging):
log('User authenticated',
  name: 'AuthService',
  time: DateTime.now(),
  sequenceNumber: _seq++,
  level: Level.INFO.value,
  error: null,
);

// Timeline метки:
Timeline.startSync('BuildProductList', arguments: {'count': products.length});
final widgets = products.map(ProductTile.new).toList();
Timeline.finishSync();

// Инспекция объекта в DevTools:
inspect(myComplexObject);

// Постить кастомные события:
postEvent('flutter.frame', {'elapsed': elapsed, 'budget': 16});

// Сервисные расширения — кастомные кнопки в DevTools:
registerExtension('ext.myApp.clearCache', (method, params) async {
  cache.clear();
  return ServiceExtensionResponse.result('{"cleared": true}');
});
```

</details>

---

### Вопрос 10 🔴

Как корректно профилировать Flutter приложение (не debug, не release)?

- A) Debug режим — самый точный
- B) `flutter run --profile` — profile режим: без JIT дебаггера, с символами для profiler, ближе к release производительности
- C) Release режим для профилирования
- D) Нет специального режима

<details>
<summary>Ответ</summary>

**B) `--profile` режим — баланс между отладкой и реальной производительностью**

| Режим | JIT/AOT | Assertions | Profiler | Скорость |
|-------|---------|------------|---------|---------|
| debug | JIT | ✅ | ✅ | ~50% от release |
| profile | AOT | ❌ | ✅ | ~95% от release |
| release | AOT | ❌ | ❌ | 100% |

```bash
# Запустить в profile режиме:
flutter run --profile

# DevTools автоматически подключается
# Все инструменты доступны: Timeline, CPU Profiler, Memory

# ВАЖНО: никогда не делать выводы о производительности из debug!
# Debug режим может быть в 2-3 раза медленнее release.

# Для воспроизведения реального поведения:
flutter run --profile --no-dds  # без Dart Development Service
```

</details>
