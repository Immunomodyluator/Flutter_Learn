# 15.1 Flutter DevTools

## 1. Суть

**Flutter DevTools** — официальный набор инструментов для профилирования, отладки и анализа Flutter-приложений. Включает:

- **Flutter Inspector** — дерево виджетов, Layout Explorer.
- **Performance** (Timeline) — построение и отрисовка кадров.
- **CPU Profiler** — горячие точки выполнения кода.
- **Memory** — утечки памяти, аллокации.
- **Network** — HTTP-запросы.
- **Logging** — логи приложения.

---

## 2. Запуск DevTools

```bash
# Автоматически открывается при запуске в VS Code Debug
flutter run --profile

# Или открыть вручную через URL из консоли:
# flutter: Observatory listening on http://127.0.0.1:55555/...
dart devtools
# Затем вставить URL Observatory
```

В **VS Code**: `F5` → откроется браузер с DevTools после запуска.

---

## 3. Performance Timeline

**Что показывает:**

```
UI Thread  ████░░░░  8ms
Raster Thread  ██░░░░  4ms
```

- **UI Thread** (синий) — Dart-код: build, layout, compositing.
- **Raster Thread** (зелёный) — GPU: рисует слои на экране.
- Красная зона выше 16ms — кадр «джанки».

**Как использовать:**

1. Запусти в `--profile` режиме.
2. Открой Performance вкладку.
3. Нажми Record.
4. Выполни действие (скролл, анимация).
5. Нажми Stop.
6. Найди кад с пиком > 16ms → нажми на него.
7. Посмотри, какой виджет/метод занял больше всего времени.

---

## 4. Flutter Inspector

**Highlight repaints:**

```dart
// Включить визуальную подсветку перерисовок
import 'package:flutter/rendering.dart';

void main() {
  debugRepaintRainbowEnabled = true; // перерисованные области меняют цвет
  runApp(const MyApp());
}
```

**Rebuild count (DevTools):**
В Flutter Inspector → нажми на виджет → счётчик rebuild покажет, сколько раз перестраивался.

---

## 5. CPU Profiler

```dart
// Добавить кастомную метрику в Timeline
import 'dart:developer' as dev;

Future<void> parseData(String json) async {
  dev.Timeline.startSync('ParseData');
  // ... тяжёлые вычисления
  dev.Timeline.finishSync();
}

// Или через TimelineTask для async операций
final task = dev.TimelineTask();
task.start('FetchAndParse');
final data = await fetchData();
task.instant('DataFetched');
final parsed = parse(data);
task.finish();
```

В CPU Profiler увидишь `ParseData` как отдельный блок в flame graph.

---

## 6. Memory Profiler

**Диагностика утечек:**

1. Открой Memory вкладку.
2. Нажми GC несколько раз — выполни принудительную сборку мусора.
3. Перейди на экран и вернись назад.
4. Снова GC.
5. Если в таблице `Instances` количество объектов растёт — утечка памяти.

**Частые причины утечек:**

```dart
// ❌ AnimationController без dispose
class _MyState extends State<MyWidget> with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(vsync: this, duration: ...);
    // ЗАБЫЛИ dispose -> утечка
  }
}

// ✅ Правильно
@override
void dispose() {
  _controller.dispose();
  super.dispose();
}
```

---

## 7. Network профилировщик

Отображает все HTTP-запросы с:

- URL, метод, статус.
- Время запроса и ответа.
- Тело запроса и ответа.
- Заголовки.

**Включение HttpClient logging:**

```dart
// С Dio — включается автоматически при использовании LogInterceptor
dio.interceptors.add(LogInterceptor(
  requestBody: true,
  responseBody: true,
));
```

---

## 8. debugPrintRebuildDirtyWidgets

```dart
void main() {
  debugPrintRebuildDirtyWidgets = true; // в консоль выводятся все rebuild
  runApp(const MyApp());
}

// Консоль:
// REBUILDING: MaterialApp (DIRTY)
// REBUILDING: MyHomePage (DIRTY)
// REBUILDING: Text (DIRTY)
```

---

## 9. Layout Explorer

Визуализирует Flex-контейнеры (Row, Column):

- Показывает, сколько пространства занимает каждый child.
- Flex factor каждого Expanded/Flexible.
- Переполнение (overflow).

---

## 10. Типичные ошибки

| Ошибка                         | Признак                                     | Решение                                         |
| ------------------------------ | ------------------------------------------- | ----------------------------------------------- |
| Профилируешь в Debug-режиме    | Результаты некорректны                      | Запускать с `--profile`                         |
| Игнорируешь Raster thread      | Тормозит GPU, не CPU                        | Использовать `RepaintBoundary`, кешировать слои |
| Не включаешь GC перед анализом | Ложные данные по памяти                     | Всегда жми GC перед snapshot                    |
| Профилируешь на эмуляторе      | Производительность отличается от устройства | Только реальное устройство                      |

---

## 11. Рекомендации

1. **`--profile` на реальном устройстве** — золотое правило.
2. **Timeline сначала** — найди пики >16ms, потом смотри CPU.
3. **Memory snapshot** — сравнивай до и после действия пользователя.
4. **`debugRepaintRainbowEnabled`** — быстрый способ найти виджеты, которые рисуются слишком часто.
5. **Кастомные метрики** `dev.Timeline` — маркируй свои операции для точного профилирования.
