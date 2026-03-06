# Квиз: Adaptive UI

**Тема:** 17.3 — Adaptive UI for Web & Desktop  
**Вопросов:** 10 | 🟢 1–3 · 🟡 4–7 · 🔴 8–10

---

### Вопрос 1 🟢

Чем адаптивный UI отличается от отзывчивого (responsive)?

- A) Это одно и то же
- B) Responsive — UI адаптируется под размер экрана (breakpoints, flex); Adaptive — UI использует нативные паттерны взаимодействия платформы (мышь, клавиатура, контекстные меню)
- C) Adaptive только для iOS и Android
- D) Responsive — для Web, Adaptive — для Desktop

<details>
<summary>Ответ</summary>

**B) Responsive = размер экрана; Adaptive = паттерны взаимодействия**

```
Responsive (размер/layout):
- Мобильный: BottomNavigation + список
- Планшет: NavigationRail + список + детали
- Десктоп: NavigationDrawer + master/detail

Adaptive (взаимодействие):
- Mobile: tap, swipe, long press
- Desktop/Web: hover, right-click меню, keyboard shortcuts,
  cursor changes, scrollbar always visible

Flutter инструменты:
- Responsive: MediaQuery, LayoutBuilder, ConstrainedBox
- Adaptive: MouseRegion, Listener, Focus, Shortcuts,
  Actions, ContextMenu, Scrollbar с всегда видимым thumb
```

</details>

---

### Вопрос 2 🟢

Как определить тип устройства ввода (мышь/тач) в Flutter?

- A) `Platform.isMobile`
- B) `MediaQuery.of(context).pointerDeviceKind`; или проверить `kIsWeb`/`Platform.isDesktop`; или `Theme.of(context).platform`
- C) `InputDevice.current()`
- D) `GestureDetector` определяет автоматически

<details>
<summary>Ответ</summary>

**B) `PointerDeviceKind` через event или platform check**

```dart
// Проверить платформу:
bool get isDesktopOrWeb {
  if (kIsWeb) return true;
  return Platform.isWindows || Platform.isMacOS || Platform.isLinux;
}

// Определить устройство ввода из события:
Listener(
  onPointerDown: (PointerDownEvent event) {
    if (event.kind == PointerDeviceKind.mouse) {
      // мышь
    } else if (event.kind == PointerDeviceKind.touch) {
      // тач
    } else if (event.kind == PointerDeviceKind.stylus) {
      // стилус
    }
  },
  child: myWidget,
)

// Адаптивный виджет — Material 3 AdaptiveTheme:
import 'package:flutter_adaptive_scaffold/flutter_adaptive_scaffold.dart';
// google/flutter.dev пакет для adaptive scaffold
```

</details>

---

### Вопрос 3 🟢

Как добавить hover-эффект в Flutter Desktop/Web?

- A) `onHover` параметр во всех виджетах
- B) `MouseRegion` виджет с `onEnter`, `onExit`, `onHover` callbacks + смена курсора через `cursor` параметр
- C) `HoverButton` виджет
- D) CSS :hover через platform channel

<details>
<summary>Ответ</summary>

**B) `MouseRegion` виджет**

```dart
class HoverCard extends StatefulWidget {
  final Widget child;
  const HoverCard({required this.child});
  @override State<HoverCard> createState() => _HoverCardState();
}

class _HoverCardState extends State<HoverCard> {
  bool _isHovered = false;

  @override
  Widget build(BuildContext context) {
    return MouseRegion(
      // Изменить курсор при наведении:
      cursor: SystemMouseCursors.click,

      onEnter: (_) => setState(() => _isHovered = true),
      onExit: (_) => setState(() => _isHovered = false),

      child: AnimatedContainer(
        duration: const Duration(milliseconds: 150),
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(8),
          boxShadow: _isHovered
              ? [BoxShadow(blurRadius: 8, color: Colors.black26)]
              : [],
          color: _isHovered ? Colors.blue.shade50 : Colors.white,
        ),
        child: widget.child,
      ),
    );
  }
}
```

</details>

---

### Вопрос 4 🟡

Как реализовать контекстное меню (правая кнопка мыши) в Flutter?

- A) `GestureDetector.onSecondaryTap`
- B) `GestureDetector.onSecondaryTap` или `ContextMenuRegion`; Flutter 3+ встроенный `AdaptiveTextSelectionToolbar`; или кастомный `Overlay`
- C) Контекстное меню только в HTML renderer
- D) `RightClickMenu` встроенный виджет

<details>
<summary>Ответ</summary>

**B) `GestureDetector.onSecondaryTap` + кастомный Overlay**

