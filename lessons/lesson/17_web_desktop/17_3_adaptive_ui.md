# 17.3 Adaptive и Responsive UI

## 1. Суть

- **Responsive** — UI меняет размер и расположение элементов под размер экрана (Small → Medium → Large).
- **Adaptive** — UI меняет поведение и компоненты под платформу (Mobile → Desktop).

```
Responsive: один компонент → разные размеры/layout
Adaptive:   разные компоненты под разные платформы
```

---

## 2. Брейкпоинты

```dart
class AppBreakpoints {
  static const double mobile = 600;
  static const double tablet = 900;
  static const double desktop = 1200;

  static bool isMobile(BuildContext context) =>
      MediaQuery.sizeOf(context).width < mobile;
  static bool isTablet(BuildContext context) {
    final w = MediaQuery.sizeOf(context).width;
    return w >= mobile && w < desktop;
  }
  static bool isDesktop(BuildContext context) =>
      MediaQuery.sizeOf(context).width >= desktop;
}
```

---

## 3. LayoutBuilder — responsive layout

```dart
class ResponsiveLayout extends StatelessWidget {
  final Widget mobile;
  final Widget? tablet;
  final Widget desktop;

  const ResponsiveLayout({
    super.key,
    required this.mobile,
    this.tablet,
    required this.desktop,
  });

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth >= AppBreakpoints.desktop) {
          return desktop;
        }
        if (constraints.maxWidth >= AppBreakpoints.mobile) {
          return tablet ?? desktop;
        }
        return mobile;
      },
    );
  }
}

// Использование:
ResponsiveLayout(
  mobile: const MobileHomeScreen(),
  tablet: const TabletHomeScreen(),
  desktop: const DesktopHomeScreen(),
)
```

---

## 4. Adaptive Navigation

```dart
class AdaptiveScaffold extends StatelessWidget {
  final int selectedIndex;
  final List<AdaptiveDestination> destinations;
  final ValueChanged<int> onDestinationSelected;
  final Widget body;

  const AdaptiveScaffold({
    super.key,
    required this.selectedIndex,
    required this.destinations,
    required this.onDestinationSelected,
    required this.body,
  });

  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        // Desktop / Tablet: NavigationRail или Drawer
        if (constraints.maxWidth >= 600) {
          return Scaffold(
            body: Row(
              children: [
                NavigationRail(
                  selectedIndex: selectedIndex,
                  onDestinationSelected: onDestinationSelected,
                  extended: constraints.maxWidth >= 1200, // расширенный вид
                  destinations: destinations
                      .map((d) => NavigationRailDestination(
                            icon: Icon(d.icon),
                            label: Text(d.label),
                          ))
                      .toList(),
                ),
                const VerticalDivider(thickness: 1, width: 1),
                Expanded(child: body),
              ],
            ),
          );
        }

        // Mobile: BottomNavigationBar
        return Scaffold(
          body: body,
          bottomNavigationBar: NavigationBar(
            selectedIndex: selectedIndex,
            onDestinationSelected: onDestinationSelected,
            destinations: destinations
                .map((d) => NavigationDestination(
                      icon: Icon(d.icon),
                      label: d.label,
                    ))
                .toList(),
          ),
        );
      },
    );
  }
}

class AdaptiveDestination {
  final IconData icon;
  final String label;
  const AdaptiveDestination({required this.icon, required this.label});
}
```

---

## 5. Платформо-специфичные виджеты

```dart
// Адаптивные диалоги
Future<bool?> showAdaptiveConfirm({
  required BuildContext context,
  required String title,
  required String message,
}) {
  if (Platform.isIOS || Platform.isMacOS) {
    return showCupertinoDialog<bool>(
      context: context,
      builder: (context) => CupertinoAlertDialog(
        title: Text(title),
        content: Text(message),
        actions: [
          CupertinoDialogAction(
            isDestructiveAction: true,
            onPressed: () => Navigator.pop(context, false),
            child: const Text('Отмена'),
          ),
          CupertinoDialogAction(
            onPressed: () => Navigator.pop(context, true),
            child: const Text('ОК'),
          ),
        ],
      ),
    );
  }

  return showDialog<bool>(
    context: context,
    builder: (context) => AlertDialog(
      title: Text(title),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context, false),
          child: const Text('Отмена'),
        ),
        TextButton(
          onPressed: () => Navigator.pop(context, true),
          child: const Text('ОК'),
        ),
      ],
    ),
  );
}
```