```dart
class ContextMenuWrapper extends StatelessWidget {
  final Widget child;
  final List<ContextMenuItem> items;

  const ContextMenuWrapper({required this.child, required this.items});

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      // ПКМ / долгое нажатие:
      onSecondaryTapDown: (details) =>
          _showContextMenu(context, details.globalPosition),
      onLongPressStart: (details) =>
          _showContextMenu(context, details.globalPosition),
      child: child,
    );
  }

  void _showContextMenu(BuildContext context, Offset position) {
    showMenu(
      context: context,
      position: RelativeRect.fromLTRB(
        position.dx, position.dy,
        position.dx + 1, position.dy + 1,
      ),
      items: items.map((item) => PopupMenuItem(
        value: item.value,
        child: Row(children: [
          Icon(item.icon),
          const SizedBox(width: 8),
          Text(item.label),
        ]),
      )).toList(),
    ).then((value) => value != null ? item.onTap() : null);
  }
}
```

</details>

---

### Вопрос 5 🟡

Как реализовать keyboard shortcuts в Flutter?

- A) `onKeyDown` в виджете
- B) `Shortcuts` + `Actions` виджеты: Shortcuts маппит LogicalKeySet → Intent; Actions выполняет Intent
- C) `KeyboardListener.onKey` достаточно
- D) `HotKey` только для десктопа

<details>
<summary>Ответ</summary>

**B) `Shortcuts` + `Actions`**

```dart
// Определить Intent:
class SaveIntent extends Intent {
  const SaveIntent();
}

class NewDocumentIntent extends Intent {
  const NewDocumentIntent();
}

// Зарегистрировать shortcuts и actions:
Shortcuts(
  shortcuts: {
    const SingleActivator(LogicalKeyboardKey.keyS, control: true):
        const SaveIntent(),
    const SingleActivator(LogicalKeyboardKey.keyN, control: true):
        const NewDocumentIntent(),
    // Cmd+S на macOS:
    const SingleActivator(LogicalKeyboardKey.keyS, meta: true):
        const SaveIntent(),
  },
  child: Actions(
    actions: {
      SaveIntent: CallbackAction<SaveIntent>(
        onInvoke: (_) => saveDocument(),
      ),
      NewDocumentIntent: CallbackAction<NewDocumentIntent>(
        onInvoke: (_) => createNewDocument(),
      ),
    },
    child: Focus(
      autofocus: true, // ВАЖНО: Focus нужен для получения клавиш
      child: myApp,
    ),
  ),
)
```

</details>

---

### Вопрос 6 🟡

Как адаптировать навигацию под размер окна (мобиль/планшет/десктоп)?

- A) Один BottomNavigationBar для всех платформ
- B) `AdaptiveScaffold` из `flutter_adaptive_scaffold`; или вручную: BottomNav (мобиль) → NavigationRail (планшет) → NavigationDrawer (десктоп)
- C) Только `Drawer` — самый универсальный
- D) `PlatformNavigationBar` встроенный виджет

<details>
<summary>Ответ</summary>

**B) `AdaptiveScaffold` или ручная адаптация**

```dart
// Ручная реализация:
class AdaptiveLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;

    if (width < 600) {
      // Мобильный — BottomNavigationBar:
      return Scaffold(
        body: _buildBody(),
        bottomNavigationBar: BottomNavigationBar(items: _navItems),
      );
    } else if (width < 1200) {
      // Планшет — NavigationRail:
      return Scaffold(
        body: Row(children: [
          NavigationRail(destinations: _railDestinations, selectedIndex: _idx),
          const VerticalDivider(width: 1),
          Expanded(child: _buildBody()),
        ]),
      );
    } else {
      // Десктоп — NavigationDrawer + Master/Detail:
      return Scaffold(
        body: Row(children: [
          NavigationDrawer(children: _drawerItems),
          Expanded(child: _masterDetailLayout()),
        ]),
      );
    }
  }
}
```

</details>

---

### Вопрос 7 🟡

Как сделать Scrollbar всегда видимым на Desktop/Web?

- A) `Scrollbar` автоматически адаптируется
- B) `ScrollbarTheme` с `thumbVisibility: true`; или `Scrollbar(thumbVisibility: true, child: ...)`; настроить через `ThemeData.scrollbarTheme`
- C) `alwaysShowScrollbar: true` в `ListView`
- D) CSS scrollbar для Web, нативный для Desktop

<details>
<summary>Ответ</summary>

**B) `thumbVisibility: true` или через `ThemeData`**

```dart
// Глобально для всего приложения:
MaterialApp(
  theme: ThemeData(
    scrollbarTheme: ScrollbarThemeData(
      thumbVisibility: WidgetStateProperty.resolveWith((states) {
        // Всегда видим на десктопе:
        if (kIsWeb || Platform.isDesktop) return true;
        // На мобильном — только при скролле:
        return states.contains(WidgetState.dragged) ||
               states.contains(WidgetState.hovered);
      }),
      thickness: WidgetStateProperty.all(8.0),
      thumbColor: WidgetStateProperty.all(Colors.grey.shade400),
      radius: const Radius.circular(4),
    ),
  ),
)

// Для конкретного ScrollView:
Scrollbar(
  thumbVisibility: true,
  controller: _scrollController,
  child: ListView.builder(
    controller: _scrollController,
    itemBuilder: ...,
  ),
)
```

</details>

---

### Вопрос 8 🔴

Как реализовать resizable panels (разделители между панелями) в Flutter Desktop?

- A) `Row` с `Flexible` достаточно
- B) Кастомный `SplitView` с `GestureDetector` на разделителе; или `split_view` пакет
- C) `ResizablePanel` встроенный виджет Flutter
- D) Только через нативный platform channel

<details>
<summary>Ответ</summary>

**B) Кастомный SplitView с draggable divider**

```dart
class ResizableSplitView extends StatefulWidget {
  final Widget left;
  final Widget right;
  final double initialLeftFraction;

  const ResizableSplitView({
    required this.left, required this.right,
    this.initialLeftFraction = 0.3,
  });

  @override State<ResizableSplitView> createState() =>
      _ResizableSplitViewState();
}

class _ResizableSplitViewState extends State<ResizableSplitView> {
  late double _leftFraction;

  @override
  void initState() {
    super.initState();
    _leftFraction = widget.initialLeftFraction;
  }

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(builder: (_, constraints) {
      final leftWidth = constraints.maxWidth * _leftFraction;
      return Row(children: [
        SizedBox(width: leftWidth, child: widget.left),
        // Разделитель:
        MouseRegion(
          cursor: SystemMouseCursors.resizeLeftRight,
          child: GestureDetector(
            onHorizontalDragUpdate: (d) => setState(() {
              _leftFraction = (_leftFraction + d.delta.dx / constraints.maxWidth)
                  .clamp(0.1, 0.9);
            }),
            child: Container(width: 4, color: Colors.grey.shade300),
          ),
        ),
        Expanded(child: widget.right),
      ]);
    });
  }
}
```

</details>

---

### Вопрос 9 🔴

Как адаптировать text selection и copy/paste для Desktop?

- A) `SelectableText` — единственный вариант
- B) `SelectionArea` (Flutter 3.3+) оборачивает любые виджеты; добавляет выделение текста, copy, поиск; настраивается через `SelectionContainer`
- C) Desktop copy/paste работает автоматически
- D) Нужен platform channel для clipboard

<details>
<summary>Ответ</summary>

**B) `SelectionArea` для выделения в любых виджетах**

```dart
// SelectionArea — включает выделение текста во всём поддереве:
SelectionArea(
  child: Column(children: [
    const Text('Этот текст можно выделить'),
    const Text('И этот тоже'),
    // Даже виджеты внутри ListTile:
    ListView.builder(
      itemBuilder: (_, i) => ListTile(
        title: Text('Item $i — text is selectable'),
      ),
    ),
  ]),
)

// Clipboard из кода:
import 'package:flutter/services.dart';

// Копировать:
await Clipboard.setData(const ClipboardData(text: 'Copied!'));

// Вставить:
final data = await Clipboard.getData('text/plain');
final text = data?.text ?? '';

// Адаптивные TextField настройки:
TextField(
  // Desktop: показывать все toolbar кнопки
  contextMenuBuilder: (context, editableTextState) {
    return AdaptiveTextSelectionToolbar.editableText(
      editableTextState: editableTextState,
    );
  },
)
```

</details>

---

### Вопрос 10 🔴

Как настроить разные размеры окна (breakpoints) для адаптивного дизайна?

- A) Использовать только `MediaQuery.size`
- B) `MediaQuery.sizeOf(context)` для размера без лишних rebuild; `LayoutBuilder` для constraints; определить систему breakpoints как константы
- C) `window.size` из dart:html
- D) Breakpoints — только для Web

<details>
<summary>Ответ</summary>

**B) `MediaQuery.sizeOf` + `LayoutBuilder` + константы**

```dart
// Система breakpoints:
class AppBreakpoints {
  static const compact = 600.0;    // мобильный (< 600)
  static const medium = 840.0;     // планшет (600-840)
  static const expanded = 1200.0;  // большой планшет (840-1200)
  // > 1200 = десктоп
}

// Эффективное использование — sizeOf не подписывает на всё:
class ResponsiveWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Только при изменении SIZE делается rebuild:
    final size = MediaQuery.sizeOf(context);
    final width = size.width;

    return LayoutBuilder(
      builder: (context, constraints) {
        // constraints = реальные ограничения этого виджета
        if (constraints.maxWidth < AppBreakpoints.compact) {
          return const CompactLayout();
        } else if (constraints.maxWidth < AppBreakpoints.medium) {
          return const MediumLayout();
        } else {
          return const ExpandedLayout();
        }
      },
    );
  }
}

// flutter_adaptive_scaffold из Google:
// Автоматическая адаптация на основе Material Design breakpoints
import 'package:flutter_adaptive_scaffold/flutter_adaptive_scaffold.dart';
```

</details>