---

## 6. flutter_adaptive_scaffold

Flutter предоставляет `adaptive_scaffold` из `flutter/packages`:

```yaml
dependencies:
  flutter_adaptive_scaffold: ^0.2.1
```

```dart
import 'package:flutter_adaptive_scaffold/flutter_adaptive_scaffold.dart';

AdaptiveScaffold(
  selectedIndex: _selectedIndex,
  onSelectedIndexChange: (index) => setState(() => _selectedIndex = index),
  destinations: const [
    NavigationDestination(icon: Icon(Icons.home), label: 'Главная'),
    NavigationDestination(icon: Icon(Icons.search), label: 'Поиск'),
    NavigationDestination(icon: Icon(Icons.person), label: 'Профиль'),
  ],
  body: (_) => _screens[_selectedIndex],
  smallBreakpoint: const Breakpoint(endWidth: 700),
  mediumBreakpoint: const Breakpoint(beginWidth: 700, endWidth: 1000),
  largeBreakpoint: const Breakpoint(beginWidth: 1000),
)
```

---

## 7. Размеры текста и padding

```dart
// Адаптивный размер текста
class AdaptiveText extends StatelessWidget {
  final String text;
  const AdaptiveText(this.text, {super.key});

  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.sizeOf(context).width;
    final scale = width < 600 ? 1.0 : (width < 900 ? 1.2 : 1.5);

    return Text(
      text,
      style: TextStyle(
        fontSize: 16 * scale,
      ),
    );
  }
}

// Адаптивный padding
EdgeInsets adaptivePadding(BuildContext context) {
  final width = MediaQuery.sizeOf(context).width;
  if (width < 600) return const EdgeInsets.all(16);
  if (width < 1200) return const EdgeInsets.all(24);
  return EdgeInsets.symmetric(
    horizontal: (width - 1200) / 2 + 32, // центрирование контента
    vertical: 32,
  );
}
```

---

## 8. Mouse / Touch / Keyboard

```dart
// Hover-состояние — только для Desktop/Web
class HoverCard extends StatefulWidget {
  final Widget child;
  const HoverCard({super.key, required this.child});

  @override
  State<HoverCard> createState() => _HoverCardState();
}

class _HoverCardState extends State<HoverCard> {
  bool _hovered = false;

  @override
  Widget build(BuildContext context) {
    return MouseRegion(
      onEnter: (_) => setState(() => _hovered = true),
      onExit: (_) => setState(() => _hovered = false),
      cursor: SystemMouseCursors.click,
      child: AnimatedContainer(
        duration: const Duration(milliseconds: 150),
        decoration: BoxDecoration(
          boxShadow: _hovered
              ? [BoxShadow(color: Colors.black26, blurRadius: 12)]
              : [],
        ),
        child: widget.child,
      ),
    );
  }
}

// Скролл мышью — уже работает в Flutter нативно
// Drag для Desktop — GestureDetector + Pan
```

---

## 9. Типичные ошибки

| Ошибка                                   | Причина               | Решение                               |
| ---------------------------------------- | --------------------- | ------------------------------------- |
| Overflow при большом экране              | Фиксированные размеры | Использовать `Flexible`/`Expanded`    |
| BottomNavBar на Desktop                  | Не адаптирован        | NavigationRail для широких экранов    |
| Маленький текст на большом экране        | Нет масштабирования   | Адаптивный размер через LayoutBuilder |
| Tap-область слишком маленькая на Desktop | Рассчитано на пальцы  | `hitTestBehavior` + минимум 44px      |

---

## 10. Рекомендации

1. **`LayoutBuilder` > `MediaQuery`** для компонентов — не зависит от глобального контекста.
2. **NavigationRail на ≥600px** — стандарт Material Design 3.
3. **`MouseRegion`** для hover-эффектов на Desktop/Web.
4. **Центрируй контент на широких экранах** — `maxWidth: 1200` + `Align.center`.
5. **`flutter_adaptive_scaffold`** — готовый адаптивный scaffold из официальной библиотеки Flutter.
